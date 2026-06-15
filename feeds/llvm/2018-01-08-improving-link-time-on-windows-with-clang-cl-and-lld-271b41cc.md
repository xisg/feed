---
title: Improving Link Time on Windows with clang-cl and lld
url: https://blog.llvm.org/2018/01/improving-link-time-on-windows-with.html
published: "2018-01-08T10:06:00Z"
feed: llvm
guid: https://blog.llvm.org/2018/01/improving-link-time-on-windows-with.html
---

# Improving Link Time on Windows with clang-cl and lld

One of our goals in bringing clang and lld to Windows has always been to improve developer experience, and what is it that developers want the most?  Faster build times!  Recently, our focus has been on improving link time because it's the step that's the hardest to parallelize so we can't fall back on the time honored tradition of throwing more cores at it.

Of the various steps involved in linking, generating the debug info (which, on Windows, is a PDB file) is by far the slowest since it involves merging O(# of linker inputs) sequences of type records, most of which are duplicate anyway.  For example, if two cpp files both include <string>, then both of those object files will have hundreds of duplicate type records that need to be de-duplicated during the link step.  This means you have to compute O(M x N) hash values, even though only a small fraction of those ultimately contribute to the final PDB.

Several strategies have been invented to deal with this over the years and try to make linking faster.  Many years ago, Microsoft introduced the notion of a _Type Server_ (enabled via **/Zi** compiler option in MSVC), which moves some of the work into the compiler (to take advantage of parallelism).  More recently we have been given the **/DEBUG:FASTLINK** linker option which attempts to solve the problem by not merging types at all in the linker.  However, each of these strategies has its own set of disadvantages, and neither can be considered perfect for all use cases.

In this blog post, we'll first go over some technical background about CodeView so that we can understand the problem, followed by a summary of existing attempts to speed up type merging.  Then, we'll describe a novel extension to the PE/COFF file format which speeds up linking by offloading part of the work required to de-duplicate types to the compiler and using a new algorithm which uniquely identifies type records even across input files, and discuss the various tradeoffs of each approach.  Finally, we'll present some benchmarks and discuss how you can try this out in clang-cl and lld today.

## Background

Consider a simple structure in C++, defined like this a header file:

structNode {

Node \*Next = nullptr;

Node \*Prev = nullptr;

int Value = 0;

     };

Since each compilation happens independently of every other compilation, the compiler cannot assume any other translation unit will ever emit the records necessary to describe this type.  As a result, to guarantee that the type makes it into the final PDB, _every_ compiler instance that encounters this definition must emit type information for this type.  So the record will be serialized by the compiler into a series of records that looks roughly like this:

0x1004 \| LF\_STRUCTURE \[size = 40\] \`Node\`

         unique name: \`.?AUNode@@\`

         vtable: <none>

         base list: <none>

         field list: <none>

         options: forward ref \| has unique name

0x1005 \| LF\_POINTER \[size = 12\]

         referent = 0x1004

         mode = pointer

         opts = None

         kind = ptr32

0x1006 \| LF\_FIELDLIST \[size = 52\]

         \- LF\_MEMBER

           name = \`Next\`

           Type = 0x1005

           Offset = 0

           attrs = public

         \- LF\_MEMBER

           name = \`Prev\`

           Type = 0x1005

           Offset = 4

           attrs = public

         \- LF\_MEMBER

           name = \`Value\`

           Type = 0x0074 (int)

           Offset = 8

           attrs = public

0x1007 \| LF\_STRUCTURE \[size = 40\] \`Node\`

         unique name: \`.?AUNode@@\`

         vtable: <none>

         base list: <none>

         field list: 0x1006

         options: has unique name

The values on the left correspond to the types _index_ in the type sequence and depend on what types have already been encountered, while other types can the refer to them (for example, referent = 0x1004) means that this record is a pointer to whatever the type at index 0x1004 was.

As a result of this design, another compilation unit which includes the same header file will need to emit this exact same type, with the only difference being the indices (since the other compilation may encounter other types before this one, causing the ordering to be different).

In short, type indices only make sense within the context of a single type sequence (i.e. compiland), but since the linker needs to see across _all_ object files, it has to have some way of identifying whether a type from object file A is isomorphic to a different type from object file B, even if its type indices might be different numerically from any previously seen type.

This algorithm, henceforth referred to as _type merging_, is the primary consumer of CPU cycles during linking (measured in LLD, and estimated in MSVC linker by comparing /DEBUG:FULL vs /DEBUG:FASTLINK times), and as such it is the portion of the linking process which this blog post presents a new solution to.

## Existing Solutions

It’s worthwhile to discuss some of the existing attempts to reduce the cost associated with type merging so that we can compare and contrast their various pros and cons.

### Type Servers (/Zi)

The /Zi compiler option was one of the first attempts to address type merging speed, and it dates back many years.  The idea behind type servers is to offload the work of de-duplication from the linking phase to the compilation phase.  Most build systems already support parallel compilation, and even if they don’t cl.exe supports it natively via the /MP compiler switch, so there is no roadblock to anyone taking advantage of parallel compilation.

To implement type servers, each compilation process communicates via IPC with a single process (mspdbsrv.exe) whose job is to de-duplicate type records on the fly, and when a record is isomorphic to an existing record, the type server communicates back the previously saved index, and when it is new it sends back a new index.  This allows type deduplication to happen _mostly_ in parallel, but adding some overhead to each compilation (since there is contention over a global lock) in return for significantly reduced link times, since types will already have been merged.

Type servers bring with them some disadvantages though, so we enumerate them here:

1. Type servers add significant context switching and global lock contention to the compilation phase, reducing parallelism and degrading overall system performance while a build is in process.  While some performance is reclaimed from the linker, some is sacrificed due to the use of a global system lock.  It’s still a net win, but as it is not free, it leaves open the possibility that we may be able to achieve better parallelism using a different approach.
2. The type server process itself (mspdbsrv.exe) introduces a single point of failure.  When it crashes (we see C1033 several times per day on Chrome, for example, which seems to indicate an mspdbsrv.exe crash) it could trigger a full rebuild if the type server PDB file is left in a corrupt state.
3. mspdbsrv is incompatible with distributed builds, which is a show-stopper for large applications that can take several hours to build on normal workstations.  Type servers operate only via local IPC.  While multi-processing works well for small applications, many large products have build farms that distribute compilations among tens or hundreds of physical machines.  Type servers are incompatible with this scenario.

## Fastlink PDBs

Fastlink PDBs are a relatively recent introduction, and the approach used by this solution is to _eliminate type merging entirely._  To support this, special metadata is set in the PDB file to indicate to the tool that this is a fastlink PDB, and when the tool (e.g. debugger) encounters this metadata, it will fetch all type information from the original object file, rather than from the PDB.  As before, there are several disadvantages to this approach, enumerated here:

1. The pdbcopy utility is almost unusable with fastlink PDBs for performance reasons.
2. Since type merging doesn’t happen, indexing of type information also doesn’t happen (since the expensive part of building an index -- the hashing -- comes for free when you were hashing the record anyway).  This leads to degradation in the debugger user experience, since waits which previously happened only at build time now happen at debug-time.
3. Fastlink PDBs are not portable.  The PDB references the object files by path, so if you copy the PDB and object files to a different machine (or even different path on the same machine) for archival purposes, they can no longer be debugged.  This is a deal-breaker for using it on production builds
4. Symbols can’t be enumerated in a Fastlink PDB.  This is most obvious if you attempt to use DIA SDK on a Fastlink PDB, where it will simply refuse to do anything at all.  This means that the only externally supported way of querying debug info for users is impossible against a Fastlink PDB.  Beyond that, however, it also means that even Microsoft’s own tools which need to enumerate symbols cannot use any standard API for doing so.  For example, WinDbg doesn’t fully support Fastlink PDBs, and many workflows are broken by the use of them, even using supported Microsoft tools.
5. It has several serious stability issues which make it unusable on large projects  \[ [ref](https://developercommunity.visualstudio.com/content/problem/158451/chrome-with-1550-preview-50-and-fastlink-crashes-d.html)\].  This is probably related to point 4 above, namely the fact that every tool that wants to be able to work with a Fastlink PDB needs to use different code than the SDK that has been tested and battle-hardened through years of development.
6. When compiling with clang-cl and linking with /debug:fastlink the compiler has to be instructed to [emit additional debug information](https://cs.chromium.org/chromium/src/build/config/compiler/BUILD.gn?type=cs&q=build/config/compiler/BUILD.gn&sq=package:chromium&l=1886), making .obj files about 29% larger.

## Clang's Solution - The COFF .debug$H section

This new approach tries to combine the ideas behind type servers and fastlink PDBs.  Like type servers, it attempts to offload the work of de-duplication to the compilation phase so that it can be done in parallel.  However, it does so using an algorithm with the property that the resulting hash can be used to identify a type record even across type streams.  Specifically, **if two records have the same hash, they are the same record even if they are from different object files.**  If you can take it on faith that such an algorithm exists (which will be henceforth referred to as a _global hash_), then the amount of work that the linker needs to perform is greatly reduced.  And the work that it does still have to do can be done much quicker.  Perhaps most importantly, _it produces a byte-for-byte identical PDB to when the option is not used_, meaning all of the issues surrounding Fastlink PDBs and compatibility are gone.

Previously, the linker would do something that looks roughly like this:

     HashTable<Type> HashedTypes;

     vector<Type> MergedTypes;

     for (ObjectFile &Obj : Objects) {

       for (Type &T : Obj.types()) {

         remapAllTypeIndices(MergedTypes, T);

         if (!HashedTypes.try\_insert(T))

           continue;

         MergedTypes.push\_back(T);

       }

     }

The important observations here are:

1. remapAllTypeIndices is called unconditionally for every type in every object file.
2. A hash of the type is computed unconditionally for every type
3. _At least one_ full record comparison is done for every type.  In practice it turns out to be much more, because hash buckets are computed modulo table size, so there will actually be 1 full record comparison for every probe.

Given a global hash function as described above, the algorithm can be re-written like this:

      HashMap<SHA1, int> HashToIndex;

      vector<Type> OrderedTypes;

      for (ObjectFile &Obj : Objects) {

        auto Hashes = Obj.DebugHSectionHashes;

        for (int I=0; I < Obj.NumTypes; ++I) {

          int NextIndex = OrderedTypes.size();

          if (!HashToIndex.try\_emplace(Hashes\[I\], NextIndex))

            continue;

          remapAllTypeIndices(T);

          OrderedTypes.push\_back(T);

        }

      }

While this appears very similar, its performance characteristics are quite different.

1. remapAllTypeIndices is only called when the record is actually new.  Which, as we discussed earlier, is a small fraction of the time over many linker inputs.
2. A hash of the type is **never** computed by the linker.It is simply there in the object file (the exception to this is mixed linker inputs, discussed earlier, but those are a small fraction of input files).
3. Full record comparisons **never** happen _._  Since we are using a strong hash function with negligible chance of false collisions, and since the hash of a record provides equality semantics across streams, the hash is as good as the record itself.

Combining all of these points, we get an algorithm that is extremely cache friendly.  Amortized over all input files, most records during type merging are cache hits (i.e. duplicate records).  With this algorithm when we get a cache hit, the only two data structures that are accessed are:

1. An array of contiguous hash values.
2. An array of contiguous hash buckets.

Since we never do full equality comparison (which would blow out the L1 and sometimes even L2 cache due to the average size of a type record being larger than a cache line) the algorithm here is very fast.

We’ve deferred discussion of how to create such a hash up until now, but it is actually fairly straightforward.  We use what is known as a “tree hash” or “Merkle tree”.  The idea is to pass bytes from a type record directly to the hash function up until the point we get to a type index.  Then, instead of passing the numeric value of the type index to the hash function, we pass the previously computed hash of the record that is being referenced.

Such a hash is very fast to compute in the compiler because _the compiler must already hash types anyway_, so the incremental cost to emit this to the .debug$H section is negligible.  For example, when a type is encountered in a translation unit, before you can add that type to the object file’s .debug$T section, it must first be verified that the type has not already been added.  And since this is happening naturally in the order in which types are encountered, all that has to be done is to save these hash values in an array indexed by type index, and subsequent hash operations will have O(1) access to all of the information needed to compute this merkle hash.

### Mixed Input Files and Compiler/Linker Compatibility

A linker must be prepared to deal with a mixed set of input files.  For example, while a particular compiler may choose to always emit .debug$H sections, a linker must be prepared to link objects that for whatever reason do not have this section.  To handle this, the linker can examine all inputs up front and manually compute hashes for inputs with missing .debug$Hsections.  In practice this proves to be a small fraction and the penalty for doing this serially is negligible, although it should be noted that in theory this can also be done as a parallel pre-processing step if some use cases show that this has non-negligible cost.

Similarly, the emission of this section in an object file has no impact on linkers which have not been taught to use it.  Since it is a purely additive (and optional) inclusion into the object file, any linker which does not understand it will continue to work exactly as it does today.

## The On-Disk Format

Clang uses the following on-disk format for the .debug$H section.

0x0     : <Section Magic>  (4 bytes)

     0x4     : <Version>        (2 bytes)

     0x6     : <Hash Algorithm> (2 bytes)

     0x8     : <Hash Value>     (N bytes)

     0x8 + N : <Hash Value>     (N bytes)

                    …

Here, “Section Magic” is an arbitrarily chosen 4-byte number whose purpose is to provide some level of certainty that what we’re seeing is a real .debug$H section, and not some section that someone created that accidentally happened to be called that.   Our current implementation uses the value 0x133C9C5, which represents the date of the initial prototype implementation.  But this can be any reasonable value here, as long as it never changes.

“Version” is reserved for future use, so that the format of the section can theoretically change.

“Hash Algorithm” is a value that indicates what algorithm was used to generate the hashes that follow.  As such, the value of N above is also a function of what hash algorithm is used.  Currently, the only proposed value for Hash Algorithm is SHA1 = 0, which would imply N = 20 when Hash Algorithm = 0.  Should it prove useful to have truncated 8-byte SHA1 hashes, we could define SHA1\_8 = 1, for example.

## Limitations and Pitfalls

The biggest limitation of this format is that it increases object file size.  Experiments locally on fairly large projects show an average aggregate object file size increase of ~15% compared to /DEBUG:FULL (which, for clang-cl, actually makes .debug$H object files _smaller_ than those needed to support /DEBUG:FASTLINK).

There is another, less obvious potential pitfall as well.  The worst case scenario is when no input files have a .debug$H section present, but this limitation is the same in principle even if only a subset of files have a .debug$H section.  Since the linker must agree on a single hash function for all object files, there is the question of what to do when not all object files agree on hash function, or when not all object files contain a .debug$H section.  If the code is not written carefully, you could get into a situation where, for example, no input files contain a .debug$H section so the linker decides to synthesize one on the fly for every input file.  Since SHA1 (for example) is quite slow, this could cause a huge performance penalty.

This limitation can be coded around with some care, however.  For example, tree hashes can be computed up-front in parallel as a pre-processing step.  Alternatively, a hash function could be chosen based on some heuristic estimate of what would likely lead to the fastest link (based on the percentage of inputs that had a .debug$H section, for example).  There are other possibilities as well.  The important thing is to just be aware of this potential pitfall, and if your links become very slow, you'll know that the first thing you should check is "do all my object files have .debug$H sections?"

Finally, since a hash is considered to be identical to the original record, we must consider the possibility of collisions.  That said, this does not appear to be a serious concern in practice.  A single PDB can have a theoretical maximum of 232 type records anyway (due to a type index being 4 bytes).  The following table shows the expected number of type records needed for a collision to exist as a function of hash size.

Hash Size (Bytes)

Average # of records needed for a collision

4

82,137

6

21,027,121

8

5,382,943,231

12

3.53 x 1014

16

2.31 x 1019

20

1.52 x 1024

Given that this is strictly for debug information and not generated code, it’s worth thinking about the _severity_ of a collision.  We feel that an 8-byte hash is probably acceptable for real world use.

## Benchmarks

Here we will give some benchmarks on large real world applications (specifically, Chrome and clang).  The times presented are **only** for the linker.  gn args for each build of chromium are specified at the end..

Toolchain

Mode

Target

blink\_core.dll

content.dll

chrome.dll

clang.exe

MSVC

/DEBUG:FULL

553.11s

205.45s

507.17s

62.45s

MSVC

/DEBUG:FASTLINK

116.77s

56.05s

67.80s

29.37s

lld-link

/DEBUG:FULL

121.17s

42.10s

42.31s

24.14s

lld-link

/DEBUG:GHASH

88.71s

33.30s

34.76s

17.99s

The numbers here indicate a reduction in link time of up to 30% by enabling /DEBUG:GHASH in lld.

It's worth mentioning that lld does not yet have support for incremental linking so we could not compare the cost of an incremental link with /DEBUG:GHASH versus MSVC.  We still expect incremental linking using MSVC under optimal conditions (e.g. change whitespace in a header file) to produce much faster links than lld is currently able to do.

There are several possible avenues for further optimization though, so we will finish up by discussing them.

## Further Improvements

There are several ways to improve the times further, which have yet to be explored.

1. Use a smaller or faster hash.  We use a 20-byte SHA1 hash.  This is not a multiple of cache line size, and in any case the probability of collision is astronomically small even in the largest PDBs, considering that the theoretical limit of a PDB is just under 2^32 possible unique types (due to the 4-byte size of a type index).  SHA1 is also notoriously slow.  It might be interesting to try, for example, a Blake2 set to output an 8 byte hash.  This should give sufficiently low probability of a collision while improving cache performance.  The on-disk format is designed with this flexibility in mind, as different hash algorithms can be specified in the header.
2. Hashes for compilands with missing .debug$H sections can be computed in parallel before linking.  Currently when we encounter an object file without a .debug$H section, we must synthesize one in the linker.  Our prototype algorithm does this serially for each input.
3. Symbol records from .debug$S sections can be merged in parallel.  Currently in lld, we first merge type records into the TPI stream, then we iterate symbol records and remap types in each symbol record to correspond to the new type indices.  If we merge types from all modules up front, the symbol records (with the exception of global symbols) can be merged in parallel since they get written to independent streams).

## Try it out!

If you're already using clang-cl and lld on Windows today, you can try this out.  There are two flags needed to enable this, one for the compiler and one for the linker:

1. To enable the emission of a .debug$H section by the compiler, you will need to pass the undocumented -mllvm -emit-codeview-ghash-section flag to clang-cl  (this flag should go away in the future, once this is considered stable and good enough to be turned on by default).
2. To tell lld to use this information, you will need to pass the /DEBUG:GHASH to lld.

Note that this feature is still considered highly experimental, so we're interested in your feedback (llvm-dev@ mailing list, direct email is ok too) and bug reports (bugs.llvm.org).
