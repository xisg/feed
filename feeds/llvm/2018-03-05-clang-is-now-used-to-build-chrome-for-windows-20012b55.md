---
title: Clang is now used to build Chrome for Windows
url: https://blog.llvm.org/2018/03/clang-is-now-used-to-build-chrome-for.html
published: "2018-03-05T12:46:00Z"
feed: llvm
guid: https://blog.llvm.org/2018/03/clang-is-now-used-to-build-chrome-for.html
---

# Clang is now used to build Chrome for Windows

As of Chrome 64, Chrome for Windows is compiled with Clang. We now use Clang to build Chrome for all platforms it runs on: macOS, iOS, Linux, Chrome OS, Android, and Windows. Windows is the platform with the second most Chrome users after Android [according to statcounter](http://gs.statcounter.com/browser-version-market-share), which made this switch particularly exciting.

Clang is the first-ever open-source C++ compiler that’s ABI-compatible with Microsoft Visual C++ (MSVC) – meaning you can build some parts of your program (for example, system libraries) with the MSVC compiler (“cl.exe”), other parts with Clang, and when linked together (either by MSVC’s linker, “link.exe”, or LLD, the LLVM project’s linker – see below) the parts will form a working program.

Note that Clang is not a replacement for Visual Studio, but an addition to it. We still use Microsoft’s headers and libraries to build Chrome, we still use some SDK binaries like midl.exe and mc.exe, and many Chrome/Win developers still use the Visual Studio IDE (for both development and for debugging).

This post discusses numbers, motivation, benefits and drawbacks of using Clang instead of MSVC, how to try out Clang for Windows yourself, project history, and next steps. For more information on the technical side you can look at the [slides of our 2015 LLVM conference talk](https://docs.google.com/presentation/d/1oxNHaVjA9Gn_rTzX6HIpJHP7nXRua_0URXxxJ3oYRq0/edit#slide=id.p), and the slides linked from there.

### Numbers

This is what most people ask about first, so let’s talk about it first. We think the other sections are more interesting though.

#### Build time

Building Chrome locally with Clang is about 15% slower than with MSVC. (We’ve heard that Windows Defender can make Clang builds a lot slower on some machines, so if you’re seeing larger slowdowns, make sure to whitelist Clang in Windows Defender.) However, the way Clang emits debug info is more parallelizable and builds with a distributed build service (e.g. [Goma](https://chromium.googlesource.com/infra/goma/client/)) are hence faster.

#### Binary size

Chrome installer size gets smaller for 64-bit builds and slightly larger for 32-bit builds using Clang. The same difference shows in uncompressed code size for regular builds as well (see the [tracking bug for Clang binary size](https://crbug.com/457078) for many numbers). However, compared to MSVC builds using link-time code generation (LTCG) and [profile-guided optimization](https://blog.chromium.org/2016/10/making-chrome-on-windows-faster-with-pgo.html) (PGO) Clang generates larger code in 64-bit for targets that use /O2 but smaller code for targets that use /Os. The installer size comparison suggests Clang's output compresses better.

Some raw numbers for versions 64.0.3278.2 (MSVC PGO) and 64.0.3278.0 (Clang). mini\_installer.exe is Chrome’s installer that users download, containing the LZMA-compressed code. chrome\_child.dll is one of the two main dlls; it contains Blink and V8, and generally has many targets that are built with /O2. chrome.dll is the other main dll, containing the browser process code, mostly built with /Os.

mini\_installer.exe

chrome.dll

chrome\_child.dll

chrome.exe

32-bit win-pgo

45.46 MB

36.47 MB

53.76 MB

1.38 MB

32-bit win-clang

45.65 MB

(+0.04%)

42.56 MB (+16.7%)

62.38 MB

(+16%)

1.45 MB

(+5.1%)

64-bit win-pgo

49.4 MB

53.3 MB

65.6 MB

1.6 MB

64-bit win-clang

46.27 MB

(-6.33%)

50.6 MB

(-5.1%)

72.71 MB

(+10.8%)

1.57 MB

(-1.2%)

#### Performance

We conducted extensive A/B testing of performance. Performance telemetry numbers are about the same for MSVC-built and clang-built Chrome – some metrics get better, some get worse, but all of them are within 5% of each other. The official MSVC builds used LTCG and PGO, while the Clang builds currently use neither of these. This is potential for improvement that we look forward to exploring. The PGO builds took a very long time to build due to the need for collecting profiles and then building again, and as a result, the configuration was not enabled on our performance-measurement buildbots. Now that we use Clang, the perf bots again track the configuration that we ship.

Startup performance was worse in Clang-built Chrome until we started using a [link-order file](https://chromium.googlesource.com/chromium/src/+/master/docs/win_order_files.md) – a form of “PGO light” .

#### Stability

We A/B-tested stability as well and found no difference between the two build configurations.

### Motivation

There were many motivating reasons for this project, the overarching theme being the benefits of using the same compiler across all of Chrome’s platforms, as well as the ability to change the compiler and deploy those changes to all our developers and buildbots quickly. Here’s a non-exhaustive list of examples.

- Chrome is heavily using technology that’s based on compiler instrumentation (ASan, CFI, [ClusterFuzz](https://blog.chromium.org/2012/04/fuzzing-for-security.html)—uses ASan). Clang supports this instrumentation already, but we can’t add it to MSVC. We previously used [after-the-fact binary instrumentation](https://github.com/google/syzygy) to mitigate this a bit, but having the toolchain write the right bits in the first place is cleaner and faster.
- Clang enables us to write compiler [plugins](https://www.chromium.org/developers/coding-style/chromium-style-checker-errors) that add Chromium-specific warnings and to write tooling for [large-scale refactoring](https://chromium.googlesource.com/chromium/src/+/lkcr/docs/clang_tool_refactoring.md). [Chromium’s code search](https://cs.chromium.org/) can now learn to index Windows code.
- Chromium is open-source, so it’s nice if it’s built with an open-source toolchain.
- Chrome runs on 6+ platforms, and most developers are only familiar with 1-3 platforms. If your patch doesn’t compile on a platform you’re unfamiliar with, due to a compiler error that you can’t locally reproduce on your local development machine, it’ll take you a while to fix. On the other hand, if all platforms use the same compiler, if it builds on your machine then it’s probably going to build on all platforms.
- Using the same compiler also means that compiler-specific micro-optimizations will help on all platforms (assuming that the same -O flags are used on all platforms – not yet the case in Chrome, and only on the same ISAs – x86 vs ARM will stay different).
- Using the same compiler enables [cross-compiling](https://cs.chromium.org/chromium/src/docs/win_cross.md) – developers who feel most at home on a Linux box can now work on Windows-specific code, from their Linux box (without needing to run Wine).
- We can [continuously build Chrome trunk with Clang trunk](https://ci.chromium.org/p/chromium/g/chromium.clang/console) to find compiler regressions quickly. This allows us to update Clang every week or two. Landing a major MSVC update in Chrome usually took a year or more, with several rounds of reporting internal compiler bugs and miscompiles. The issue here isn’t that MSVC is more buggy than Clang – it isn’t, all software is buggy – but that we can continuously improve Clang due to Clang being open-source.
- C++ receives major new revisions every few years. When C++11 was released, we were still using six different compilers, and [enabling C++11](http://chromium-cpp.appspot.com/) was difficult. With fewer compilers, this gets much easier.
- We can prioritize compiler features that are important to us. For example:

  - [Deterministic builds](https://www.chromium.org/developers/testing/isolated-testing/deterministic-builds) were important to us before they were important for the MSVC team. For example, link.exe /incremental depends on an incrementing mtime timestamp in each object file.
  - We could enable warnings that fired in system headers long before [MSVC added support for the system header concept](https://blogs.msdn.microsoft.com/vcblog/2017/12/13/broken-warnings-theory/).
  - cl.exe always prints the name of the input file, so that the [build system has to filter it out for quiet builds](https://github.com/ninja-build/ninja/blob/master/src/clparser.cc#L67).

Of course, not all – or even most – of these reasons will apply to other projects.

### Benefits and drawbacks of using Clang instead of Visual C++

Benefits of using Clang, if you want to try for your project:

- Clang supports 64-bit inline assembly. For example, in Chrome we built libyuv (a video format conversion library) with Clang long before we built all of Chrome with it. libyuv had highly-tuned 64-bit inline assembly with performance not reachable with intrinsics, and we could just use that code on Windows.
- If your project runs on multiple platforms, you can use one compiler everywhere. Building your project with several compilers is generally considered good for code health, but in Chrome we found that Clang’s diagnostics found most problems and we were mostly battling compiler bugs (and if another compiler has a great new diagnostic, we can add that to Clang).
- Likewise, if your project is Windows-only, you can get a second compiler’s opinion on your code, and Clang’s warnings might find bugs.
- You can use [Address Sanitizer](https://clang.llvm.org/docs/AddressSanitizer.html) to find memory bugs.
- If you don’t use LTCG and PGO, it’s possible that Clang might create faster code.
- Clang’s [diagnostics and fix-it hints](https://clang.llvm.org/diagnostics.html).

There are also drawbacks:

- Clang doesn’t support C++/CX or #import “foo.dll”.
- MSVC offers paid support, Clang only gives you the code and the ability to write patches yourself (although the community is very active and helpful!).
- MSVC has better documentation.
- Advanced debugging features such as Edit & Continue don’t work when using Clang.

### How to use

If you want to give Clang for Windows a try, there are two approaches:

1. You could use clang-cl, a compiler driver that tries to be command-line flag compatible with cl.exe (just like Clang tries to be command-line flag compatible with gcc). The [Clang user manual](https://clang.llvm.org/docs/UsersManual.html#clang-cl) describes how you can tell popular Windows build systems how to call clang-cl instead of cl.exe. We used this approach in Chrome to keep the Clang/Win build working alongside the MSVC build for years, with minimal maintenance cost. You can keep using link.exe, all your current compile flags, the MSVC debugger or windbg, ETW, etc. clang-cl even writes warning messages in a format that’s compatible with cl.exe so that you can click on build error messages in Visual Studio to jump to the right file and line. Everything should just work.
2. Alternatively, if you have a cross-platform project and want to use gcc-style flags for your Windows build, you can pass a Windows triple (e.g. --target=x86\_64-windows-msvc) to regular Clang, and it will produce MSVC-ABI-compatible output. Starting in Clang 7.0.0, due Fall 2018, Clang will also default to CodeView debug info with this triple.

Since Clang’s output is ABI-compatible with MSVC, you can build parts of your project with clang and other parts with MSVC. You can also pass [/fallback](https://clang.llvm.org/docs/UsersManual.html#the-fallback-option) to clang-cl to make it call cl.exe on files it can’t yet compile (this should be rare; it never happens in the Chrome build).

clang-cl accepts Microsoft language extensions needed to parse system headers but tries to emit -Wmicrosoft-foo warnings when it does so (warnings are ignored for system headers). You can choose to fix your code, or pass -Wno-microsoft-foo to Clang.

link.exe can produce regular PDB files from the CodeView information that Clang writes.

### Project History

We switched chrome/mac and [chrome/linux](http://blog.llvm.org/2015/01/using-clang-for-chrome-production.html) to Clang a while ago. But on Windows, Clang was still missing support for parsing many Microsoft language extensions, and it didn’t have any Microsoft C++ ABI-compatible codegen at all. In 2013, we [spun up a team](http://blog.llvm.org/2013/09/a-path-forward-for-llvm-toolchain-on.html) to improve Clang’s Windows support, consisting half of Chrome engineers with a compiler background and half of other toolchain people. In mid-2014, Clang could [self-host on Windows](http://blog.llvm.org/2014/07/clangllvm-on-windows-update.html). In February 2015, we had the first fallback-free build of 64-bit Chrome, in July 2015 the first fallback-free build of 32-bit Chrome (32-bit SEH was difficult). In Oct 2015, we shipped a first clang-built Chrome to the Canary channel. Since then, we’ve worked on [improving the size of Clang’s output](http://https//crbug.com/457078), [improved Clang’s debug information](https://crbug.com/636111) (some of it behind -instcombine-lower-dbg-declare=0 for now), and A/B-tested stability and telemetry performance metrics.

We use versions of Clang that are pinned to a recent upstream revision that we update every one to three weeks, without any local patches. All our work is done in upstream LLVM.

Mid-2015, Microsoft announced that they were building on top of our work of making Clang able to parse all the Microsoft SDK headers with [clang/c2](https://blogs.msdn.microsoft.com/vcblog/2015/05/01/bringing-clang-to-windows/), which used the Clang frontend for parsing code, but cl.exe’s codegen to generate code. [Development on clang/c2 was halted again](https://twitter.com/stephantlavavej/status/871861920315211776?lang=en) in mid-2017; it is conceivable that this was related to our improvements to MSVC-ABI-compatible Clang codegen quality. We’re thankful to Microsoft for publishing documentation on the PDB file format, answering many of our questions, fixing Clang compatibility issues in their SDKs, and for giving us publicity on their blog! Again, Clang is not a replacement for MSVC, but a complement to it.

Opera for Windows is also [compiled with Clang](http://blogs.opera.com/desktop/2018/02/opera-51/) starting in version 51.

Firefox is also looking at [using clang-cl for building Firefox for Windows](https://ehsanakhgari.org/blog/2016-01-29/building-firefox-with-clang-cl-a-status-update).

### Next Steps

Just as clang-cl is a cl.exe-compatible interface for Clang, lld-link is a link.exe-compatible interface for lld, the LLVM linker. Our next step is to use lld-link as an alternative to link.exe for linking Chrome for Windows. This has many of the same advantages as clang-cl (open-source, easy to update, …). Also, using clang-cl together with lld-link allows using [LLVM-bitcode-based LTO](https://llvm.org/docs/LinkTimeOptimization.html) (which in turn enables using [CFI](https://clang.llvm.org/docs/ControlFlowIntegrity.html)) and [using PE/COFF extensions to speed up linking](http://blog.llvm.org/2018/01/improving-link-time-on-windows-with.html). A prerequisite for using lld-link was [its ability to write PDB files](http://blog.llvm.org/2017/08/llvm-on-windows-now-supports-pdb-debug.html).

We’re also considering using libc++ instead of the MSVC STL – this allows us to instrument the standard library, which is again useful for CFI and Address Sanitizer.

### In Closing

Thanks to the whole LLVM community for helping to create the first new production C++ compiler for Windows in over a decade, and the first-ever open-source C++ compiler that’s ABI-compatible with MSVC!
