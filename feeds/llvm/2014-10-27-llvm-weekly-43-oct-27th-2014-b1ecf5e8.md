---
title: 'LLVM Weekly - #43, Oct 27th 2014'
url: https://blog.llvm.org/2014/10/llvm-weekly-43-oct-27th-2014.html
published: "2014-10-27T04:02:00Z"
feed: llvm
guid: https://blog.llvm.org/2014/10/llvm-weekly-43-oct-27th-2014.html
---

# LLVM Weekly - #43, Oct 27th 2014

Welcome to the forty-third issue of LLVM Weekly, a weekly newsletter (published every Monday) covering developments in LLVM, Clang, and related projects. LLVM Weekly is brought to you by [Alex Bradbury](http://asbradbury.org). Subscribe to future issues at [http://llvmweekly.org](http://llvmweekly.org) and pass it on to anyone else you think may be interested. Please send any tips or feedback to [asb@asbradbury.org](mailto:asb@asbradbury.org), or [@llvmweekly](https://twitter.com/llvmweekly) or [@asbradbury](https://twitter.com/asbradbury) on Twitter.

This week it's the LLVM Developers' Meeting in San Jose. Check out the [schedule](http://llvm.org/devmtg/2014-10/). Unfortunately I won't be there, so I'm looking forward to the slides and videos going online.

The canonical home for this issue [can be found here at llvmweekly.org](http://llvmweekly.org/issue/43).

### News and articles from around the web

Philip Reames has written up a detailed [discussion of statepoints vs gcroot](http://www.philipreames.com/Blog/2014/10/21/statepoints-vs-gcroot-for-representing-call-safepoints/) for representing call safepoints. The aim is to clearly explain how the safepoint functionality provided by the [patches currently up for review](http://reviews.llvm.org/D5683) differ to the current gc.root support.

The Haskell community have put together a [proposal for an improved LLVM backend to GHC](https://ghc.haskell.org/trac/ghc/wiki/ImprovedLLVMBackend). They intend to ship GHC with its own local LLVM build.

CoderGears have published a blog post about [using Clang to get better warnings in Visual C++ projects](http://www.codergears.com/Blog/?p=246).

There is going to be a dedicated LLVM devroom at FOSDEM 2015. Here is the [call for speakers and participation](http://article.gmane.org/gmane.comp.compilers.clang.devel/39450).

### On the mailing lists

- Elena Demikhovsky has asked for comments on a [proposal to add masked vector load and store intrinsics](http://article.gmane.org/gmane.comp.compilers.llvm.devel/78160). Essentially all feedback so far is positive on the idea.

- Renato Golin [proposes moving libunwind into compiler-rt](http://article.gmane.org/gmane.comp.compilers.llvm.devel/78099). One of the subtleties is hat libunwind isn't fully compatible with GCC's unwind implementation (due to different data structure layouts), which means they can't be mixed.

- Kristof Beyls has posted some [notes in preparation for the benchmarking infrastructure BoF at the LLVM dev meeting](http://article.gmane.org/gmane.comp.compilers.llvm.devel/78068).


### LLVM commits

- The `nonnull` metadata has been introduced for Load instructions. [r220240](http://reviews.llvm.org/rL220240).

- minnum and maxnum intrinsics have been added. [r220341](http://reviews.llvm.org/rL220341), [r220342](http://reviews.llvm.org/rL220342).

- The Hexagon backend gained a basic disassembler. [r220393](http://reviews.llvm.org/rL220393).

- PassConfig gained usingDefaultRegAlloc to tell if the default register allocator is being used. [r220321](http://reviews.llvm.org/rL220321).

- An llvm-go tool has been added. It is intended to be used to build components such as the Go frontend in-tree. [r220462](http://reviews.llvm.org/rL220462).


### Clang commits

- C compilation defaults to C11 by default, matching the behaviour of GCC 5.0. [r220244](http://reviews.llvm.org/rL220244).

- Clang should now be better at finding Visual Studio in non-standard setups. [r220226](http://reviews.llvm.org/rL220226).

- The Windows toolchain is now known as MSVCToolChain, to allow the addition a CrossWindowsToolChain which will use clang/libc++/lld. [r220362](http://reviews.llvm.org/rL220362), [r220546](http://reviews.llvm.org/rL220546).


### Other project commits

- The libcxxabi gained support for running libc++abi tests with sanitizers. [r220464](http://reviews.llvm.org/rL220464).
