---
title: Another step forward towards interactive programming
url: https://blog.llvm.org/posts/2023-12-31-compiler-research-internships-2023/
published: "2023-12-31T00:00:00Z"
feed: llvm
guid: https://blog.llvm.org/posts/2023-12-31-compiler-research-internships-2023/
---

# Another step forward towards interactive programming

The [Compiler Research](https://compiler-research.org) team is pleased to announce the successful completionof another round of internships focused on enhancements in interactiveprogramming, specifically in relation to the [Clang-REPL](https://clang.llvm.org/docs/ClangRepl.html) component in LLVM.

The Compiler Research team includes researchers located at Princeton Universityand CERN. Our primary goal is best described as follows:

> To establish a proficient workflow in LLVM, where [interactive development](https://compiler-research.org/interactive_cpp) inC++ is possible, and exploratory C++ becomes an accessible experience to awider audience.

Following are some notable contributions by our interns this year.

## Yuquan Fu - Autocompletion in Clang-REPL

Clang-Repl allows developers to program in C++ interactively with a REPLenvironment. However, it was missing the ability to suggest code completion orauto-complete options for user input, which can be time-consuming and prone totyping errors.

With this code completion system, users can either complete their input quicklyor see a list of valid completion candidates. The code completion is alsocontext-aware, providing semantically relevant results based on the currentposition and input on the current line.

Mentors: Vassil Vassilev ( [Princeton.edu](https://www.princeton.edu/)) & David Lange ( [Princeton.edu](https://www.princeton.edu/))

Project Details: [Autocompletion in Clang-REPL](https://www.syntaxforge.net/clang-repl-cc/)

Funding: Google Summer of Code 2023

### Example – avoiding tedious typing

```
clang-repl> struct WhateverMeaningfulLoooooooooongName{ int field;};clang-repl> Wh<tab>
```

With code completion, hitting tab completes the entity name:

```
clang-repl> WhateverMeaningfulLoooooooooongName
```

> For implementation details, please see the respective [slides](https://compiler-research.org/assets/presentations/CaaS_Weekly_30_08_2023_Fred-Code_Completion_in_ClangRepl_GSoC.pdf) and the [blog](https://compiler-research.org/blogs/gsoc23_ffu_experience_blog/).

## Anubhab Ghosh - WebAssembly Support for Clang-Repl

The Xeus Framework enables accessing Clang-REPL (an interpreter that JITcompiles C++ code into native code) in a web browser, using Jupyter. However,this shifts the computational load to the server.

A more scalable approach is to use WebAssembly. It allows sandboxed executionof native (e.g. C/C++/Rust) programs compiled to an intermediate bytecode atcloser to native speeds. The idea is to run clang-repl within WebAssembly andgenerate JIT-compiled WebAssembly code and execute it on the client side.

However, this comes with some challenges (e.g., code in WebAssembly isimmutable, which is unacceptable for JIT).

Solution: To address the code immutability issue, a new WebAssembly module iscreated at each iteration of the REPL loop. Initially, a precompiled modulecontaining the Standard C/C++ libraries, LLVM, Clang, and wasm-ld is sent tothe browser, which runs the interpreter and compiles the user code.

Since we cannot call Interpreter::Execute() to execute the module (due toJITLink reliance), the LLVM WebAssembly backend is used manually to produce anobject file. This file is then passed to the WebAssembly version of LLD(wasm-ld) to turn it into a shared library which is written to the virtual filesystem of Emscripten. The dynamic linking facilities of Emscripten can be usedto load this library.

Mentors: Vassil Vassilev ( [Princeton.edu](https://www.princeton.edu/)) & Alexander Penev ( [Uni-Plovdiv.bg](https://uni-plovdiv.bg/en/))

Project Details: [WebAssembly Support for Clang-Repl](https://gist.github.com/argentite/c0852d3e178c4770a429f14291e83475)

Funding: Google Summer of Code 2023

### Example:

```
SDL_Init(SDL_INIT_VIDEO);SDL_Window *window;SDL_Rendered *renderer;SDL_CreateWindowAndRenderer (300, 300, 0, &window, &renderer);
```

This should connect to a simple black canvas. Next, we can draw things into it.

```
SDL_SetRenderDrawColor(renderer, 0x80, 0x00, 0x00, 0xFF);SDL_Rect rect3 = {.x = 20, .y = 20, .w = 150, .h = 100};SDL_RenderFillRect(rendered, &rect3); SDL_SetRenderDrawColor(renderer, 0x00, 0x80, 0x00, 0xFF);SDL_Rect rect4 = {.x = 40, .y = 40, .w = 150, .h = 100};SDL_RenderFillRect(rendered, &rect4); SDL_RenderPresent(renderer);
```

The output should look something like this:

![Web Assembly Example](https://blog.llvm.org/img/WebAssemblyExample.png)

## Sunho Kim - Re-optimization using JITLink

In order to support re-optimization, the JITLink API was extended by adding thecross-architecture stub creation API. This API works in all platforms andarchitectures that JITLink supports and through this we can create theredirectable stubs by using JITLink.

Once the re-optimization API was developed, it was time to actually implementre-optimization. A new layer was introduced to support re-optimization of IRmodules. There were many abstraction levels where redirection could beimplemented, but we ended up doing it at IR level since that brings a lot ofre-optimization techniques to be implemented easily by transforming IRdirectly. From an API perspective, the most flexible abstraction level to dothis may be at the FrontEnd AST level.

Clang-Repl relies on LLJIT to do JIT-related tasks. Enabling re-optimizationfor LLJIT also helped enable it in Clang-Repl. However, there were minorchallenges (e.g., mismatch in what clang-repl expects from how the runtimeexecutes the static initializers and how ELF orc runtime runs it). Possiblesolutions for these are in discussion (e.g., adding a new dl function).Nevertheless, we now have a real-world experimental environment where we cantest new re-optimization techniques and perform benchmarks to see if they areuseful.

Finally, based on the above infrastructure, profile guided optimization is nowpossible (by transforming the IR module). There are still some enhancementspending before the code is fully upstreamed, but the current code achievesinstrumentation on the orc-runtime side, which simplifies implementation by alot.

Mentors: Vassil Vassilev ( [Princeton.edu](https://www.princeton.edu/)) & Lang Hames/ lhames ( [Apple](https://www.apple.com))

Project Details: [Re-optimization using JITLink](https://gist.github.com/sunho/bbbf7c415ea4e16d37bec5cea8adce5a)

Funding: Google Summer of Code 2023

### Example: Doing the -O2 optimization if function was called more than 10 times

The following example builds a PassManager using the LLVM library and then runsthe optimization pipeline.

```
static Error reoptimizeTo02(ReOptimizeLayer &Parent, ReOptMaterializationUnitID MUID, unsigned Curverison, ResourceTrackerSP OldRT, ThreadSafeModule &TSM) { TSM.withModuleDo([&]{llvm::Module &M) { auto PassManager = buildPassManager(); PassManager.run(M); }); return Error::success();}ReOptLayer ->setReoptimizeFunc(reoptimizeTo02);ReOptLayer ->setAddProfileFunc(reoptimizeIfCallFrequent);
```

For more examples, please see the [LLVM-JITLink-COFF-Example](https://github.com/sunho/LLVM-JITLink-COFF-Example) repo.

## Krishna Narayanan - Tutorial development with clang-repl

Open Source documentation is often a neglected area in the software lifecycle.Specifically, this project targeted helping contributors by documenting howthey can set up respective environments on their local machines to contributeto the code and documentation of the respective project. These environmentswere set up locally, tested and then the setup methodology was updated in therelevant documentation.

Besides other compiler research technologies, write-ups were also added to LLVM(specifically the Clang-Repl documentation) as part of this project. Usageexamples were also added.

Mentors: Vassil Vassilev ( [Princeton.edu](https://www.princeton.edu/)) & David Lange ( [Princeton.edu](https://www.princeton.edu/))

Project Details: [Tutorial development with clang-repl](https://github.com/Krishna-13-cyber/GSoC23-LLVM/blob/main/README.md)

Funding: Google Summer of Code 2023

### Example

```
// Classes and Structuresclang-repl> #include <iostream>clang-repl> class Rectangle {int width, height; public: void set_values (int,int);\clang-repl... int area() {return width*height;}};clang-repl> void Rectangle::set_values (int x, int y) { width = x;height = y;}clang-repl> int main () { Rectangle rect;rect.set_values (3,4);\clang-repl... std::cout << "area: " << rect.area() << std::endl;\clang-repl... return 0;}clang-repl> main();area: 12
```
