---
title: Simple guided fuzzing for libraries using LLVM's new libFuzzer
url: https://blog.llvm.org/2015/04/fuzz-all-clangs.html
published: "2015-04-09T13:40:00Z"
feed: llvm
guid: https://blog.llvm.org/2015/04/fuzz-all-clangs.html
---

# Simple guided fuzzing for libraries using LLVM's new libFuzzer

Fuzzing (or [fuzz testing](http://en.wikipedia.org/wiki/Fuzz_testing)) is becoming increasingly popular. Fuzzing Clang and fuzzing with Clang is not new: Clang-based [AddressSanitizer](http://clang.llvm.org/docs/AddressSanitizer.html) has been used for fuzz-testing the Chrome browser for [several years](http://blog.chromium.org/2012/04/fuzzing-for-security.html) and Clang itself has been extensively fuzzed using [csmith](https://embed.cs.utah.edu/csmith/)and, more recently, using [AFL](http://lcamtuf.coredump.cx/afl/). Now we’ve closed the loop and started to fuzz parts of LLVM (including Clang) using LLVM itself.

[LibFuzzer](http://llvm.org/docs/LibFuzzer.html), recently added to the LLVM tree, is a library for in-process fuzzing that uses [Sanitizer Coverage instrumentation](https://code.google.com/p/address-sanitizer/wiki/AsanCoverage) to guide test generation. With LibFuzzer one can implement a guided fuzzer for some library by writing one simple function:

extern "C" void TestOneInput(const uint8\_t \*Data, size\_t Size);

We have implemented two fuzzers on top of LibFuzzer: [clang-format-fuzzer](http://llvm.org/viewvc/llvm-project/cfe/trunk/tools/clang-format/fuzzer/ClangFormatFuzzer.cpp?view=markup) and [clang-fuzzer](http://llvm.org/viewvc/llvm-project/cfe/trunk/tools/clang-fuzzer/ClangFuzzer.cpp?view=markup). Clang-format is mostly a lexical analyzer, so giving it random bytes to format worked perfectly and discovered [over 20 bugs](https://llvm.org/bugs/show_bug.cgi?id=23052). Clang however is more than just a lexer and giving it random bytes barely scratches the surface, so in addition to testing with random bytes we also fuzzed Clang in [token-aware mode](http://llvm.org/docs/LibFuzzer.html#tokens). Both modes found [bugs](https://llvm.org/bugs/show_bug.cgi?id=23057); some of them were previously [detected](http://lists.cs.uiuc.edu/pipermail/cfe-dev/2015-January/040705.html)[by AFL](http://lists.cs.uiuc.edu/pipermail/cfe-dev/2015-January/040705.html), some others were not: we’ve run this fuzzer with AddressSanitizer and some of the bugs are not easily discoverable without it.

Just to give you the feeling, here are some of the input samples discovered by the token-aware clang-fuzzer starting from an empty test corpus:

static void g(){}

signed\*Qwchar\_t;

overridedouble++!=~;inline-=}y=^bitand{;\*=or;goto\*&&k}==n

int XS/=~char16\_t&s<=const\_cast<Xchar\*>(thread\_local3+=char32\_t

Fuzzing is not a one-off thing -- it shines when used continuously. We have set up a [public build bot](http://lab.llvm.org:8011/builders/sanitizer-x86_64-linux-fuzzer) that runs clang-fuzzer and clang-format-fuzzer 24/7. This way, the fuzzers keep improving the test corpus and will periodically find old bugs or fresh regressions (the bot has [caught](http://reviews.llvm.org/D7871) at least one such regression already).

The benefit of in-process fuzzing compared to out-of-process is that you can test more inputs per second. This is also a weakness: you can not effectively limit the execution time for every input. If some of the inputs trigger superlinear behavior, it may slow down or paralyze the fuzzing. Our fuzzing bot was nearly dead after it discovered [exponential parsing time in clang-format](https://llvm.org/bugs/show_bug.cgi?id=23052#c3). You can workaround the problem by setting a timeout for the fuzzer, but it’s always better to fix superlinear behavior.

It would be interesting to fuzz other parts of LLVM, but a requirement for in-process fuzzing is that the library does not crash on invalid inputs. This holds for clang and clang-format, but not for, e.g., the LLVM bitcode reader.

Help is more than welcome! You can start by fixing one of the existing bugs in clang or clang-format (see [PR23057](https://llvm.org/bugs/show_bug.cgi?id=23057), [PR23052](https://llvm.org/bugs/show_bug.cgi?id=23052) and the [results from AFL](http://sli.dy.fi/~sliedes/clang-triage/triage_report.xhtml)) or write your own fuzzer for some other part of LLVM or profile one of the existing fuzzers and try to make it faster by fixing performance bugs.

Of course, LibFuzzer can be used to test things outside of the LLVM project. As an example, and following [Hanno Böck’s blog post](https://blog.hboeck.de/archives/868-How-Heartbleed-couldve-been-found.html) on [Heartbleed](http://en.wikipedia.org/wiki/Heartbleed), we’ve applied LibFuzzer to [openssl](https://www.openssl.org/)and found Heartbleed in [less than a minute](http://llvm.org/docs/LibFuzzer.html#heartbleed). Also, quite a few new bugs have been discovered in [PCRE2](http://www.pcre.org/) ( [1](http://www.exim.org/viewvc/pcre2/code/trunk/ChangeLog?r1=228&r2=227&pathrev=228), [2](http://www.exim.org/viewvc/pcre2/code/trunk/ChangeLog?r1=231&r2=230&pathrev=231), [3](http://www.exim.org/viewvc/pcre2?view=revision&revision=232), [4](http://www.exim.org/viewvc/pcre2/code/trunk/ChangeLog?r1=233&r2=232&pathrev=233), [5](http://www.exim.org/viewvc/pcre2/code/trunk/ChangeLog?r1=234&r2=233&pathrev=234), [6](http://www.exim.org/viewvc/pcre2/code/trunk/ChangeLog?r1=235&r2=234&pathrev=235), [7](http://www.exim.org/viewvc/pcre2/code/trunk/ChangeLog?r1=236&r2=235&pathrev=236), [8](http://www.exim.org/viewvc/pcre2/code/trunk/ChangeLog?r1=237&r2=236&pathrev=237), [9](http://www.exim.org/viewvc/pcre2/code/trunk/ChangeLog?r1=238&r2=237&pathrev=238), [10](http://www.exim.org/viewvc/pcre2/code/trunk/ChangeLog?r1=241&r2=240&pathrev=241), [11](http://www.exim.org/viewvc/pcre2/code/trunk/ChangeLog?r1=244&r2=243&pathrev=244)), [Glibc](https://sourceware.org/glibc/wiki/FuzzingLibc)and [MUSL libc](http://musl-libc.org/) ( [1](http://git.musl-libc.org/cgit/musl/patch/?id=39dfd58417ef642307d90306e1c7e50aaec5a35c), [2](http://git.musl-libc.org/cgit/musl/patch/?id=fc13acc3dcb5b1f215c007f583a63551f6a71363)) .

Fuzz testing, especially coverage-directed and sanitizer-aided fuzz testing, should directly compliment unit testing, integration testing, and system functional testing. We encourage everyone to start actively fuzz testing their interfaces, especially those with even a small chance of being subject to attacker-controlled inputs. We hope the LLVM fuzzing library helps you start leveraging our tools to better test your code, and let us know about any truly exciting bugs they help you find!
