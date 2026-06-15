---
title: (untitled)
url: https://blog.llvm.org/2015/04/llilc-llvm-based-compiler-for-dotnet.html
published: "2015-04-14T07:03:00Z"
feed: llvm
guid: https://blog.llvm.org/2015/04/llilc-llvm-based-compiler-for-dotnet.html
---

# (untitled)

## [LLILC](https://github.com/dotnet/llilc) : An LLVM based compiler for dotnet CoreCLR.

The LLILC project (we pronounce it "lilac") is a new effort started at Microsoft to produce MSIL code generators based on LLVM and targeting the open source dotnet [CoreCLR](https://github.com/dotnet/coreclr). We are envisioning using the LLVM infrastructure for a number of scenarios, but our first tool is a Just in Time (JIT) compiler for CoreCLR. This new project is being developed on GitHub and you can check it out [here](https://github.com/dotnet/llilc). The rest of this post outlines the rational and goals for the project as well as our experience using LLVM.

#### Why a new JIT for CoreCLR?

While the CoreCLR already has JIT, we saw an opportunity to provide a new code generator that has the potential to run across all the targets and platforms supported by LLVM. To enable this, as part of our project we're opening an MSIL reader that operates directly against the same common JIT interface as the production JIT (RyuJIT). This new JIT will allow any C# program written for the .NET Core class libraries to run on any platform that CoreCLR can be ported to and that LLVM will target.

#### There are several ongoing efforts to compile MSIL in the LLVM community, SharpLang springs to mind. Why build another one?

When we started thinking about the fastest way to get a LLVM based code generation working we looked around at the current open source projects as well as the code we had internally. While a number of the OSS projects already targeted LLVM BitCode, no one had anything that was a close match for the CoreCLR interface. Looking at our options it was simplest for us to refactor a working MSIL reader to target BitCode then teach a existing project to support the contracts and APIs the CoreCLR uses for JITing MSIL. Using a existing MSIL reader let us quickly start using a number of building-block components that we think the community can take advantage of. This fast bootstrap for C# across multiple platforms was the idea that was the genesis of this project and the compelling reason to start a new effort. We hope LLILC will provide a useful example - and reusable components - for the community and make it easier for other projects to interoperate with the CoreCLR runtime.

#### Why LLVM?

Basically we think LLVM is awesome. It's already got great support across many platforms and chipsets and the community is amazingly active. When we started getting involved, just trying to stay current with the developer mailing list was a revelation! The ability for LLVM to operate as both a JIT and as an AOT compiler was especially attractive. By bringing MSIL semantics to LLVM we plan to construct a number of tools that can work against CoreCLR or some sub set of its components. We also hope the community will produce tools what we haven't thought of yet.

#### Tool roadmap

- CoreCLR JIT
  - Just In Time - A classic JIT. This is expected to be throughput-challenged but will be correct and usable for bringup. Also possible to use with more optimization enabled as a higher tier JIT
  - Install-time JIT - What .NET calls NGen. This will be suitable for install-time JITing (LLVM is still slow in a runtime configuration)
- Ahead of Time compiler. A build lab compiler that produces standalone executables, using some shared components from CoreCLR. The AOT compiler will be used to improve startup time for important command line applications like the [Roslyn Compiler](https://github.com/dotnet/roslyn).

The LLIC JIT will be a functionally correct and complete JIT for the CoreCLR runtime. It may not have sufficient throughput to be a first-tier jit, but is expected to produce high-quality code and so might make a very interesting second-tier or later JIT, or a good vehicle for prototyping codegen changes to feed back into RyuJIT.

## What's Actually Working

Today on Windows we have the MSIL reader & LLVM JIT implemented well enough to compile a significant number of methods in the JIT bring up tests included in CoreCLR. In these tests we compile about 90% the methods and then fall back to RyuJIT for cases we can't handle yet. The testing experience is pretty decent for developers. The tests we run can be seen in the CoreCLR [test repo](https://github.com/dotnet/coreclr/tree/master/tests/src/JIT/CodeGenBringUpTests).

We've establish builds on Linux and Mac OSX and are pulling together mscorlib, the base .NET Core library from [CoreFx](https://github.com/dotnet/corefx), and test asset dependencies to get testing off-the-ground for those platforms.

All tests run against the CoreCLR GC in conservative mode - which scans the frame for roots - rather than precise mode. We don't yet support Exception Handling.

## Architecture

Philosophically LLILC is intended to provide a lean interface between CoreCLR and LLVM. Where possible we rely on preexisting technology.

[![](http://2.bp.blogspot.com/-5_tqqSVjMf4/VS0cQEhSHrI/AAAAAAAAAE0/kOOiDM4_XAs/s1600/JITArch.png)](http://2.bp.blogspot.com/-5_tqqSVjMf4/VS0cQEhSHrI/AAAAAAAAAE0/kOOiDM4_XAs/s1600/JITArch.png)

For the JIT, when we are compiling on demand, we map the runtime types and MSIL into LLVM BitCode. From there compilation uses LLVM MCJIT infrastructure to produce compiled code that is output to buffers provided by CoreCLR.

[![](http://1.bp.blogspot.com/-NUl_WpPgYL8/VS0cWHeLVlI/AAAAAAAAAE8/0zJmQGulwc8/s1600/AOTArch.png)](http://1.bp.blogspot.com/-NUl_WpPgYL8/VS0cWHeLVlI/AAAAAAAAAE8/0zJmQGulwc8/s1600/AOTArch.png)

Our AOT diagram is more tentative but the basic outline is that the code generator is driven using the same interface and the JIT but that there is a static type system behind it and we build up a whole program module with in LLVM and run in a LTO like mode. Required runtime components are emitted with the output objs and the platform linker then produces the target executable. There are still a number of open questions around issues like generics that need resolution but this is our first stake in the ground.

## Experience with LLVM

In the few months we've been using LLVM, we've had a really good experience but with a few caveats. Getting started with translating to BitCode has been a very straightforward experience and ramp-up time for someone with compiler experience has been very quick. The MCJIT, which we started with for our JIT, was easy to configure and get code compiled and returned to the runtime. Outside of the COFF issue discussed below, we only had to make adjustments in configuration or straightforward overrides of classes, like EEMemoryManager, to enable working code. Of the caveats, the first was simple, but the other two are going to require sustained work to bring up to the level we'd like. The first issue was a problem with Windows support in the DynamicRuntime of the MCJIT infrastructure. The last two, Precise Garbage Collection, and Exception Handling, arise because of the different semantics of managed languages. Luckily for us, people in the community have already started working in these areas so we don't have to start from zero.

#### COFF dynamic loader support

One of the additions we made to LLVM to unblock ourselves was to implement a COFF dynamic loader. (The patch to add RuntimeDyldCoff.{h,cpp} is through review and has been committed). This was the only addition we directly had to make to LLVM to enable bring-up of the code generator. Once this is in, we see a number of bugs in the database around Windows JIT support that should be easier to close.

#### Precise Garbage Collection

Precise GC is central to the CoreCLR approach to memory management. Its intent is to keep the overhead of managed memory as low as possible. It is assumed by the runtime that the JIT will generate precise information about the GC ref lifetimes and provide it with the compiled code for execution. To support this we're beginning to use the [StatePoint](http://llvm.org/docs/Statepoints.html) approach, with additions to convert the standard output format to the custom format expected by CoreCLR. We share some of the same concerns that Philip Reames wrote about in the initial design of StatePoints. E.g. preservation of "GCness" through the optimizer is critical, but must not block optimizer transformations. Given this concern one of our open questions is how to enable testing to find GC holes that creep in, but also enable extra checking that can be opted into if the incoming IR contains GC pointers.

There is a more detailed document included in our repo that outlines our more-specific GC plans [here](https://github.com/dotnet/llilc/blob/master/Documentation/llilc-gc.md).

#### Exception Handling

The MSIL EH model is specific to the CLR as you'd expect, but it descends in part conceptually from Windows Structured Exception Handling (SEH). In particular, the implicit exception flow from memory accesses to implement null checks, and the use of filters and funclets in the handling of exceptions, mirrors SEH ( [here](https://msdn.microsoft.com/en-us/library/ms173162.aspx). is an outline of C# EH) Our plans at this point are to add all checks required by MSIL as explicit compare/branch/throw sequences to better match C++ EH as well as building on the SEH support currently being put into Clang. Then, once we have correctness, see if there is a reasonable way forward that improves performance.

Like GC, there's a detailed doc outlining our specific issues and plans in the repo [here](https://github.com/dotnet/llilc/blob/master/Documentation/llilc-jit-eh.md)

## Future Work

- More platforms. Today we're running on Windows and starting to build for Linux and Mac OSX. We'd like more.
- Complete JIT implementation
  - More MSIL opcodes supported
  - Precise GC support
  - EH support
- Specialized memory allocators for hosted solutions. CoreCLR has been used as a hosted solution (run in process by other programs) but to support this we need a better memory allocation story. The runtime should be able to provide a memory allocator that is used for all compilation.
- AOT - Fully flesh out the AOT story.

#### Links

[LLILC](https://github.com/dotnet/llilc)

[CoreCLR](https://github.com/dotnet/coreclr)

[CoreFx](https://github.com/dotnet/corefx)

[.NET Foundation](http://www.dotnetfoundation.org/)
