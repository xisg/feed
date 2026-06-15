---
title: 'LLVM Weekly - #5, Feb 3rd 2014'
url: https://blog.llvm.org/2014/02/llvm-weekly-5-feb-3rd-2014.html
published: "2014-02-03T07:36:00Z"
feed: llvm
guid: https://blog.llvm.org/2014/02/llvm-weekly-5-feb-3rd-2014.html
---

# LLVM Weekly - #5, Feb 3rd 2014

Welcome to the fifth issue of LLVM Weekly, a weekly newsletter (published every Monday) covering developments in LLVM, Clang, and related projects. LLVM Weekly is brought to you by [Alex Bradbury](http://asbradbury.org). Subscribe to future issues at [http://llvmweekly.org](http://llvmweekly.org) and pass it on to anyone else you think may be interested. Please send any tips or feedback to [asb@asbradbury.org](mailto:asb@asbradbury.org), or [@llvmweekly](https://twitter.com/llvmweekly) or [@asbradbury](https://twitter.com/asbradbury) on Twitter. I've been keeping the [@llvmweekly Twitter account](https://twitter.com/llvmweekly) updated throughout the week, so follow that if you want more frequent news updates.

I'm afraid my summary of mailing list activities is much less thorough than usual, as I've been rather busy this weekend both moving house and suffering from a cold. Do ping me if you think I've missed anything important.

The canonical home for this issue [can be found here at llvmweekly.org](http://llvmweekly.org/issue/5).

### News and articles from around the web

This weekend there was an LLVM devroom at FOSDEM 2014. Slides have [already been posted](http://llvm.org/devmtg/2014-02/) for some of the talks. Hopefully videos will follow.

Pocl (Portable Computing Language) 0.9 has been [released](http://portablecl.org/pocl-0.9.html). Pocl aims to be an efficient MIT-licensed implementation of the OpenCL 1.2 standard.

Mike Ash has published a useful [introduction to libclang](https://mikeash.com/pyblog/friday-qa-2014-01-24-introduction-to-libclang.html).

Ever wanted to use LLVM from within Rust? This [blog post will tell you how](http://hydrocodedesign.com/2014/01/31/llvm-with-rust/).

Phoronix has published a [benchmark of Clang 3.4 vs GCC 4.9.0 20140126 on AMD Kaveri](http://www.phoronix.com/scan.php?page=article&item=amd_kaveri_gcc49clang34&num=1).

### On the mailing lists

- A question about the status of [SEH support in LLVM/Clang](http://article.gmane.org/gmane.comp.compilers.clang.devel/34747) was quickly derailed after the expiration of a key patent on SEH in June was brought up. The conversation moved to LLVM's [developer policy on patents](http://llvm.org/docs/DeveloperPolicy.html#patents) and whether the LLVM or Clang mailing lists are a suitable place for their discussion. In the end, Chris Lattner [chimes in to clarify the policy](http://article.gmane.org/gmane.comp.compilers.clang.devel/34810). The technical discussion [moved to this thread](http://article.gmane.org/gmane.comp.compilers.llvm.devel/69945).

- Pekka Jääskeläinen answers a bugpoint question with a [useful guide on using bugpoint with a custom exec-command option](http://article.gmane.org/gmane.comp.compilers.llvm.devel/69807).

- Nick Lewycky suggests [making datalayout a mandatory part of an LLVM module](http://article.gmane.org/gmane.comp.compilers.llvm.devel/69890).

- Chandler Carruth has proposed in an RFC that [BlockFrequency is the wrong metric](http://article.gmane.org/gmane.comp.compilers.llvm.devel/70027) for profile info.

- Sara Elshobaky asks for advice on finding the [number of instructions executed when running LLVM bytecode under lli](http://article.gmane.org/gmane.comp.compilers.llvm.devel/69797). There are a number of suggestions including callgrind, Pin, and other similar tools.

- Markus Timpl raises an interesting question about [describing a load instruction that changes the value of two registers](http://article.gmane.org/gmane.comp.compilers.llvm.devel/69852).

- Baoshan Pang writes to the LLVM list asking [how to start getting involved in LLVM](http://article.gmane.org/gmane.comp.compilers.llvm.devel/69973). If you've emailed me suggesting you'd like more pointers on how to get stuck in to LLVM, this thread is for you.


### LLVM commits

- The ARM exception handling ABI ( [EHABI](http://infocenter.arm.com/help/topic/com.arm.doc.ihi0038a/IHI0038A_ehabi.pdf)) is now enabled by default. [r200388](http://reviews.llvm.org/rL200388).

- TargetLowering gained a hook which targets can implement to indicate whether a load of a constant should be converted to just the constant. [r200271](http://reviews.llvm.org/rL200271).

- Line table debug info is now supported for COFF files when targeting win32. [r200340](http://reviews.llvm.org/rL200340).

- LLVM now has the beginnings of a line editor library, initially to be used by clang-query but possibly consumed by LLDB as well in the future. [r200595](http://reviews.llvm.org/rL200595).

- The R600 backend learned intrinsics for `S_SENDMSG` and `BUFFER_LOAD_DWORD*` instructions. [r200195](http://reviews.llvm.org/rL200195), [r200196](http://reviews.llvm.org/rL200196).

- The loop vectorizer gained a number of flags to help experiment with changing thresholds. It now also only unrolls by powers of 2. [r200212](http://reviews.llvm.org/rL200212), [r200213](http://reviews.llvm.org/rL200213).

- The loop vectorizer now supports conditional stores by scalarizing (they are put behind an if). This improves performance on the SPEC libquantum benchmark by 4.15%. [r200270](http://reviews.llvm.org/rL200270).

- MCSubtargetInfo is now explicitly passed to the `EmitInstruction`, `EmitInstTo*`, `EncodeInstruction` and other functions in the MC module. [r200345](http://reviews.llvm.org/rL200345) and others.

- llvm-readobj learned to decode ARM attributes. [r200450](http://reviews.llvm.org/rL200450).

- Speculative execution of llvm.{sqrt,fma,fmuladd} is now allowed. [r200501](http://reviews.llvm.org/rL200501).


### Clang commits

- Position Independent Code (PIC) is now turned on by default for Android targets. [r200290](http://reviews.llvm.org/rL200290).

- The Parser::completeExpression function was introduced, which returns a list of completions for a given expression and completion position. [r200497](http://reviews.llvm.org/rL200497).

- The default CPU for 32-bit and 64-bit MIPS targets is now mips32r2 and mips64r2 respectively. [r200222](http://reviews.llvm.org/rL200222).

- The ARM and AArch64 backends saw some refactoring to share NEON intrinsics. [r200524](http://reviews.llvm.org/rL200524) and others.


### Other project commits

- Compiler-rt gained a cache invalidation implementation for AArch64 [r200317](http://reviews.llvm.org/rL200317).

- Compiler-rt now features an optimised implementation of `__clzdi2` and `__clzsi2` for ARM. [r200394](http://reviews.llvm.org/rL200394).

- Compiler-rt's CMake files will now compile the library for ARM. Give it a go and see what breaks. [r200546](http://reviews.llvm.org/rL200546).

- The iohandler LLDB branch was merged in. The commit log describes the benefits. [r200263](http://reviews.llvm.org/rL200263).
