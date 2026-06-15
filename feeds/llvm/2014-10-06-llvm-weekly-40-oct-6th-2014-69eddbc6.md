---
title: 'LLVM Weekly - #40, Oct 6th 2014'
url: https://blog.llvm.org/2014/10/llvm-weekly-40-oct-6th-2014.html
published: "2014-10-06T05:49:00Z"
feed: llvm
guid: https://blog.llvm.org/2014/10/llvm-weekly-40-oct-6th-2014.html
---

# LLVM Weekly - #40, Oct 6th 2014

Welcome to the fortieth issue of LLVM Weekly, a weekly newsletter (published every Monday) covering developments in LLVM, Clang, and related projects. LLVM Weekly is brought to you by [Alex Bradbury](http://asbradbury.org). Subscribe to future issues at [http://llvmweekly.org](http://llvmweekly.org) and pass it on to anyone else you think may be interested. Please send any tips or feedback to [asb@asbradbury.org](mailto:asb@asbradbury.org), or [@llvmweekly](https://twitter.com/llvmweekly) or [@asbradbury](https://twitter.com/asbradbury) on Twitter.

I'll be in Munich next weekend for the [OpenRISC conference](http://tum-lis.github.io/orconf2014/) where I'll be presenting on the [lowRISC](http://lowrisc.org/) project to produce an open-source SoC. I'll be giving a similar talk in London at the Open Source Hardware User Group [on 23rd October](http://oshug.org/event/36).

The canonical home for this issue [can be found here at llvmweekly.org](http://llvmweekly.org/issue/40).

### News and articles from around the web

Capstone 3.0 RC1 [has been released](http://capstone-engine.org/Version-3.0-RC1.html) Capstone is an open source disassembly engine, based initially on code from LLVM. This release features support for Sparc, SystemZ and XCore as well as the previously supported architectures. Among other changes, the Python bindings are now compatible with Python 3.

An interesting paper from last year came up on the mailing list. From EPFL, it proposes [adding -OVERIFY to optimise programs for fast verification](http://dslab.epfl.ch/pubs/overify.pdf). The performance of symbolic execution tools is improved by reducing the number of paths to explore and the complexity of branch conditions. They managed a maximum 95x reduction in total compilation and analysis time.

The next Cambridge (UK) social [will take place on Wed 8th Oct at 7.30 pm](http://article.gmane.org/gmane.comp.compilers.llvm.devel/77453).

### On the mailing lists

- Reid Kleckner has posted an RFC on [approaches to representing structured exception handling (SEH) in LLVM IR](http://article.gmane.org/gmane.comp.compilers.clang.devel/39152). This is the exception handling model used on Windows.

- Chandler Carruth has written to the list to [announce his new x86 vector shuffle lowering path is now enabled by default](http://article.gmane.org/gmane.comp.compilers.llvm.devel/77439). This code path has seen extensive fuzz testing. The performance improvement is largest on AMD chips with older SSE versions. If anyone is able to find a performance regression, you are encouraged to report it.

- Richard Pennington who maintains the Clang/LLVM ELLCC cross-development toolchain is [considering dropping support for Microblaze](http://article.gmane.org/gmane.comp.compilers.llvm.devel/77431). The Microblaze backend was dropped from LLVM last year, but Richard has been maintaining it out of tree. However there seems to be very little actual interest. If somebody wants to pick it up, now is the time to jump in.


### LLVM commits

- The expansion of atomic loads/stores for PowerPC has been improved. [r218922](http://reviews.llvm.org/rL218922). The documentation on atomics has also been updated. [r218937](http://reviews.llvm.org/rL218937).

- For the past few weeks, Chandler Carruth has been working on a new vector shuffle lowering implementation. There have been too many commits to summarise, but the time has come and the new codepath is now enabled by default. It claims 5-40% improvements in the right conditions (when the loop vectorizer fires in the hot path for SSE2/SSE3). [r219046](http://reviews.llvm.org/rL219046).

- The Cortex-A57 scheduling model has been refined. [r218627](http://reviews.llvm.org/rL218627).

- SimplifyCFG now has a configurable threshold for folding branches with common destination. Changing this threshold can be worthwhile for GPU programs where branches are expensive. [r218711](http://reviews.llvm.org/rL218711).

- Basic support for the newly-announced Cortex-M7 has been added. [r218747](http://reviews.llvm.org/rL218747).

- As discussed on the mailing list last week, the sqrt intrinsic will now return undef when given a negative input. [r218803](http://reviews.llvm.org/rL218803).

- llvm-readobj learnt `-coff-imports` which will print out the COFF import table. [r218891](http://reviews.llvm.org/rL218891), [r218915](http://reviews.llvm.org/rL218915).


### Clang commits

- Support for the `align_value` attribute has been added, matching the behaviour of the attribute in the Intel compiler. The commit message explains why this attribute is useful in addition to `aligned`. [r218910](http://reviews.llvm.org/rL218910).

- A rather useful diagnostic has been added. `-Winconsistent-missing-override` will warn if override is missing on an overridden method if that class has at least one override specified on its methods. [r218925](http://reviews.llvm.org/rL218925).

- Support for MS ABI continues. `thread_local` is now supported for global variables. [r219074](http://reviews.llvm.org/rL219074).

- Matcher and DynTypedMatcher saw some nice performance tweaking, resulting in a 14% improvement on a clang-tidy benchmark and compilation of Dynamic/Registry.cpp sped up by 17%. [r218616](http://reviews.llvm.org/rL218679).

- lifetime.start and lifetime.end markers are now emitted for unnamed temporary objects. [r218865](http://reviews.llvm.org/rL218865).

- The `__sync_fetch_and_nand` intrinsic was re-added. See the commit message for a history of its removal. [r218905](http://reviews.llvm.org/rL218905).

- Clang gained its own implementation of C11 `stdatomic.h`. The system header will be used in preference if present. [r218957](http://reviews.llvm.org/rL218957).

- Clang now understands `-mthread-model` to specify the thread model to use, e.g. posix, single (for bare-metal and single-threaded targets). [r219027](http://reviews.llvm.org/rL219027).


### Other project commits

- libcxxabi should now work with the ARM Cortex-M0. [r218869](http://reviews.llvm.org/rL218869).

- lldb gained initial support for scripting stepping. This is the ability to add new stepping modes implemented by python classes. The example in the follow-on commit has a large comment at the head of the file to explain its operation. [r218642](http://reviews.llvm.org/rL218642), [r218650](http://reviews.llvm.org/rL218650).
