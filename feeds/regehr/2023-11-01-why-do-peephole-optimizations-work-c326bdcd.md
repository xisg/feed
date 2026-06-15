---
title: Why Do Peephole Optimizations Work?
url: https://blog.regehr.org/archives/2485
published: "2023-11-01T16:23:20Z"
feed: regehr
guid: https://blog.regehr.org/?p=2485
---

# Why Do Peephole Optimizations Work?

In [its original form](https://dl.acm.org/doi/10.1145/364995.365000), a peephole optimization applied to a collection of instructions located close together in a program. For example, in a register transfer language we might find this sequence of instructions:

```
r0 = xor r8, -1
r1 = xor r9, -1
r0 = and r0, r1
```

Here, assuming the two’s complement representation, -1 is just a convenient way to specify a register filled with ones. Following a De Morgan law, a peephole optimization could rewrite this sequence as:

```
r0 = or r8, r9
r0 = xor r0, -1
```

But modern compilers don’t use RTL as their primary intermediate representation (IR), they’re more likely to have an SSA-based IR that represents the flow of data between instructions explicitly. You can think of SSA as teasing out a purely functional subset of a program that you write. For example in LLVM IR we might have:

```
%x = xor %a, -1
...
%y = xor %b, -1
...
%z = and %x, %y
```

Here the instructions need not be located near each other, they could even live in different basic blocks; it is only required that %x and %y both _dominate_ %z — this just means that there’s no way to get to %z without having run across definitions for both %x and %y. There’s a bit more to SSA than this, but the rest doesn’t matter to us right now. Here, again, we could use the De Morgan law to rewrite this computation and reduce the overall instruction count by one:

```
%tmp = or %a, %b
%z = xor %tmp, -1
```

These kinds of localized rewrites are intuitively appealing: they look like they should work. And they do! But let’s make sure we understand why.

First, let’s talk about what a peephole optimization, in isolation, should do. A strong correctness requirement would be that the new code is equivalent to the old code. That is, for all possible values of the inputs to the code that’s going to be optimized, the unoptimized and optimized code have to have the same observable behavior. Here we have to be careful about what we mean by “observable behavior” — it should include the value actually computed by the code in addition to any side effects, for example on the state of memory. We usually _do not want_ the execution time of the program, nor its code size, to be part of its observable behavior, since preserving those behaviors would preclude all optimizations for speed or size. We also don’t want microarchitectural effects, such as changes to the state of branch predictors and caches, to be observable; though in certain security-sensitive applications, we might change our minds about this. But anyway, however we define it, equivalence is a stronger criterion than we want optimizations to obey. Rather, we want the optimized code to _refine_ the unoptimized code.

Refinement roughly means “take a description of a system and produce a more specific version of that system,” and it is an ubiquitous concept in programming. We often start with a rough design for a software system and then refine it into a specification, which we then refine into an implementation. A compiler then refines the source-level implementation when it converts it into object code. If the compiler ever produces a non-refinement, this is considered to be a compiler bug. (We can flip this around and say that producing a refinement is the top-level correctness requirement for a compiler.) At the level of the compiler, non-trivial refinements correspond to removing possible behaviors from the code. For example, consider a high-performance parallel sorting algorithm that isn’t stable: it doesn’t necessarily preserve the ordering of elements whose sort keys compare the same. Depending on how the parallel execution goes, this algorithm might produce different outputs different times it’s run, but the different unstably-sorted outputs are all considered to be correct. Now consider a compiler that takes this algorithm and compiles it to a sequential platform, meaning that the entire sorting algorithm runs without concurrency, on a single core. On this platform we would expect that, for a given unsorted input, the sorting algorithm will produce the same (unstable) result every time that it is run. The compiler has eliminated non-determinism, and we generally consider that to be an OK thing for a compiler to do. This is an example of a compiler performing a non-trivial refinement.

Refinement also happens for sequential code. For example, the size of the “int” type in the C++ language is implementation-defined. One C++ compiler might refine an int in the source code to a 32-bit value, whereas a different one might refine it to a 64-bit value. Unspecified behavior in C and C++, such as the order of evaluation of arguments to a function, is also refined into some specific order at compile time.

For peephole optimizations, refinement is generally about undefined behavior. This isn’t the evil kind of programmer-facing undefined behavior that we often read about; this is specific to the compiler IR and it allows the compiler to perform interesting optimizations even when, for example, we’re using LLVM to compile Swift or Rust. [We wrote somewhat extensively about this topic](https://users.cs.utah.edu/~regehr/papers/undef-pldi17.pdf) a few years ago. [Here’s an example](https://gcc.godbolt.org/z/PjYee78j5) of LLVM’s peephole pass “InstCombine” performing a non-trivial refinement where it takes an immediate undefined behavior, dividing by zero, and turns it into a function returning LLVM’s poison value. Then, we can see an LLVM backend [further refining the poison value](https://gcc.godbolt.org/z/W56jxajsY) into code that returns whatever value happened to be sitting in w0, which is the register that the AArch64 ABI uses to return a 32-bit integer value. The backend could also have refined this function to return a constant such as zero, but that would have been less efficient.

So now we hopefully understand what peephole optimizations are supposed to do: the optimized code must refine the original code, in a context where “refinement” means something like “for defined inputs, preserve the behavior of the code; for undefined inputs, feel free to pick a subset of the behaviors.” A much more precise definition can be found in [the Alive2 paper](https://users.cs.utah.edu/~regehr/alive2-pldi21.pdf). But we still haven’t figured out why peephole optimizations work. The thing we’re going to need to do here is show that a refinement at the site of a peephole optimization (that is, buried somewhere deep in the code) leads to a refinement at the whole program level.

Let’s start with a function _f_ that contains some peephole-optimizable code. Logically, we’ll split it into two functions: a tiny little one, _o_, that contains the instructions that are going to get optimized, and a larger residual function, _r_, that contains that rest of the code. Composing the two new functions together is equivalent to the original function: _f_ = _r_( _o_). Now, by assumption, we have the optimized version of _o_ refining _o_: that is, _o_ → _o’_. (Read _o_ → _o’_ as “ _o_ is refined by _o_-prime.”) To move forward we need to know that refinement is compositional. In other words, if _x_ → _y_ then _g_( _x_) → _g_( _y_), for some arbitrary function _g_. We haven’t formally defined refinement, but any reasonable definition of it will have this property. Compositionality lets us prove that _r_( _o_) → _r_( _o_‘). And now, since _f_ = _r_( _o_), we get f → _r_(o’), and that is the function-level version of the result that we wanted in the first place. To get the whole program result we can keep applying the compositionality of refinement. And that’s why peephole optimizations work!

\[My students Zhengyang Liu and Manasij Mukherjee contributed to this piece. It happened because one time I remarked to Nuno Lopes that I wasn’t sure I actually understood why peephole optimizations work and he said “Oh yeah refinement is compositional.”\]
