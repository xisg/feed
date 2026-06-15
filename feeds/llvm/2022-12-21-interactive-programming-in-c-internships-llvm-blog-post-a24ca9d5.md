---
title: Interactive programming in C++ internships — LLVM Blog Post
url: https://blog.llvm.org/posts/2022-12-21-compiler-research-internships/
published: "2022-12-21T00:00:00Z"
feed: llvm
guid: https://blog.llvm.org/posts/2022-12-21-compiler-research-internships/
---

# Interactive programming in C++ internships — LLVM Blog Post

* * *

Another program year is ending and our [Compiler Research](https://compiler-research.org/) team is extremely happy to share the hard work and the results of our internscontributors!

The Compiler Research team includes researchers located at Princeton Universityand CERN.Our primary goal is research into foundational software tools helping scientiststo program for speed, interoperability, interactivity, flexibility, andreproducibility. We work in various fields of science such as high energyphysics, where research is fundamentally connected to software at exabyte scale.We develop computational methods and research software for scientificexploration and discovery. Our current research focuses on three main topics: [interpretative C/C++/CUDA](https://compiler-research.org/interactive_cpp), automatic differentiation tools, and C++ language interoperability with Pythonand D.

This year, we had six participants who worked on seven projects, covering a widerange of topics, from LLVM’s new JIT linker( [JITLink](https://llvm.org/docs/JITLink.html#jit-linking)) to [Clang-Repl](https://clang.llvm.org/docs/ClangRepl.html),an interactive C++ interpreter for incremental compilation integratedin [Clang](https://clang.llvm.org/). All projects rise toward acommon, ambitious goal: **to establish a proficient workflow in LLVM,where [interactivedevelopment in C++](https://compiler-research.org/interactive_cpp) is possible, and exploratory C++ becomes an accessibleexperience to a wider audience.**

Below you can find the list of projects from our interns, an overview of theobjectives and results. We invite you to follow the links to access a moredetailed description of each project.

## [JITLink support for a new format/architecture (ELF/AARCH64)](https://compiler-research.org/assets/docs/Sunho_Kim_GSoC22_Report.pdf)

Developer: **[Sunho Kim](https://github.com/sunho)**( _Computer Science, De Anza College, Cupertino, California_)

Mentors: **Stefan Gränitz** ( _Freelance Compiler Developer, Berlin,Deutschland_), **Lang Hames** ( _Apple_), **Vassil Vassilev** ( _Princeton University/CERN_)

Funding: **Google Summer of Code 2022**

Suhno developed a [JITLink](https://llvm.org/docs/JITLink.html#jit-linking) specialization that extends JITLink’s generic linker algorithm, allowingJITLink to support ELF/aarch64 target and provides full support of all advancedPE/COFF object file features. By supporting the ELF/aarch64 target, JITLink cannow be used in Julia, while COFF/X86\_64 enables Microsoft Visual C++ (MSCV)target in [Clang-Repl](https://clang.llvm.org/docs/ClangRepl.html).Here you can find Sunho’s GSoC [final report](https://compiler-research.org/assets/docs/Sunho_Kim_GSoC22_Report.pdf).

**Hurray!** This project has been accepted as a Tutorial at the [2022 LLVM Developers’ Meeting](https://llvm.swoogo.com/2022devmtg/speakers)!The conference took place in San Jose, California, from 7th to 10th November2022.Here you can find the [video](https://www.youtube.com/watch?v=UwHgCqQ2DDA&list=PL_R5A0lGi1ACZDCQw533fo2dBljmOqIYx&index=42) and the [slides](https://llvm.org/devmtg/2022-11/slides/Tutorial2-JITLink.pdf) of Sunho’s presentation.

The project was also presented at the Compiler As a Service (CaaS) monthlymeeting. Here you can find the [video](https://www.youtube.com/watch?v=_5_gm58sQIg) and the [slides](https://compiler-research.org/assets/presentations/S_Kim-Jitlink_Coff.pdf) of Sunho’s presentation.

## [Extend Clang to resugar template specialization accesses](https://compiler-research.org/blogs/gsoc22_izvekov_experience_blog/)

Developer: **[Matheus Izvekov](https://github.com/mizvekov)**

Mentors: **Richard Smith** ( _Google_), **Vassil Vassilev** ( _PrincetonUniversity/CERN_)

Funding: **Google Summer of Code 2022**

[Clang](https://clang.llvm.org/)’s type system was optimized bypushing type-syntactic-sugar on the arguments of a template specialization intothose member accesses. This was achieved by: 1. creating a new type node intothe Abstract Syntax Tree (AST) that represents the sugar of a member access ina template specialization; and 2. implementing single step desugaring logicwhich will perform the substitution of template parameter sugar into thecorresponding specialization argument sugar. As a result, we improved Clang’sdiagnostic system by enabling other constructs to preserve type sugar, allowingboth for their representation when present in the specialization argument.Here you can find Matheus’ GSoC [final report](https://compiler-research.org/assets/docs/Matheus_Izvekov_GSoC22_Report.pdf).

**Hurray!** This project has been accepted as a Lightning Talk at the [2022 LLVM Developers’ Meeting](https://llvm.swoogo.com/2022devmtg/speakers)!The conference took place in San Jose, California, from 7th to 10th November2022.Here you can find the [video](https://www.youtube.com/watch?v=bZ2HPrSkTZI) and the [slides](https://llvm.org/devmtg/2022-11/slides/Lightning9-TypeResugaringInClang%20.pdf) of Matheus’ presentation.

## [Recovering from Errors in Clang-Repl and Code Undo](https://compiler-research.org/blogs/gsoc22_zhang_chaudhari_experience_blog/)

Developers: **[Jun Zhang](https://github.com/junaire)**( _Software Engineering, Anhui Normal University, WuHu, China_) and **[Purva Chaudhari](https://github.com/Purva-Chaudhari)**( _California State University Northridge, Northridge CA, USA_)

Mentors: **Vassil Vassilev** ( _Princeton University/CERN_)

[Clang-Repl](https://clang.llvm.org/docs/ClangRepl.html) is aninteractive C++ interpreter integrated in Clang, which enables incrementalcompilation. It supports interactive programming for C++ in aRead-Eval\_Print Loop( [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop))style, compiling the code just-in-time with a JIT approach that reduces thecompile-run cycles. We added the “undo” functionality to Clang-REPL, and weimproved error-recovery by adding the possibility to recover the low-levelexecution infrastructure. As a result, Clang-REPL now supports reversalexecution and recovery actions on behalf of a user in an automatic andconvenient way.

**Hurray!** This project has been accepted as a Lightning Talk at the [2022 LLVM Developers’ Meeting](https://llvm.swoogo.com/2022devmtg/speakers)!The conference took place in San Jose, California, from 7th to 10th November2022.Here you can find the [video](https://www.youtube.com/watch?v=LSPBPC2av54) and the [slides](https://llvm.org/devmtg/2022-11/slides/Lightning6-RecoveringFromErrorsInClang-ReplAndCodeUndo.pdf) of Purva and Jun’s presentation.

## [Shared Memory Based JITLink Memory Manager](https://compiler-research.org/blogs/gsoc22_ghosh_experience_blog/)

Developer: **[Anubhab Ghosh](https://github.com/argentite)**( _Computer Science and Engineering, Indian Institute of Information Technology, Kalyani, India_)

Mentors: **Stefan Gränitz** ( _Freelance Compiler Developer, Berlin, Deutschland_), **Lang Hames** ( _Apple_), **Vassil Vassilev** ( _Princeton University/CERN_)

Funding: **Google Summer of Code 2022**

Anubhab introduced a new [JITLinkMemoryManager](https://llvm.org/docs/JITLink.html) based on a MemoryMapper abstraction that is capable of allocating JIT code(and data) using shared memory. The following advantages should arise from theimplemented strategy: 1. a faster transport (and access) for code and data whenrunning JIT’d code in a separate process on the same machine, and 2. theguarantee that all allocations are close together in memory and meet theconstraints of the default code model allowing the use of outputs from regularcompilers.Here you can find Anubhab’s GSoC [final report](https://compiler-research.org/assets/docs/Anubhab_Ghosh_GSoC2022_Report.pdf).

**Hurray!** This project has been accepted as a Lightning Talk at the [2022 LLVM Developers’ Meeting](https://llvm.swoogo.com/2022devmtg/speakers)!The conference took place in San Jose, California, from 7th to 10th November2022.Here you can find the [video](https://www.youtube.com/watch?v=dosXtBAFWiE) and the [slides](https://llvm.org/devmtg/2022-11/slides/Lightning4-EfficientJIT-basedRemoteExecution.pdf) of Anubhab’s presentation.

## [Optimize ROOT use of modules for large codebases](https://hepsoftwarefoundation.org/gsoc/blogs/2022/blog_ROOT_JunZhang.html)

Developer: **[Jun Zhang](https://github.com/junaire)**( _Software Engineering, Anhui Normal University, WuHu, China_)

Mentors: **David Lange** ( _Princeton University_), **Alexander Penev**( _University of Plovdiv Paisii Hilendarski, Bulgaria_), **Vassil Vassilev**( _Princeton University/CERN_)

Funding: **Google Summer of Code 2022**

The current performance of the modules usage in [ROOT](https://root.cern/) was evaluated, and a strategy for optimizing the memory footprint was developed.This strategy includes reducing unnecessary symbol-lookups and module-loading,and is especially useful for very large codebases like [CMSSW](https://cms-sw.github.io/),where Jun reduced the amount of memory by half (from 1.1GB to 600MB) for simpleworkflows like hsimple.C, and the number of loaded from 180 to 52.Here you can find Jun’s GSoC [final report](https://compiler-research.org/assets/docs/Jun_Zhang_GSoC22_Report.pdf).

## [Add Initial Integration of Clad with Enzyme](https://hepsoftwarefoundation.org/gsoc/blogs/2022/blog_CompilerResearch-ManishKausik.html)

Developer: **[Manish Kausik H](https://github.com/Nirhar)**( _.Tech and M.Tech in Computer Science and Engineering(Dual Degree),Indian Institute of Technology Bhubaneswar_)

Mentors: **David Lange** ( _Princeton University_), **William Moses**( _Massachusetts Institute of Technology_), **Vassil Vassilev** ( _Princeton University/CERN_)

Funding: **Google Summer of Code 2022**

[Clad](https://clad.readthedocs.io/en/latest/index.html) is an opensource plugin to the Clang compiler that enables [Automatic Differentiation](https://en.wikipedia.org/wiki/Automatic_differentiation) for C++. Clad receives an Abstract Syntax Tree (AST) from the underlying

compiler platform (Clang), decides whether a derivative is requested andproduces it, and modifies the AST to insert the generated code. [Enzyme AD](https://enzyme.mit.edu/) is an LLVM based AD plugin.It works by taking existing code as LLVM IR and computing the derivative(and gradient) of that function.In this project, Manish integrated Enzyme within Clad, giving a Clad user theoption of selecting Enzyme for Automatic Differentiation on demand. Theintegration of Clad and Enzyme t results in a new tool that offers an optimizedand flexible implementation of automatic differentiation.Here you can find Manish’s GSoC [final report](https://compiler-research.org/assets/docs/Manish_Kausik_H_GSoC22_Report.pdf).

## Thank you, developers!

We hope our interns contributors enjoyed our community. Our best reward is toknow that we supported your early steps into the world of open-source softwaredevelopment and compiler construction. We hope that this experience motivatedyou to continue to be involved with the broad LLVM ecosystem.We express our gratitude to the [Google Summer of Code](https://summerofcode.withgoogle.com/) Program and to the Institute for Research and Innovation in Software for HighEnergy Physics ( [IRIS-HEP](https://iris-hep.org/)) for supporting ourresearch and providing six young developers this wonderful opportunity.A special thanks goes to the LLVM community for being supportive and for sharingtheir knowledge and introducing our community to the new contributors. Thank youso much!Thanks for choosing Compiler-Research for your internship! We were lucky tohave you!
