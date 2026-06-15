---
title: 'LLVM Weekly - #19, May 12th 2014'
url: https://blog.llvm.org/2014/05/llvm-weekly-19-may-12th-2014.html
published: "2014-05-12T06:47:00Z"
feed: llvm
guid: https://blog.llvm.org/2014/05/llvm-weekly-19-may-12th-2014.html
---

# LLVM Weekly - #19, May 12th 2014

Welcome to the ninteenth issue of LLVM Weekly, a weekly newsletter (published every Monday) covering developments in LLVM, Clang, and related projects. LLVM Weekly is brought to you by [Alex Bradbury](http://asbradbury.org). Subscribe to future issues at [http://llvmweekly.org](http://llvmweekly.org) and pass it on to anyone else you think may be interested. Please send any tips or feedback to [asb@asbradbury.org](mailto:asb@asbradbury.org), or [@llvmweekly](https://twitter.com/llvmweekly) or [@asbradbury](https://twitter.com/asbradbury) on Twitter.

I'm flying out to San Francisco tomorrow and will be there for the Bay Area Maker Faire at the weekend with some other Raspberry Pi Foundation people. If you're around, be sure to say hi.

The canonical home for this issue [can be found here at llvmweekly.org](http://llvmweekly.org/issue/19).

### News and articles from around the web

LLVM 3.4.1 [has been released](http://blog.llvm.org/2014/05/llvm-341-release.html). This is a bug-fix release so offers API and ABI compatibility with LLVM 3.4. Thanks to everyone who contributed to the release by suggesting or backporting patches, and for testing.

John Regehr has [shared some early results and discussion](http://blog.regehr.org/archives/1146) on using [Souper](https://github.com/google/souper/) (a new superoptimizer for LLVM IR) in combination with Csmith and C-reduce in order to find missed optimisations and then produce minimal test cases. This has already resulted in a [new performance bug being filed](http://llvm.org/bugs/show_bug.cgi?id=19711) with I'm sure many more to come.

[Crange](https://github.com/crange/crange), a tool to index and cross-reference C/C++ source code built on top of Clang has [been released](http://article.gmane.org/gmane.comp.compilers.clang.devel/36677). It aims to offer a more complete database than e.g. ctags, though the running time on a large codebase like the Linux kernel is currently very high.

[llgo](https://github.com/go-llvm/llgo), the LLVM-based compiler for Go is [now self-hosting](https://groups.google.com/forum/#!topic/llgo-dev/8uBfmIkGM88/discussion).

Last week I asked for benchmarks of the new [JavascriptCore Fourth Tier LLVM JIT](https://trac.webkit.org/wiki/FTLJIT). Arewefastyet from Mozilla now includes such results. FTLJIT [does particularly well on asm.js examples](http://arewefastyet.com/#machine=12&view=breakdown&suite=asmjs-apps).

### On the mailing lists

- The merge of AArch64 in to ARM64 has [made impressive progress](http://article.gmane.org/gmane.comp.compilers.llvm.devel/72832). It's likely the final switchover will take place this week, with AArch64 being deleted and ARM64 renamed to AArch64. Ana Pazos [shared a list of bugs currently waiting to be squashed](http://article.gmane.org/gmane.comp.compilers.llvm.cvs/187546).

- Andy Lutomirski [requests that the llvm integer arithmetic with overflow intrinsics be exposed as C intrinsics](http://article.gmane.org/gmane.comp.compilers.clang.devel/36685). It was pointed out that they [actually are exposed](http://clang.llvm.org/docs/LanguageExtensions.html#checked-arithmetic-builtins) though Google is doing a poor job of indexing this fact, and in response a [bug has been filed with GCC](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=61129) to expose similar intrinsics.

- Suminda Dharmasena [kicked off a conversation about reducing or removing platform specific LLVM instructions](http://article.gmane.org/gmane.comp.compilers.llvm.devel/72850). The thread morphed into an interesting discussion about [the issues arising from gaps in LLVM's coverage for atomic memory operations](http://article.gmane.org/gmane.comp.compilers.llvm.devel/72866).

- Chandler Carruth [asks for input on the problem of safely suspending JITting threads through the C LLVM API](http://article.gmane.org/gmane.comp.compilers.llvm.devel/72847).


### LLVM commits

- A new algorithm has been implemented for tail call marking. A build of clang now ends up with 470k calls in the IR marked as tail vs 375k before. The total tail call to loop conversions remains the same though. [r208017](http://reviews.llvm.org/rL208017).

- `llvm::function_ref` has been introduced and described in the LLVM programmers manual. It is a type-erased reference to a callable object. [r208025](http://reviews.llvm.org/rL208025), [r208067](http://reviews.llvm.org/rL208067).

- Initial support for named register intrinsics (as previously discussed [on the mailing list](http://article.gmane.org/gmane.comp.compilers.llvm.devel/71567) has landed. Right now, only the stack pointer is supported. Other non-allocatable registers could be supported with not too much difficulty, allocatable registers are much harder. [r208104](http://reviews.llvm.org/rL208104).

- The `-disable-cfi` option has been removed. LLVM now requires assemblers to support cfi (control-flow integrity) directives in order to generate stack unwinding information. [r207979](http://reviews.llvm.org/rL207979).

- The superword-level parallelism (SLP) pass is now enabled by default for link time optimisation. [r208013](http://reviews.llvm.org/rL208013).

- The llvm-cov documentation has been expanded [r208098](http://reviews.llvm.org/rL208098).

- The second and third patch of a series to improve MergeFunctions performance to `O(n*log(n))` has been merged. [r208173](http://reviews.llvm.org/rL208173), [r208189](http://reviews.llvm.org/rL208189).

- The standard 'x86-64' CPU used as the default architecture now uses the Sandy Bridge scheduling model in the hope this provides a reasonable default over a wide range of modern x86-64 CPUs. [r208230](http://reviews.llvm.org/rL208230).

- Custom lowering for the `llvm.{u|s}add.with.otherflow.i32` intrinsics as been added for ARM. [r208435](http://reviews.llvm.org/rL208435).


### Clang commits

- MSVC ABI compatibility has again been improved. Clang now understands that the 'sret' (a structure return pointer) is passed after 'this' for MSVC. [r208458](http://reviews.llvm.org/rL208458).

- Initial codegen from OpenMP's `#pragma omp parallel` has landed. [r208077](http://reviews.llvm.org/rL208077).

- Field references to struct names and C++11 aliases are now supported from inline asm. [r208053](http://reviews.llvm.org/rL208053).

- Parsing and semantic analysis has been implemented for the OpenMP `proc_bind` clause. [r208060](http://reviews.llvm.org/rL208060).

- clang-format gained initial support for JavaScript regex literals (yes, clang-format can reformat your JavaScript!). [r208281](http://reviews.llvm.org/rL208281).


### Other project commits

- libcxxabi gained support for ARM zero-cost exception handling. [r208466](http://reviews.llvm.org/rL208466).

- In libcxx, std::vector gained Address Sanitizer support. [r208319](http://reviews.llvm.org/rL208319).

- The test suite from [OpenUH](http://web.cs.uh.edu/~openuh/) has been added to the openmp repository. [208472](http://reviews.llvm.org/rL208472).
