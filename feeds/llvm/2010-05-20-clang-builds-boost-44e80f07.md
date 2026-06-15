---
title: Clang++ Builds Boost!
url: https://blog.llvm.org/2010/05/clang-builds-boost.html
published: "2010-05-20T13:42:00Z"
feed: llvm
guid: https://blog.llvm.org/2010/05/clang-builds-boost.html
---

# Clang++ Builds Boost!

[Boost](http://www.boost.org/) is a collection of open-source, peer-reviewed C++ libraries that's well-known for having many good utility components for C++ programmers. It's also well-known for using bleeding-edge C++ techniques, such as extensive template and preprocessor metaprogramming, that have pushed many C++ compilers beyond their breaking point. Beyond just being a library, Boost has become a benchmark and a selling point for C++ compilers: is your compiler standards-confomant enough to build Boost?

Clang is.

This morning, Clang++ had its first fully-successful Boost regression test run, passing every applicable C++ test on the Boost release branch \[\*\]. According to today's [results](http://www.boost.org/development/tests/release/developer/summary.html), Clang is successfully compiling more of Boost than other, established compilers for which Boost has historically been tailored (through various workarounds and configuration switches). In fact, Clang's [compiler configuration](http://svn.boost.org/svn/boost/branches/release/boost/config/compiler/clang.hpp) in Boost is completely free of any of Boost's C++98/03 defect macros.

As for the specifics: tests were run on Mac OS X 10.6 (Snow Leopard) with Clang targeting x86-64, debug build of Boost with the Clang toolset and a Release build of Clang from Subversion trunk.

If you want to try Boost with Clang, first [get Clang from Subversion](http://clang.llvm.org/get_started.html) trunk and then [get Boost from Subversion](https://svn.boost.org/trac/boost/wiki/BoostSubversion) (the release branch is most stable) and edit your ~/user-config.jam file by adding the following line:

using clang ;

assuming that "clang++" is in your path. Then, you can run

bjam toolset=clang

to instruct Boost.Build to use Clang as its compiler.

\[\*\] For those keeping score, the "build" results are build-system tests, not compiler tests, and the MPI tests are disabled due to [breakage](https://svn.boost.org/trac/boost/ticket/4214) in the Serialization library. We are working with the library author to address the Serialization problems in Boost.
