---
title: 'LLVM Weekly - #73, May 25th 2015'
url: https://blog.llvm.org/2015/05/llvm-weekly-73-may-25th-2015.html
published: "2015-05-25T09:39:00Z"
feed: llvm
guid: https://blog.llvm.org/2015/05/llvm-weekly-73-may-25th-2015.html
---

# LLVM Weekly - #73, May 25th 2015

Welcome to the seventy-third issue of LLVM Weekly, a weekly newsletter (published every Monday) covering developments in LLVM, Clang, and related projects. LLVM Weekly is brought to you by [Alex Bradbury](http://asbradbury.org). Subscribe to future issues at [http://llvmweekly.org](http://llvmweekly.org) and pass it on to anyone else you think may be interested. Please send any tips or feedback to [asb@asbradbury.org](mailto:asb@asbradbury.org), or [@llvmweekly](https://twitter.com/llvmweekly) or [@asbradbury](https://twitter.com/asbradbury) on Twitter.

The canonical home for this issue [can be found here at llvmweekly.org](http://llvmweekly.org/issue/73).

### News and articles from around the web

The LLVM blog has properly [announced](http://blog.llvm.org/2015/05/openmp-support_22.html) full support for OpenMP 3.1 in Clang.

The Clang-derived [Zapcc](http://www.zapcc.com/) has had [some attention](https://news.ycombinator.com/item?id=9592601) this week. It claims higher compilation speeds than the baseline Clang or other compilers. Yaron Keren, the principal developer has shared [many more details about its implementation](http://article.gmane.org/gmane.comp.compilers.clang.devel/42950) on the Clang mailing list.

### On the mailing lists

- The discussion about upstreaming the LLVM/SPIR-V converter has continued. Chandler Carruth has [responded with feedback](http://article.gmane.org/gmane.comp.compilers.llvm.devel/86000), and Philip Reames has [shared his concerns](http://article.gmane.org/gmane.comp.compilers.llvm.devel/86054) about the merge proposal as-is. Neil Henning has [responded to some of these concerns](http://article.gmane.org/gmane.comp.compilers.llvm.devel/86056).

- Adam Nemet has kicked off a thread about [alias-based loop versioning](http://article.gmane.org/gmane.comp.compilers.llvm.devel/86024), with the hope that others working in the area can chime in.

- Félix Cloutier queries why [MemoryDependencyAnalysis reports dependencies between NoAlias pointers](http://article.gmane.org/gmane.comp.compilers.llvm.devel/86029). Daniel Berlin points to his very interesting looking work on [MemorySSA](http://reviews.llvm.org/D7864).

- Duncan P.N. Exon Smith has posted an RFC on [reducing the memory footprint of debug info entries](http://article.gmane.org/gmane.comp.compilers.llvm.devel/85981). The attached patches reduce peak memory usage from 920MB to 884MB for the tested workload.

- John Criswell has a helpful answer regarding [how to determine whether a branch instruction may depend on function parameters](http://article.gmane.org/gmane.comp.compilers.llvm.devel/85987).

- Andrew Kaylor has shared a [detailed description of the work to be done for exception handling on Windows](http://article.gmane.org/gmane.comp.compilers.llvm.devel/85926).

- Andrew Bokhanko is looking for feedback on [adding an option to control a level of OpenMP support in Clang](http://article.gmane.org/gmane.comp.compilers.clang.devel/42622). Now that 3.1 support is complete, OpenMP 4.0 is the next target but this is likely to remain incomplete for some time. The question is whether those features which are implemented are available by default, or whether users should opt-in with a compiler flag while support remains incomplete.


### LLVM commits

- The `dereferenceable_or_null` attribute will now be exploited by the loop environment code motion pass. [r237593](http://reviews.llvm.org/rL237593).

- Commits have started on the 'MIR serialization' project, which aims to print machine functions in a readable format. [r237954](http://reviews.llvm.org/rL237954).

- A GCStrategy for CoreCLR has been committed alongside some documentation for it. [r237753](http://reviews.llvm.org/rL237753), [r237869](http://reviews.llvm.org/rL237869).

- libFuzzer gained some more documentation. [r237836](http://reviews.llvm.org/rL237836).

- libFuzzer can now be used with user-supplied mutators. [r238059](http://reviews.llvm.org/rL238059), [r238062](http://reviews.llvm.org/rL238062).


### Clang commits

- `-fopenmp` will turn on OpenMP support and link with libiomp5 (libgomp can alternatively be specified). [r237769](http://reviews.llvm.org/rL237769).

- The `-mrecip` flag has been added to match GCC. [r238055](http://reviews.llvm.org/rL238055).


### Other project commits

- C++1z status for libcxx has been updated. [r237606](http://reviews.llvm.org/rL237606).

- `std::bool_constant` and `uninitialized_copy()` was added to libcxx. [r237636](http://reviews.llvm.org/rL237636), [r237699](http://reviews.llvm.org/rL237699).

- libcxx gained a TODO list. Plenty of tasks that might be interesting to new contributors. [r237813](http://reviews.llvm.org/rL237813), [r237988](http://reviews.llvm.org/rL237988).

- LDB has enabled debugging of multithreaded programs on Windows and gained support for attaching to process. [r237637](http://reviews.llvm.org/rL237637), [r237817](http://reviews.llvm.org/rL237817).
