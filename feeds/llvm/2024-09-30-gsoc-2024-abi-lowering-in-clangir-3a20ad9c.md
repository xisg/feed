---
title: 'GSoC 2024: ABI Lowering in ClangIR'
url: https://blog.llvm.org/posts/2024-09-07-abi-lowering-in-clangir/
published: "2024-09-30T00:00:00Z"
feed: llvm
guid: https://blog.llvm.org/posts/2024-09-07-abi-lowering-in-clangir/
---

# GSoC 2024: ABI Lowering in ClangIR

ClangIR is an ongoing effort to build a high-level intermediate representation(IR) for C/C++ within the LLVM ecosystem. Its key advantage lies in its abilityto retain more source code information. While ClangIR is making progress, itstill lacks certain features, notably ABI handling. Currently, ClangIR lowersmost functions without accounting for ABI-specific calling convention details.

## Goals

The “Build & Run SingleSource Benchmarks with ClangIR - Part 2” Google Summer ofCode 2024 builds on my contributions from GSoC 2023 by addressing one of themain issues I encountered: target-specific lowering. It focuses on extendingClangIR’s code generation capabilities, particularly in ABI-lowering for X86-64.Several tests rely on operations and types (e.g., `va_arg` calls and complexdata types) that require target-specific information to compile correctly.

The concrete steps to achieve this were:

1. **Implement foundational infrastructure** that can scale to multiplearchitectures while adhering to ClangIR design principles such as CodeGenparity, feature guarding, and AST backreferences.
2. **Handle basic calling convention scenarios** as a proof of concept tovalidate the foundational infrastructure.
3. **Add lowering for a second architecture** to further validate theinfrastructure’s extensibility to multiple architectures.
4. **Unify target-specific ClangIR lowering into the library**, as there are afew isolated methods handling target-specific code lowering like `cir.va_arg`.
5. **Integrate calling convention lowering into the main pipeline** to ensurefuture contributions and continued development of this infrastructure.

## Contributions

The list of contribution (PRs) can be found [here](https://github.com/llvm/clangir/pulls?q=is%3Apr+is%3Aclosed+author%3Asitio-couto+closed%3A%3E2024-05-01).

### Target Lowering Library

The most significant contribution of this project was the development of amodular [`TargetLowering` library](https://github.com/llvm/clangir/pull/643).This ensures that target-specific MLIR lowering passes can leverage this sharedlibrary for lowering logic. The library also follows ClangIR’s feature guardingprinciples, ensuring that any contributor can refer to the original CodeGen forcontributions, and any unimplemented feature is asserted at specific codepoints, making it easy to track missing functionality.

### Calling Convention Lowering Pass

As a proof of concept, the initial development of the `TargetLowering` libraryfocused on implementing a [calling convention loweringpass](https://github.com/llvm/clangir/pull/642) that targets multiplearchitectures. Currently, ClangIR ignores the target ABI during CodeGen toretain high-level information. For example, structs are not unraveled to improveargument-passing efficiency. ABI-specific LLVM attributes are also ignored. Thispass addresses these issues by properly tagging LLVM attributes and rewritingfunction definitions and calls to handle unraveled structs. This was implementedfor both X86-64 and [AArch64](https://github.com/llvm/clangir/pull/679),demonstrating the library’s multi-architecture support.

## Shortcomings

### Target-Specific Lowering Unification

While some target-specific lowering code was moved into the library, it wascopied and pasted rather than properly integrated. This is not ideal forleveraging the library’s multi-architecture features.

### Inclusion in the Main Pipeline

This is still a work in progress, as the library is not yet mature enough tohandle most pre-existing ClangIR tests. There are also feature guards withunreachable statements for many unimplemented features.

## Future Work

Now that there is a base infrastructure for handling target-agnostic totarget-specific CIR code, there is a large amount of future work to be done,including:

- Improving DataLayout-related queries using MLIR’s built-in tools.
- Implementing calling convention lowering for additional types, such aspointers.
- Extending the TargetLowering library to support more architectures.
- Unifying remaining target-specific lowering code from other parts of ClangIR.

## Acknowledgements

I would like to thank my Google Summer of Code mentors, Bruno Cardoso Lopes andNathan Lanza, for another great GSoC experience. I also want to thank the LLVMcommunity and Google for organizing the program.
