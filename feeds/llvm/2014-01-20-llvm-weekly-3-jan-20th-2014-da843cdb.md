---
title: 'LLVM Weekly - #3, Jan 20th 2014'
url: https://blog.llvm.org/2014/01/llvm-weekly-3-jan-20th-2014.html
published: "2014-01-20T07:42:00Z"
feed: llvm
guid: https://blog.llvm.org/2014/01/llvm-weekly-3-jan-20th-2014.html
---

# LLVM Weekly - #3, Jan 20th 2014

Welcome to the third issue of LLVM Weekly, a weekly newsletter (published every Monday) covering developments in LLVM, Clang, and related projects. LLVM Weekly is brought to you by [Alex Bradbury](http://asbradbury.org). Subscribe to future issues at [http://llvmweekly.org](http://llvmweekly.org) and pass it on to anyone else you think may be interested. Please send any tips or feedback to [asb@asbradbury.org](mailto:asb@asbradbury.org), or [@llvmweekly](https://twitter.com/llvmweekly) or [@asbradbury](https://twitter.com/asbradbury) on Twitter.

The canonical home for this issue [can be found here at llvmweekly.org](http://llvmweekly.org/issue/3).

### News and articles from around the web

Eli Bendersky has penned [some thoughts on LLVM vs. libjit](http://eli.thegreenplace.net/2014/01/15/some-thoughts-on-llvm-vs-libjit/). Eli describes libjit as being more limited, yet easier to understand and to get going with due to its focus. He also makes interesting claims such as "to be honest, I don't think it's possible to create a really fast JIT within the framework of LLVM, because of its modularity. The faster the JIT, the more you’ll have to deviate from the framework of LLVM". As well as the comments directly on the blog post, there is some good discussion over [at Reddit](http://www.reddit.com/r/programming/comments/1vf1lx/some_thoughts_on_llvm_vs_libjit/) .

[Version 2.0-RC1 Capstone disassembly framework has been released](http://www.capstone-engine.org/Version-2.0-RC1.html). Capstone is built using code from LLVM. The new release features reduced memory usage, faster Python bindings, and support for PowerPC among other changes.

[Planet Clang](http://planet.clang.org/) has been [announced](http://article.gmane.org/gmane.comp.compilers.clang.devel/34439). It is a news feed following blog posts from Clang and LLVM committers and contributors. The blog roll is fairly short right now, but you're welcome to submit your RSS feed via the email address in the announcement post.

The PDF of an upcoming paper to be presented at CGO next month has been released. [WatchdogLite: Hardware-Accelerated Compiler-Based Pointer Checking](http://www.cs.rutgers.edu/~santosh.nagarakatte/papers/cgo2014-final.pdf) proposes instruction set extensions to accelerate pointer checking functions and achieves a performance overhead of 29% in return for memory safety. The compiler extends (and is compared to) [SoftBound + CETS](http://acg.cis.upenn.edu/softbound/).

### On the mailing lists

- David Woodhouse has posted a [detailed update on the status of 16-bit x86 in LLVM](http://article.gmane.org/gmane.comp.compilers.llvm.devel/69440). David has successfully built the 16-bit startup code of the Linux kernel and invites people to start testing it on real code.

- Tom Stellard opens a discussion on [stable LLVM 3.4.x releases](http://article.gmane.org/gmane.comp.compilers.llvm.devel/69428). A number of people volunteer their assistance and there seems to be general agreement that any 3.4.1 release would include bug-fixes only with no ABI changes.

- Diego Novillo is looking to boost the performance of the SPEC benchmark libquantum [using profile info and loop unrolling](http://article.gmane.org/gmane.comp.compilers.llvm.devel/69494). Sean Silva did us all a great service by asking for clarification on what a "runtime unroller" means in this context. The answer is that the trip count (the number of times the loop is executed) is not known at compile time. The thread is worth a read if you're interested in loop unrolling or vectorization.

- Aaron Ballman has stepped up as [code owner for the attribute subsystem](http://article.gmane.org/gmane.comp.compilers.clang.scm/90431) with unanimous approval.

- Skye Wanderman-Milne was looking for help on [loop unrolling a single function using the C++ API](http://article.gmane.org/gmane.comp.compilers.llvm.devel/69470). Simply adding the LoopUnrollPass to a FunctionPassManager had no effect, but after some advice from the mailing list Skye did respond to confirm that the set of ScalarReplAggregates, LoopRotate, and LoopUnroll passes did have the desired effect.

- Tobias Grosser asks why LLVM's LNT (used for performance tracking) defaults to [aggregating results by taking the minimum rather than an average](http://article.gmane.org/gmane.comp.compilers.llvm.devel/69555). Replies quickly hone on in the real problem at hand, which is that results are 'noisy' potentially due to other processes on the machine but also quantised to certain values due to the timer being relatively coarse-grained in comparison to the execution time for the benchmarks.

- This week's unsolved question is from Keith Walker, who's noticed that on ARM, the function prologue generated in GCC and LLVM ends up with [the frame register pointing to a different address](http://article.gmane.org/gmane.comp.compilers.llvm.devel/69514). The LLVM prologue results in the frame pointer pointing to just after the pushed r11 register (the saved frame pointer) while on GCC the frame pointer points to just after the pushed link register. The difference makes it difficult to produce a generic stack walker.


### LLVM commits

- The MCJIT remote execution protocol was heavily refactored and it was hoped fixed on ARM where it was previously non-functional. There are still some random failures on ARM though, see [bug 18507](http://llvm.org/bugs/show_bug.cgi?id=18057). [r199261](http://reviews.llvm.org/rL199261)

- The cutoff on when to attempt to convert a switch to a lookup table was changed from 4 to 3. Experimentally, Hans Wennborg found that there was no speedup for two cases but three produced a speedup. When building Clang, this results in 480 new switches to be transformed and an 8KB smaller binary size. [r199294](http://reviews.llvm.org/rL199294)

- Support for the `preserve_mostcc` and `preserve_allcc` calling conventions was introduced and implemented for x86-64. These are intend to be used by a future version of the ObjectiveC runtime in order to reduce overhead of runtime calls. [r199508](http://reviews.llvm.org/rL199508)

- The configure script now checks for a sufficiently modern host compiler (Clang 3.1 or GCC 4.7) [r199182](http://reviews.llvm.org/rL199182)

- More work on the new PassManager driver. Bitcode can now be written using the new PM and more preparation/cleanup work has been performed. [r199078](http://reviews.llvm.org/rL199078), [r199095](http://reviews.llvm.org/rL199095), [r199104](http://reviews.llvm.org/rL199104)

- Dominators.h and Verifier.h moved from the Analysis directory to the IR directory. [r199082](http://reviews.llvm.org/rL199082)

- The DAGCombiner learned to reassociate (i.e. change the order of) vector operations [r199135](http://reviews.llvm.org/rL199135)

- dllexport and dllimport are no longer represented as linkage types [r199218](http://reviews.llvm.org/rL199218)

- Parsing of the .symver directive in ARM assembly was fixed [r199339](http://reviews.llvm.org/rL199339)


### Clang commits

- The MS ABI is now used for Win32 targets by default [r199131](http://reviews.llvm.org/rL199131)

- The MicrosoftMode language option was renamed to MSVCCompat and its role clarified (see the commit message for a description of MicrosoftExt vs MSVCCompat). [r199209](http://reviews.llvm.org/rL199209)

- The `-cxx-abi` command-line flag was killed and is instead inferred depending on the target. [r199250](http://reviews.llvm.org/rL199250)

- The analyzer learned that shifting a constant value by its bit width is undefined. [r199405](http://reviews.llvm.org/rL199405)

- The `nonnull` attribute can now be applied to parameters directly. [r199467](http://reviews.llvm.org/rL199467)

- Support for AArch64 on NetBSD was added to the compiler driver. [r199124](http://reviews.llvm.org/rL199124)


### Other project commits

- AddressSanitizer in compiler-rt gained the ability to start in 'deactivated' mode. It can later be activated when `__asan_init` is called in an instrumented library. [r199377](http://reviews.llvm.org/rL199377)

- A number of patches were committed to lld for better MIPS support. [r199231](http://reviews.llvm.org/rL199231) and many more.

- lldb recognises Linux distribution in the vendor portion of the host triple. e.g. `x86_64-ubuntu-linux-gnu`. [r199510](http://reviews.llvm.org/rL199510)
