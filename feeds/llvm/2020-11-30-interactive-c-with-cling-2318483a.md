---
title: Interactive C++ with Cling
url: https://blog.llvm.org/posts/2020-11-30-interactive-cpp-with-cling/
published: "2020-11-30T10:00:00Z"
feed: llvm
guid: https://blog.llvm.org/posts/2020-11-30-interactive-cpp-with-cling/
---

# Interactive C++ with Cling

The C++ programming language is used for many numerically intensive scientificapplications. A combination of performance and solid backward compatibility hasled to its use for many research software codes over the past 20 years. Despiteits power, C++ is often seen as difficult to learn and inconsistent with rapidapplication development. Exploration and prototyping is slowed down by the longedit-compile-run cycles during development.

[Cling](https://github.com/root-project/cling/) has emerged as a recognizedcapability that enables interactivity, dynamic interoperability and rapidprototyping capabilities to C++ developers. Cling supports the full C++ featureset including the use of templates, lambdas, and virtual inheritance. Cling isan interactive C++ interpreter, built on top of the Clang and [LLVM](llvm.org) compiler infrastructure. The interpreter enables interactive exploration andmakes the C++ language more welcoming for research.

The main tool for storage, research and visualization of scientific data in thefield of high energy physics (HEP) is the specialized software package [ROOT](root.cern/). ROOT is a set of interconnected components that assistscientists from data storage and research to their visualization when publishedin a scientific paper. ROOT has played a significant role in scientificdiscoveries such as gravitational waves, the great cavity in the Pyramid ofCheops, the discovery of the Higgs boson by the Large Hadron Collider. For thelast 5 years, Cling has helped to analyze 1 EB physical data, serving as a basisfor over 1000 scientific publications, and supports software run across adistributed million CPU core computing facility.

Recently we started a project aiming to leverage our experience in interactiveC++, just-in-time compilation technology (JIT), dynamic optimizations, and largescale software development to greatly reduce the impedance mismatch between C++and Python. We will generalize Cling to offer a robust, sustainable andomnidisciplinary solution for C++ language interoperability.The scope of ourobjectives is to:

- advance the interpretative technology to provide a state-of-the-art C++execution environment,
- enable functionality which can provide native-like, dynamic runtimeinteroperability between C++ and Python (and eventually other languages suchas Julia and Swift)
- allow seamless utilization of heterogeneous hardware (such as hardwareaccelerators)

Project results will be integrated into the widely used tools LLVM, Clang andCling. The outcome of the proposed work is a platform which provides a C++compiler as a service (CaaS) for both rapid application development andcomputational performance.

The rest of this post intends to demonstrate the design and several features ofCling. Want to follow along? You can get cling from conda

```
conda config --add channels conda-forgeconda install clingconda install llvmdev=5.0.0
```

or from docker-hub if you don’t already use conda:

```
docker pull compilerresearch/clingdocker run -t -i compilerresearch/cling
```

Either way, type “cling” to start its interactive shell:

```
cling****************** CLING ******************* Type C++ code and press enter to run it ** Type .q to exit ********************************************[cling]$[cling]$ #include "cling/Interpreter/Interpreter.h"[cling]$ gCling->allowRedefinition(false)
```

We will explain the purpose for these commands, and other alternatives for usingcling in further parts of this post.

# Interpreting C++

Exploratory programming (or Rapid Application Development) is an effective wayto gain understanding of the requirements for a project; to reduce thecomplexity of the problem; and to provide an early validation of the systemdesign and implementation. In particular, interactively probing data andinterfaces makes complex libraries and complex data more accessible to users.It is important in data science, computational science and debugging. Itsignificantly reduces the time consumed by edit-run cycles during development.In practice, only few programming languages offer both a compiler and aninterpreter translating them into machine code, although whether a language isto be interpreted or compiled is a property of the implementation.

Languages which enable exploratory programming tend to have interpreters whichshorten the compile-link cycle; this generally has a noticeable cost inperformance. Language developers who acknowledge the use case of exploratoryprogramming may also put syntactic sugar, but that is mostly for convenience andterseness. The performance penalty is largely mitigated by using just-in-time(JIT) or ahead-of-time (AOT) compilation technology.

For the sake of this post series, interpreting C++ means enabling exploratoryprogramming for C++ while mitigating the performance cost with JIT compilation.Figure 1 shows an illustrative example of exploratory programming. It becomestrivial to orient the shape, choose size and color or compare to previoussettings. The invisible compile-link cycle aids interactive use which allowssome qualitatively different approaches to program development and enhancedproductivity.

![](https://blog.llvm.org/img/cling-2020-11-30-figure1.gif)

Figure 1. Interactive OpenGL Demo, adapted from \[here\](https://www.youtube.com/watch?v=eoIuqLNvzFs).

## Design principles

Some of the design goals of cling include:

- Do not pay for what you do not use – prioritize performance of processingcorrect code. For example, in order to provide error recovery do not penalizeusers typing syntactically and semantically correct C++; and interactive C++transformations are only done when necessary and can be disabled.
- Reuse Clang & LLVM at (almost) any cost – do not reinvent the wheel. If afeature is not available, then try finding a minimalistic way to implement itand propose it for a review to the LLVM community. Otherwise find the minimalpatch, even at the cost of misusing API, which satisfies the requirements.
- Continuous feature delivery – focus on a minimal feature, its integrationin the main use-case (ROOT), deployment in production, repeat.
- Library design – allow Cling to be used as a library from third partyframeworks.
- Learn and evolve – experiment with user experience. There is no formalspecification or consensus on overall user experience. Apply lessons learnedfrom the legacy from CINT.

## Architecture

Cling accepts partial input and ensures that the compiler process keeps runningto act on code as it comes in. It includes an API providing access to theproperties of recently compiled chunks of code. Cling can apply customtransformations to each chunk before execution. Cling orchestrates the existingLLVM and Clang infrastructure following a data flow described in Figure 2.

![](https://blog.llvm.org/img/cling-2020-11-30-figure2.svg)

Figure 2. Information flow in Cling

In short:

1. The tool controls the input infrastructure by interactive prompt or by aninterface allowing the incremental processing of input (➀).
2. It sends the input to the underlying clang library for compilation (➁).
3. Clang compiles the input, possibly wrapped into a function, into an AST (➂).
4. When necessary the AST is further transformed in order to attach specificbehavior (➃).

For example, reporting execution results, or other interpreter-related features.Once the high-level AST representation is ready, it is sent for lowering to anLLVM-specific assembly format, the LLVM IR (➄). The LLVM IR is the input formatfor LLVM’s just-in-time compilation infrastructure. Cling instructs the JIT torun specified functions (➅), translating them into machine code (MC) targetingthe underlying device architecture (eg. Intel x86 or NVPTX) (➆,➇).

The C++ standard is developed towards compilers and does not cover interactiveuse well. Execution of statements on the global scope, reporting executionresults, and entity redefinitions are the three most important features when itcomes to user friendliness. Long running interpreter sessions are prone totyping errors and make flawless error recovery essential. More advanceduse-cases require extra flexibility at runtime and lookup rules extensionsaiding eval-style programming. Efficient watermark-based code removal isimportant when C++ is used as scripting language.

## Execution of statements

Cling processes C++ incrementally. Incremental input consists of one or multipleC++ statements. C++ does not allow expressions in the global scope.

```cpp
[cling] #include <vector>[cling] #include <iostream>[cling] std::vector<int> v = {1,2,3,4,5}; v[0]++;[cling] std::cout << "v[0]=" << v[0] <<"\n";v[0]=2
```

Instead, Cling moves each input into a unique wrapper function. Eg:

```cpp
void __unique_1 () { std::vector<int> v = {1,2,3,4,5};v[0]++;; } // #1void __unique_2 () { std::cout << "v[0]=" << v[0] <<"\n";; } // #2
```

After the clang AST is built, cling detects that wrapper #1 contains adeclaration and moves the declaration’s AST node to the global scope, such that `v` can be referenced by subsequent inputs. Wrapper #2 contains a statement andis executed as is. Internally to Cling, the example is transformed to:

```cpp
#include <vector>#include <iostream>std::vector<int> v = {1,2,3,4,5};void __unique_1 () { v[0]++;; }void __unique_2 () { std::cout << "v[0]=" << v[0] <<"\n";; }
```

Cling runs these wrappers after they are compiled to machine code.

## Reporting execution results

An integral part of interactivity is printing expression values. Typing `printf` each time is laborious and does not naturally include object type information.Instead, omitting the semicolon of the last statement of the input tells Clingto report the expression result. When wrapping the input, Cling textuallyattaches a semicolon to the end of it. If an execution report is requested thecorresponding wrapper AST does not contain a _NullStmt_ (modelling extrasemicolons).

```cpp
[cling] #include <vector>[cling] std::vector<int> v = {1,2,3,4,5} // Note the missing semicolon(std::vector<int> &) { 1, 2, 3, 4, 5 }
```

A transformation injects extra code depending on the properties of theparticular entity such as if it is copyable, if it is a wrapper temporary or anarray. Cling can report information about non-copyable or temporary objects byproviding a ‘managed’ storage. The managed storage ( _cling::Value_) is also usedfor exchanging values between interpreted and compiled code in embedded setup.

## Entity Redefinition

Name redefinition is an important scripting feature. It is also essential fornotebook-based C++ as each cell is a somewhat separate computation. C++ does notsupport redefinitions of entities.

```bash
[cling] #include <string>[cling] std::string v(std::string &) ""[cling] #include <vector>[cling] std::vector<int> vinput_line_7:2:19: error: redefinition of 'v' with a different type: 'std::vector<int>' vs 'std::string' (aka 'basic_string<char, char_traits<char>, allocator<char> >') std::vector<int> v ^input_line_4:2:14: note: previous definition is here std::string v ^
```

Cling implements entity redefinition using inline namespaces and rewires clanglookup rules to give higher priority to more recent declarations. The fulldescription of this feature was published as a conference paper on CC 2020( [ACM conference on Compiler Construction](https://doi.org/10.1145/3377555.3377901)).We enable it by calling `gCling->allowRedefinition()`:

```bash
[cling] #include "cling/Interpreter/Interpreter.h"[cling] gCling->allowRedefinition()[cling] #include <vector>[cling] std::vector<int> v(std::vector<int> &) {}[cling] #include <string>[cling] std::string v(std::string &) ""
```

## Invalid Code. Error Recovery

When used in interactive mode, invalid C++ does not terminate the session.Instead invalid code is discarded. The underlying clang process keeps theinvalid AST nodes in its internal data structures for better error diagnosticsand recovery, expecting the process will end shortly after issuing thediagnostics. This particular example is more challenging because it firstcontains both valid and invalid constructs. The error recovery should undo asignificant amount of changes in internal structures such as the name lookup andthe AST. Cling is used in many high-performance environments; usingcheckpointing is not a viable option as it introduces overhead for correct code.

```bash
[cling] #include <vector>[cling] std::vector<int> v; v[0].error_here;input_line_4:2:26: error: member reference base type 'std::__1::__vector_base<int, std::__1::allocator<int> >::value_type' (aka 'int') is not a structure or union std::vector<int> v; v[0].error_here; ~~~~^~~~~~~~~~~
```

In order to handle the example, Cling models the incremental input into a _Transaction_. A transaction represents the delta of the changes of internaldata structures of Clang. Cling listens to events coming from various Clangcallbacks such as declaration creation, deserialization and macro definition.This information is sufficient to undo the changes and continue with a validstate. The implementation is very intricate and in many cases requires extrawork depending on the input declaration kinds.

Cling also protects against null pointer dereferences via a code transformation,avoiding a session crash.

```bash
[cling] int *p = nullptr; *pinput_line_3:2:21: warning: null passed to a callee that requires a non-null argument [-Wnonnull] int *p = nullptr; *p ^[cling]
```

The implementation of error recovery and code unloading still has rough edgesand it is being improved constantly.

## Code Removal

Incremental, interactive C++ assumes long lived sessions where not only syntaxerror can happen but also semantic ones. That poses one level of extracomplexity if we want to re-execute the same code with minor adjustments.

```cpp
[cling] .L Adder.h // #1, similar to #include "Adder.h"[cling] Add(3, 1) // int Add(int a, int b) {return a - b; }(int) 2[cling] .U Adder.h // reverts the state prior to #1[cling] .L Adder.h[cling] Add(3, 1) // int Add(int a, int b) {return a + b; }(int) 4
```

In the example, we include a header file with the _.L_ meta command;“uninclude” it with _.U_ and “reinclude” it with _.L_ to re-read the modifiedfile. Unlike in the error recovery case, Cling cannot fence the machine codelowering infrastructure and needs to undo state changes in clang CodeGen and thellvm JIT and machine code infrastructure. The implementation of this featurerequires expertise in a big portion of the LLVM toolchain.

# Conclusion

Cling has been one of the systems enabling interactive C++ for more than a decade.Cling’s extensibility and fast prototyping features is of fundamental importancefor researchers in high-energy physics, and an enabler for many of thetechnologies that they rely on. Cling has several unique features tailored tothe challenges which come with incremental C++. Our work on interactive C++ isalways evolving. In the next blog post we will focus on interactive C++ for DataScience; Eval-Style Programming; Interactive CUDA; and C++ in notebooks.

You can find out more about our activities at [https://root.cern/cling/](https://root.cern/cling/) and [https://compiler-research.org](https://compiler-research.org).

# Acknowledgements

The author would like to thank Sylvain Corlay, Simeon Ehrig, David Lange,Chris Lattner, Javier Lopez Gomez, Wim Lavrijsen, Axel Naumann, Alexander Penev,Xavier Valls Pla, Richard Smith, Martin Vassilev, who contributed to this post.
