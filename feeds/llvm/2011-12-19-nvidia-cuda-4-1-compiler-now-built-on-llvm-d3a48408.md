---
title: NVIDIA CUDA 4.1 Compiler Now Built on LLVM
url: https://blog.llvm.org/2011/12/nvidia-cuda-41-compiler-now-built-on.html
published: "2011-12-19T14:28:00Z"
feed: llvm
guid: https://blog.llvm.org/2011/12/nvidia-cuda-41-compiler-now-built-on.html
---

# NVIDIA CUDA 4.1 Compiler Now Built on LLVM

From the NVIDIA CUDA compiler team:

> CUDA is a parallel programming model and platform created by NVIDIA for harnessing the power of hundreds of cores in modern graphics processing units (GPUs). NVIDIA provides free support for CUDA C and C++ in the CUDA toolkit. The CUDA programming environment consists of a compiler targeting NVIDIA GPUs and has been adopted by thousands of developers.
>
> At NVIDIA we have switched over to using LLVM inside the CUDA C/C++ compiler for Fermi and future architectures. We use LLVM for optimizations and PTX code generation and for generating debug information for CUDA debugging. From a developer's perspective the new compiler is functionally on par with the previous compilers and produces better code with better compile times. We have extended the LLVM core compiler to understand data parallel programming model. It is now available, as part of CUDA 4.1 and you can [learn more here](http://developer.nvidia.com/content/new-cuda-now-available).
>
> Our experience with the use of LLVM has been very positive, starting with a modern compiler infrastructure and with high quality optimizations contributed by a large community of developers. The effort required to learn LLVM infrastructure is quite small and reasonable.
