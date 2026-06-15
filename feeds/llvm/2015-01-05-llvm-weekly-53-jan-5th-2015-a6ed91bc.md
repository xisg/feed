---
title: 'LLVM Weekly - #53, Jan 5th 2015'
url: https://blog.llvm.org/2015/01/llvm-weekly-53-jan-5th-2015.html
published: "2015-01-05T06:30:00Z"
feed: llvm
guid: https://blog.llvm.org/2015/01/llvm-weekly-53-jan-5th-2015.html
---

# LLVM Weekly - #53, Jan 5th 2015

Welcome to the fifty-third issue of LLVM Weekly, a weekly newsletter (published every Monday) covering developments in LLVM, Clang, and related projects. LLVM Weekly is brought to you by [Alex Bradbury](http://asbradbury.org). Subscribe to future issues at [http://llvmweekly.org](http://llvmweekly.org) and pass it on to anyone else you think may be interested. Please send any tips or feedback to [asb@asbradbury.org](mailto:asb@asbradbury.org), or [@llvmweekly](https://twitter.com/llvmweekly) or [@asbradbury](https://twitter.com/asbradbury) on Twitter.

I'm going to be in California next week for the [RISC-V workshop](http://riscv.org/workshop). I'm arriving at SFO on Monday 12th and leaving on Sunday the 18th. Do let me know if you want to meet and talk [lowRISC](http://www.lowrisc.org)/RISC-V or LLVM, and we'll see what we can do.

The canonical home for this issue [can be found here at llvmweekly.org](http://llvmweekly.org/issue/53).

### News and articles from around the web

I was getting ready to break out gitstats for some analysis of the LLVM repo and I find to my delight that Phoronix has saved me the trouble and has [shared some stats on activity in the LLVM repo over the past year](http://www.phoronix.com/scan.php?page=news_item&px=MTg3OTA).

Tom Stellard has made a blog post [announcing some recent RadeonSI performance improvements](http://www.stellard.net/tom/blog/?p=69) on his LLVM development branch. This includes 60% improvement in one OpenCL benchmark and 10-25% in a range of other OpenCL tests.

Gaëtan Lehmann has written a blog post about [getting started with libclang using the Python bindings](http://blog.glehmann.net/2014/12/29/Playing-with-libclang/).

The C++ Filesystem Technical Specification, based on the Boost.Filesystem library [has been approved](http://article.gmane.org/gmane.comp.lib.boost.devel/256220).

### On the mailing lists

- Virgile Bello has some questions on [how he can control the calling convention in LLVM](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80215). In this case, he has an CLR frontend and is trying to pass an object on the CLR stack to a native Win32 function. Reid Kleckner [suggests](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80217) the best way may be to just link with Clang and use its implementation. In another followup, he links to the [talk on this topic](http://llvm.org/devmtg/2014-10/Slides/Skip%20the%20FFI.pdf) at the last LLVM dev meeting.

- [Is anybody using the ModuleBuilder class in Clang?](http://article.gmane.org/gmane.comp.compilers.clang.devel/40426). If so, now is the time to speak up as it's slated to be removed.

- Sami Liedes has [set up a new bot to test Clang with fuzzed inputs](http://article.gmane.org/gmane.comp.compilers.clang.devel/40474). A report from the bot is available [here](http://sli.dy.fi/~sliedes/clang-triage/triage_report.xhtml) and the code is [here](https://github.com/sliedes/clang-triage).

- The release of LLVM/Clang 3.5.1 may be slightly delayed due to the addition of new patches late in the process. Chandler Carruth [points out](http://article.gmane.org/gmane.comp.compilers.llvm.devel/80202) that there are some unpleasant bugs in InstCombine in the current 3.5.1 release candidate. If there is a release candidate 3, the patch in question will definitely make it in.


### LLVM commits

- Instruction selection for bit-permuting operations on PowerPC has been improved. [r225056](http://reviews.llvm.org/rL225056).

- The scalar replacement of aggregates (SROA) pass has started to learn how to more intelligently handle split loads and stores. As explained in detail in the commit message, the old approach lead to complex IR that can be difficult for the optimizer to work with. SROA is now also more aggressive in its splitting of loads. [r225061](http://reviews.llvm.org/rL225061), [r225074](http://reviews.llvm.org/rL225074).

- InstCombine will now try to transform `A-B < 0` in to `A < B`. [r225034](http://reviews.llvm.org/rL225034).

- The Hexagon (a Qualcomm DSP) backend has seen quite a lot of work recently. Interested parties are best of flicking through the commit log of lib/Target/Hexagon. [r225005](http://reviews.llvm.org/rL225005), [r225006](http://reviews.llvm.org/rL225006), etc.


### Clang commits

- More crash bugs have been uncovered and fixed by the [naive fuzzing technique](http://article.gmane.org/gmane.comp.compilers.llvm.devel/79491) previously covered in LLVM Weekly. e.g. [r224915](http://reviews.llvm.org/rL224915).

### Other project commits

- The lldb website has been updated with more information about LLDB on windows, including [build instructions](http://lldb.llvm.org/build.html#BuildingLldbOnWindows). [r225023](http://reviews.llvm.org/rL225023).
