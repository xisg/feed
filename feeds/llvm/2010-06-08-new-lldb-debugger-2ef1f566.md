---
title: New "lldb" Debugger
url: https://blog.llvm.org/2010/06/new-lldb-debugger.html
published: "2010-06-08T21:00:00Z"
feed: llvm
guid: https://blog.llvm.org/2010/06/new-lldb-debugger.html
---

# New "lldb" Debugger

I'm happy to announce a great new subproject of LLVM: LLDB. LLDB is a modern debugger infrastructure which is built (like the rest of LLVM) as a series of modular and reusable libraries. LLDB builds on existing LLVM technologies like the [enhanced disassembler](http://blog.llvm.org/2010/01/x86-disassembler.html) APIs, the Clang ASTs and expression parser, the LLVM code generator and JIT compiler.

While still in early development, LLDB supports basic command line debugging scenarios on the Mac, is scriptable, and has great support for multithreaded debugging. LLDB is already much faster than GDB when debugging large programs, and has the promise to provide a much better user experience (particularly for C++ programmers). We are excited to see the new platforms, new features, and enhancements that the broader LLVM community is interested in.

If you'd like to try out LLDB and participate in its development, please visit [http://lldb.llvm.org/](http://lldb.llvm.org/) and consider signing up for the [lldb-dev](http://lists.cs.uiuc.edu/mailman/listinfo/lldb-dev) and [lldb-commits](http://lists.cs.uiuc.edu/mailman/listinfo/lldb-commits) mailing lists.

-Chris and the LLDB Team
