---
title: llvm-reduce
url: https://blog.regehr.org/archives/2109
published: "2021-05-13T16:58:00Z"
feed: regehr
guid: https://blog.regehr.org/?p=2109
---

# llvm-reduce

Test-case reduction is more or less a necessity when debugging failures of complex programs such as compilers. Automated test-case reduction is useful not only because it allows developers to avoid wasting time reducing inputs by hand, but also because it supports new techniques such as automatically triaging bulk failures seen in the field or during fuzzing campaigns.

llvm-reduce is a newish tool for reducing LLVM IR. Structurally, it is similar to C-Reduce, using a reducer core that calls out to a modular collection of reduction passes. Each pass implements a potentially-reducing transformation on LLVM IR, such as removing some instructions, in order to create variants: candidates for further reduction. The core inspects each variant, keeping only the ones that are “interesting” according to some user-defined criterion. A module full of LLVM IR might, for example, be interesting if it triggers some assertion violation in a backend.

In test-case reduction there’s a tension between creating generic tools that are intended to reduce all kinds of input, and creating specific tools that contain transformations that only work on one kind of input. llvm-reduce is all the way at one end of this spectrum: it can’t be used to reduce anything other than LLVM IR. This kind of specificity has advantages, such as reducing more rapidly by not creating illegal variants and reducing more effectively by supporting invariant-preserving transformations that are infeasible to perform for tools that don’t understand the relevant invariants. For example, when removing a parameter from a function, it is necessary to also remove the corresponding argument from each location where that function is called.

Something that I like quite a bit about llvm-reduce is that its interestingness tests are compatible with C-Reduce’s, so people can switch between the two tools to see if one of them works better than the other for some particular problem.

llvm-reduce is part of LLVM, so you will get it for free by building or downloading an LLVM. The implementation is super approachable if you’re interested in digging into it. You can learn more about this tool from [this talk by its creator](https://www.youtube.com/watch?v=n1jDj7J9N8c), and from [this page](https://llvm.org/docs/HowToSubmitABug.html).
