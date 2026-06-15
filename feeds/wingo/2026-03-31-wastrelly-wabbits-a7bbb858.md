---
title: wastrelly wabbits
url: https://wingolog.org/archives/2026/03/31/wastrelly-wabbits
published: "2026-03-31T20:34:23Z"
feed: wingo
guid: https://wingolog.org/2026/03/31/wastrelly-wabbits
---

# wastrelly wabbits

Good day! Today (tonight), some notes on the last couple months of
[Wastrel](https://codeberg.org/andywingo/wastrel/), my ahead-of-time
WebAssembly compiler.

Back in the beginning of February, I showed [Wastrel running programs\
that use garbage\
collection](https://wingolog.org/archives/2026/02/06/ahead-of-time-wasm-gc-in-wastrel),
using an embedded copy of the [Whippet\
collector](https://github.com/wingo/whippet), specialized to the types
present in the Wasm program. But, the two synthetic GC-using programs I
tested on were just ported microbenchmarks, and didn’t reflect the
output of any real toolchain.

In this cycle I worked on compiling the output from the [Hoot\
Scheme-to-Wasm compiler](https://spritely.institute/hoot/). There were
some interesting challenges!

### bignums

When I originally wrote the Hoot compiler, it targetted the browser,
which already has a bignum implementation in the form of
[BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt#Browser_compatibility),
which [I worked on back in the\
day](https://wingolog.org/archives/2019/05/23/bigint-shipping-in-firefox).
Hoot-generated Wasm files use host bigints via `externref` (though
[wrapped in structs to allow for hashing and\
identity](https://wingolog.org/archives/2023/03/20/a-world-to-win-webassembly-for-the-rest-of-us#:~:text=common%20struct%20supertype)).

In Wastrel, then, I implemented the imports that implement bignum
operations: addition, multiplication, and so on. I did so using
[mini-gmp](https://gmplib.org/repo/gmp/file/tip/mini-gmp/README), a
stripped-down implementation of the workhorse GNU multi-precision
library. At some point if bignums become important, this gives me the
option to link to the full GMP instead.

Bignums were the first managed data type in Wastrel that wasn’t defined
as part of the Wasm module itself, instead hiding behind `externref`, so
I had to add a facility to [allocate type\
codes](https://wingolog.org/archives/2026/02/18/two-mechanisms-for-dynamic-type-checks)
to these “host” data types. More types will come in time: weak maps,
ephemerons, and so on.

I think bignums would be a great proposal for the Wasm standard, similar
to [stringref](https://github.com/WebAssembly/stringref) ideally
( [sniff!](https://wingolog.org/archives/2023/10/19/requiem-for-a-stringref)),
possibly in an [attenuated\
form](https://github.com/WebAssembly/js-string-builtins/blob/main/proposals/js-string-builtins/Overview.md).

### exception handling

Hoot used to emit a [pre-standardization form of exception\
handling](https://github.com/WebAssembly/exception-handling/blob/main/proposals/exception-handling/legacy/Exceptions.md),
and hadn’t gotten around to updating to the [newer\
version](https://wingolog.org/archives/2026/03/10/nominal-types-in-webassembly)
that was standardized last July. I updated Hoot to emit the newer kind
of exceptions, as it was easier to implement them in Wastrel that way.

Some of the problems [Chris Fallin contended with in\
Wasmtime](https://cfallin.org/blog/2025/11/06/exceptions/) don’t apply
in the Wastrel case: since the set of instances is known at
compile-time, we can statically allocate type codes for exception tags.
Also, I didn’t really have to do the back-end: I can just use [`setjmp`\
and `longjmp`](https://man7.org/linux/man-pages/man3/longjmp.3.html).

This whole paragraph was meant to be a bit of an aside in which I
briefly mentioned why just using `setjmp` was fine. Indeed, because
Wastrel never re-uses a temporary, relying entirely on GCC to “re-use”
the register / stack slot on our behalf, I had thought that I didn’t
need to worry about the “volatile problem”. From the C99 specification:

> \[...\] values of objects of automatic storage duration that
> are local to the function containing the invocation of the corresponding
> setjmp macro that do not have volatile-qualified type and have been
> changed between the setjmp invocation and longjmp call are
> indeterminate.

My thought was, though I might set a value between `setjmp` and
`longjmp`, that would only be the case for values whose lifetime did
not reach the `longjmp` (i.e., whose last possible use was before the
jump). Wastrel didn’t introduce any such cases, so I was good.

However, I forgot about `local.set`: mutations of locals (ahem, _objects_
_of automatic storage duration_) in the source Wasm file could run afoul
of this rule. So, because of writing this blog post, I went back and
did an analysis pass on each function to determine the set of locals
which are mutated inside the body of a `try_table`. Thank you, rubber duck readers!

### bugs

Oh my goodness there were many bugs. Lacunae, if we are being generous;
things not implemented quite right, which resulted in errors either when
generating C or when compiling the C. The [type-preserving translation\
strategy](https://wingolog.org/archives/2026/02/09/six-thoughts-on-generating-c)
does seem to have borne fruit, in that I have spent very little time in
GDB: once things compile, they work.

### coevolution

Sometimes Hoot would use a browser facility where it was convenient, but
for which in a better world we would just do our own thing. Such was the
case for the `number->string` operation on floating-point numbers: we
did something [awful but\
expedient](https://codeberg.org/spritely/hoot/src/branch/main/reflect-js/reflect.js#L817-L828).

I didn’t have this facility in Wastrel, so instead we moved to do
float-to-string conversions [in\
Scheme](https://codeberg.org/spritely/hoot/src/branch/main/lib/hoot/write.scm#L75-L208).
This turns out to have been a good test for bignums too; the [algorithm\
we use](https://doi.org/10.1145/231379.231397) is a bit dated and relies
on bignums to do its thing. The move to Scheme also allows for printing
floating-point numbers in other radices.

There are a few more Hoot patches that were inspired by Wastrel, about
which more later; it has been good for both to work on the two at the
same time.

### tail calls

My plan for Wasm’s
[`return_call`](https://webassembly.github.io/spec/core/exec/instructions.html#xref-syntax-instructions-syntax-instr-control-mathsf-return-call-x)
and friends was to use the new `musttail` annotation for calls, which
has been in clang for a while and was recently added to GCC. I was
careful to [limit the number of function\
parameters](https://wingolog.org/archives/2026/02/09/six-thoughts-on-generating-c#:~:text=for%20ABI%20and%20tail%20calls%2C%20perform%20manual%20register%20allocation)
such that no call should require stack allocation, and therefore a
compiler should have no reason to reject any particular tail call.

However, there were bugs. Funny ones, at first: [attributes applying to\
a preceding label instead of the following\
call](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=124533), or [the need\
to insert `if (1)` before the tail\
call](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=124532). More dire
ones, in which [tail callers inlined into _their_ callees would cause\
the tail calls to\
fail](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=124534), worked
around with judicious application of `noinline`. Thanks to GCC’s Andrew
Pinski for help debugging these and other issues; with GCC things are
fine now.

I did have to change the code I emitted to return “top types only”: if
you have a function returning type T, you can tail-call a function
returning U if U is a subtype of T, but there is [no nice way to encode\
this into the C type\
system](https://codeberg.org/andywingo/wastrel/issues/25). Instead, we
return the top type of T (or U, it’s the same), e.g. `anyref`, and
insert downcasts at call sites to recover the precise types. Not so
nice, but it’s what we got.

Trying tail calls on clang, I ran into a funny restriction: clang not
only requires that return types match, but requires that tail caller and
tail callee have the same parameters as well. I can see why they did
this (it requires no stack shuffling and thus such a tail call is always
possible, even with 500 arguments), but it’s not the design point that I
need. Fortunately there are discussions about [moving to a different\
constraint](https://discourse.llvm.org/t/experience-with-clang-musttail/89085/7).

### scale

I spent way more time that I had planned to on improving the speed of
Wastrel itself. My initial idea was to just emit one big C file, and
that would provide the maximum possibility for GCC to just go and do its
thing: it can see everything, everything is static, there are loads of
`always_inline` helpers that should compile away to single instructions,
that sort of thing. But, this doesn’t scale, in a few ways.

In the first obvious way, consider [whitequark’s\
`llvm.wasm`](https://mastodon.social/@wingo/115378481224538009). This
is all of LLVM in one 70 megabyte Wasm file. Wastrel made a huuuuuuge C
file, then GCC chugged on it forever; 80 minutes at `-O1`, and I wasn’t
aiming for `-O1`.

I realized that in many ways, GCC wasn’t designed to be a compiler
target. The shape of code that one might emit from a Wasm-to-C compiler
like Wastrel is different from that that one would write by hand. I
even ran into a [segfault compiling with\
-Wall](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=124651), because GCC
accidentally recursed instead of iterated in the `-Winfinite-recursion`
pass.

So, I dealt with this in a few ways. After many hours spent pleading
and bargaining with different `-O` options, I bit the bullet and made
Wastrel emit multiple C files. It will compute a DAG forest of all the
functions in a module, where edges are direct calls, and go through that
forest, [greedily consuming (and possibly splitting) subtrees until we\
have “enough” code to split out a partition](https://codeberg.org/andywingo/wastrel/src/branch/main/module/wastrel/partitions.scm#L436), as measured by number of
Wasm instructions. They say that `-flto` makes this a fine approach,
but one never knows when a translation unit boundary will turn out to be
important. I compute needed symbol visibilities as much as I can so as
to declare functions that don’t escape their compilation unit as
`static`; who knows if this is of value. Anyway, this partitioning
introduced no performance regression in my limited tests so far, and
compiles are much much much faster.

### scale, bis

A brief observation: Wastrel used to emit indented code, because it
could, and what does it matter, anyway. However, consider Wasm’s
[`br_table`](https://webassembly.github.io/spec/core/valid/instructions.html#xref-syntax-instructions-syntax-instr-control-mathsf-br-table-l-ast-l-n):
it takes an array of _n_ labels and an integer operand, and will branch
to the _n_ th label, or the last if the operand is out of range. To set
up a label in Wasm, you make a block, of which there are a handful of
kinds; the label is visible in the block, and for _n_ labels, the
`br_table` will be the most nested expression in the _n_ nested blocks.

Now consider that block indentation is proportional to _n_. This means,
the file size of an indented C file is quadratic in the number of branch
targets of the `br_table`.

Yes, this actually bit me; there are `br_table` instances with tens of
thousands of targets. No, wastrel does not indent any more.

### scale, ter

Right now, the long pole in Wastrel is the compile-to-C phase; the
C-to-native phase parallelises very well and is less of an issue. So,
one might think: OK, you have partitioned the functions in this Wasm
module into a number of files, why not emit the files in parallel?

I gave this a go. It did not speed up C generation. From my cursory
investigations, I think this is because the bottleneck is garbage
collection in Wastrel itself; Wastrel is written in Guile, and Guile
still uses the Boehm-Demers-Weiser collector, which does not parallelize
well for multiple mutators. It’s terrible but I ripped out
parallelization and things are fine. [Someone on Mastodon suggested\
`fork`](https://jorts.horse/@migratory/116279275172476524); they’re not
wrong, but also not Right either. I’ll just keep this as a nice test
case for the Guile-on-Whippet branch I want to poke later this year.

### scale, quator

Finally, I had another realization: GCC was having trouble compiling the
C that Wastrel emitted, because Hoot had emitted bad WebAssembly. Not
bad as in “invalid”; rather, “not good”.

There were two cases in which Hoot emitted ginormous (technical term)
functions. One, for an odd debugging feature: Hoot does a CPS transform
on its code, and allocates return continuations on a stack. This is a
gnarly technique but it gets us delimited continuations and all that
goodness even before stack switching has landed, so it’s here for now.
It also gives us a reified return stack of `funcref` values, which lets
us print Scheme-level backtraces.

Or it would, if we could associate data with a `funcref`. Unfortunately
`func` is not a subtype of `eq`, so we can’t. Unless... we pass the
funcref out to the embedder (e.g. JavaScript), and the embedder checks
the funcref for equality (e.g. using `===`); then we can map a funcref
to an index, and use that index to map to other properties.

How to pass that `funcref`/index map to the host? When I initially
wrote Hoot, I didn’t want to just, you know, put the funcrefs of interet
into a table and let the index of a function’s slot be the value in the
key-value mapping; that would be useless memory usage. Instead, we
emitted functions that took an integer, and which would return a
funcref. Yes, these used `br_table`, and yes, there could be tens of
thousands of cases, depending on what you were compiling.

Then to map the integer index to, say, a function name, likewise I
didn’t want a table; that would force eager allocation of all strings.
Instead I emitted a function with a `br_table` whose branches would
return `string.const` values.

Except, of course, [stringref didn’t become a\
thing](https://wingolog.org/archives/2023/10/19/requiem-for-a-stringref),
and so instead we would end up lowering to allocate string constants as
globals.

Except, of course, Wasm’s idea of what a “constant” is is quite
restricted, so [we have a pass that moves non-constant global\
initializers to the “start”\
function](https://codeberg.org/spritely/hoot/src/branch/main/module/wasm/lower-globals.scm).
This results in an enormous start function. The straightforward
solution was to partition global initializations into separate
functions, called by the start function.

For the `funcref` debugging, the solution was more intricate: firstly,
we represent the `funcref`-to-index mapping just as a table. It’s fine.
Then for the side table mapping indices to function names and sources,
we emit DWARF, and attach a special attribute to each “introspectable”
function. In this way, reading the DWARF sequentially, we reconstruct a
mapping from index to DWARF entry, and thus to a byte range in the Wasm
code section, and thus to source information in the `.debug_line`
section. It sounds gnarly but Guile already used DWARF as its own
debugging representation; switching to emit it in Hoot was not a huge
deal, and as we only need to consume the DWARF that we emit, we only
needed some [400 lines of\
JS](https://codeberg.org/spritely/hoot/src/branch/main/reflect-js/reflect.js#L3-L377)
for the web/node run-time support code.

This switch to data instead of code removed the last really long pole
from the GCC part of Wastrel’s pipeline. What’s more, Wastrel can now
implement the `code_name` and `code_source` imports for Hoot programs
ahead of time: it can parse the DWARF at compile-time, and generate
functions that look up functions by address in a sorted array to return
their names and source locations. As of today, this works!

### fin

There are still a few things that Hoot wants from a host that Wastrel
has stubbed out: weak refs and so on. I’ll get to this soon; my goal is
a proper Scheme REPL. Today’s note is a waypoint on the journey. Until
next time, happy hacking!
