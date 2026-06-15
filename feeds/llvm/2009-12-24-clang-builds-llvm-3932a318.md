---
title: Clang Builds LLVM
url: https://blog.llvm.org/2009/12/clang-builds-llvm.html
published: "2009-12-24T13:23:00Z"
feed: llvm
guid: https://blog.llvm.org/2009/12/clang-builds-llvm.html
---

# Clang Builds LLVM

Just in time for the Christmas holiday, the Clang project has hit a major milestone: Clang can now build all of LLVM and Clang!

The resulting Clang-built-Clang is not yet functional, so this "self-build" milestone is well short of full self-hosting. However, self-building indicates that C++ parsing, semantic analysis, and code generation is solid enough to compile the entirety of LLVM (~350k lines of C++ code) and Clang (~200k lines of C++ code) and produce object files that link together properly. To get to this point, we've fixed many bugs in Clang (when compiling C++ code), but also several bugs in LLVM and Clang that were found by Clang itself.

We are tracking [a number of Clang bugs](http://llvm.org/bugs/show_bug.cgi?id=5221) that manifest when building LLVM and Clang, as we make our way to the next big milestone: full self-hosting of Clang!
