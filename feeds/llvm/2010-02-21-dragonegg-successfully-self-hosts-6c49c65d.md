---
title: Dragonegg Successfully Self-Hosts!
url: https://blog.llvm.org/2010/02/dragonegg-successfully-self-hosts.html
published: "2010-02-21T05:07:00Z"
feed: llvm
guid: https://blog.llvm.org/2010/02/dragonegg-successfully-self-hosts.html
---

# Dragonegg Successfully Self-Hosts!

The [dragonegg GCC plugin](http://dragonegg.llvm.org/) can host itself! Dragonegg lets you use the [LLVM](http://llvm.org/) optimizers with [GCC-4.5](http://gcc.gnu.org/), much like [llvm-gcc](http://llvm.org/cmds/llvmgcc.html), but unlike llvm-gcc does not involve modifying GCC, thanks to the new GCC plugin infrastructure (currently one small patch is required). We built all of GCC-4.5, LLVM and dragonegg with dragonegg, then used the resulting binaries to build them all again. Why? Because we love to build! And because this was a great way of checking that nothing was miscompiled. The final dragonegg plugin was fully functional, successfully passing the entire dragonegg test suite.
