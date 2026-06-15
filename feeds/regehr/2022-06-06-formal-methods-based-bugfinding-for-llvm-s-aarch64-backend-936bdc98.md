---
title: Formal-Methods-Based Bugfinding for LLVM’s AArch64 Backend
url: https://blog.regehr.org/archives/2265
published: "2022-06-06T14:58:02Z"
feed: regehr
guid: https://blog.regehr.org/?p=2265
---

# Formal-Methods-Based Bugfinding for LLVM’s AArch64 Backend

\[This piece is co-authored by Ryan Berger and Stefan Mada (both Utah CS undergrads), by Nader Boushehri, and by John Regehr.\]

An optimizing compiler traditionally has three main parts: a frontend that translates a source language into an intermediate representation (IR), a “middle end” that rewrites IR into better IR, and then a backend that translates IR into assembly language. My group and our collaborators have spent a lot of time finding and reporting defects in the LLVM compiler’s middle end optimizers using [Alive2](https://alive2.llvm.org/ce/), a formal methods tool that we created. This piece is about extending that work to cover one of LLVM’s backends.

Why is it useful to try to prove that an LLVM backend did the right thing? It turns out that (despite the name) LLVM IR isn’t all that low-level — the backends need to do a whole lot of work in order to create efficient object code. In fact, there’s quite a bit of code in the backends, such as peephole optimizations that clean up local inefficiencies, that ends up duplicating analogous code in the LLVM middle end optimizers. And where there’s a lot of code, there’s a lot of potential for bugs.

The basic job performed by Alive2 is to prove that a function in LLVM IR refines another one, or else to provide a counterexample showing a violation of refinement. To understand “refinement” we need to know that due to undefined behaviors, an LLVM function can effectively act non-deterministically. For example, a function that returns a value that it loaded from uninitialized storage could return “1” one time we execute it and “2” the next time. An LLVM optimization pass is free to turn this function into one that always returns “1”. The refinement relation, then, holds when — for every possible circumstance (values of arguments, values stored in memory, etc.) in which that function could end up being called — the optimized function exhibits a subset of the behaviors of the original function. Non-trivial refinement, where the behaviors after optimization are a proper subset of the original behaviors, is ubiquitous in practice. If the optimized code contains any behavior not in the unoptimized code, this is a violation of refinement.

So how can we take a refinement checker for LLVM IR and use it to validate the correctness of an LLVM backend? We accomplish this by lifting the compiled AArch64 code back into Alive2 IR, which is very similar to LLVM IR. Then we check that the lifted IR refines the original code. A number of such binary lifters already exist and the techniques they employ are fairly well-understood: it’s basically a matter of converting each ARM instruction into one or more Alive2 instructions that implement that instruction, and then putting the resulting code into SSA form. We tried out some of the existing lifters, and for a few reasons chose not to base our project on any of them. First, fuzzing thrives on throughput, but forking and execing an external tool isn’t fast. Second, we ran into a lot of bugs in some of the lifters we tried — these bugs impede our work by hiding the LLVM bugs we’d have hoped to find. And third, we wanted control over the lifted code so we could tailor it to our needs.

So we wrote a new lifter, basically just by reading about the AArch64 ISA and ABI and implementing them. It’s very much a work in progress and also it’s not really general-purpose: it only needs to deal with the specific case of lifting AArch64 that was just generated from some LLVM IR that we have available to us. This allowed us to take some nice little shortcuts: we don’t have to worry about any instruction that the LLVM backend doesn’t emit, we get the types of function arguments and return values for free, and we didn’t need to write a parser for AArch64 assembly language because LLVM already has one.

Let’s work through a quick example. Here’s an LLVM function:

```
define i32 @f(i32) {
  %c = icmp ult i32 %0, 777
  br i1 %c, label %t, label %f
t:
  %a = add i32 %0, 1
  ret i32 %a
f:
  ret i32 %0
}
```

If f()’s argument is unsigned-smaller-than 777, the argument is incremented and returned, otherwise the argument is returned unmodified. The AArch64 backend translates this into:

```
f:
        cmp     w0, #777
        cinc    w0, w0, lo
        ret
```

[Here’s this example in Compiler Explorer](https://gcc.godbolt.org/z/qf979TrEr). Do these three instructions faithfully implement the semantics of the LLVM code? It’s fairly clear that they do, but we want an automated proof. Here’s the Alive2 IR that we get from lifting the ARM code:

```
define i32 @f-tgt(i32 X0) {
f:
  X0_1 = freeze i32 X0
  X0_2 = zext i32 X0_1 to i64
  WZR_2x1 = trunc i64 X0_2 to i32
  WZR_2x5 = icmp uge i32 WZR_2x1, 777
  X0_3x1 = trunc i64 X0_2 to i32
  X0_3x2 = trunc i64 X0_2 to i32
  X0_3x3 = add i32 X0_3x2, 1
  X0_3x4 = select i1 WZR_2x5, i32 X0_3x1, i32 X0_3x3
  ret i32 X0_3x4
}
```

Does this lifted Alive2 IR refine the original LLVM IR? It does. Alive2 asks Z3 a series of questions and gets back answers that it likes and eventually it prints:

```
Transformation seems to be correct!
```

[Here’s this example online.](https://alive2.llvm.org/ce/z/XCpfBu)

The lifted code isn’t very pretty, what’s going on? First, the “freeze” instruction — which tames LLVM’s poison and undef values — is required because those concepts do not exist at the AArch64 level. Second, there’s a good bit of extending and truncating going on, this happens because this platform has 64-bit registers but the instructions are only manipulating the bottom 32 bits of these registers. Third, there’s a bit of unnecessary code; we wrote a rudimentary optimizer for Alive2 IR, but this isn’t a place where we want to put much complexity, so we don’t try very hard to clean up the mess left behind by our lifter. The intent of our optimizer is simply to increase readability by removing as many irrelevant instructions as possible. For example, AArch64 assembly sometimes has to build constants using multiple instructions. This LLVM IR:

```
define i64 @f() {
  ret i64 4183428261488279476
}
```

turns into this AArch64:

```
f:
	mov	x0, #49076
	movk	x0, #52566, lsl #16
	movk	x0, #34262, lsl #32
	movk	x0, #14862, lsl #48
	ret
```

which we lift to this Alive2 IR:

```
define i64 @f-tgt() {
f:
  %X0_2x1x0 = add i64 49076, 0
  %X0_3x1x0 = shl i64 52566, 16
  %X0_3x2x0 = and i64 %X0_2x1x0, -4294901761
  %X0_3x3x0 = or i64 %X0_3x2x0, %X0_3x1x0
  %X0_4x1x0 = shl i64 34262, 32
  %X0_4x2x0 = and i64 %X0_3x3x0, -281470681743361
  %X0_4x3x0 = or i64 %X0_4x2x0, %X0_4x1x0
  %X0_5x1x0 = shl i64 14862, 48
  %X0_5x2x0 = and i64 %X0_4x3x0, 281474976710655
  %X0_5x3x0 = or i64 %X0_5x2x0, %X0_5x1x0
  ret i64 %X0_5x3x0
}
```

This is no fun to read. However, after some straightforward constant folding, we get this:

```
define i64 @f-tgt() {
f:
  ret i64 4183428261488279476
}
```

If we had lifted AArch64 to LLVM IR, instead of Alive2 IR, we could have gotten a lot of optimizations for free, but we didn’t do that since we try to avoid trusting LLVM in a tool that is designed to find bugs in LLVM.

So far we only have half of a fuzzing loop; if we’re going to look for bugs, we need to push a large number of test cases through the compile-decompile-verify cycle. We do this in three ways. First, using the LLVM unit test suite, which contains about 270,000 functions in LLVM IR. Second, by fuzzing with [the mutation engine we recently posted about](https://blog.regehr.org/archives/2148), using the LLVM unit tests as seeds. Third, using [opt-fuzz](https://github.com/regehr/opt-fuzz), a standalone LLVM generator that John wrote. So far the mutation approach is performing best in terms of finding bugs, probably because opt-fuzz generates a lot of stuff that’s just too simple to trigger bugs, but also because we’ve been leaning harder on the mutator. We’ve already seen several cases where the mutator and opt-fuzz can both find the same bug, which makes us feel warm and fuzzy.

So what do the bugs we find look like? Last week [we reported that the AArch64 backend miscompiles this code](https://github.com/llvm/llvm-project/issues/55833):

```
define i64 @f(i64) {
  %2 = lshr i64 %0, 32
  %3 = shl i64 %2, 16
  %4 = trunc i64 %3 to i32
  %5 = ashr exact i32 %4, 16
  %6 = zext i32 %5 to i64
  ret i64 %6
}
```

by translating it to this:

```
_f:
	sbfx	x0, x0, #32, #16
	ret
```

The bad behavior is seen both in the most recent release (LLVM 14) and also in the current sources.

Here are the bugs we’ve found so far (we’re not looking for crashes — these are all miscompilations):

- [https://github.com/llvm/llvm-project/issues/55003](https://github.com/llvm/llvm-project/issues/55003
  )
- [https://github.com/llvm/llvm-project/issues/55129](https://github.com/llvm/llvm-project/issues/55129
  )
- [https://github.com/llvm/llvm-project/issues/55150](https://github.com/llvm/llvm-project/issues/55150
  )
- [https://github.com/llvm/llvm-project/issues/55178](https://github.com/llvm/llvm-project/issues/55178
  )
- [https://github.com/llvm/llvm-project/issues/55201](https://github.com/llvm/llvm-project/issues/55201
  )
- [https://github.com/llvm/llvm-project/issues/55271](https://github.com/llvm/llvm-project/issues/55271
  )
- [https://github.com/llvm/llvm-project/issues/55284](https://github.com/llvm/llvm-project/issues/55284
  )
- [https://github.com/llvm/llvm-project/issues/55287](https://github.com/llvm/llvm-project/issues/55287
  )
- [https://github.com/llvm/llvm-project/issues/55296](https://github.com/llvm/llvm-project/issues/55296
  )
- [https://github.com/llvm/llvm-project/issues/55342](https://github.com/llvm/llvm-project/issues/55342
  )
- [https://github.com/llvm/llvm-project/issues/55484](https://github.com/llvm/llvm-project/issues/55484
  )
- [https://github.com/llvm/llvm-project/issues/55490](https://github.com/llvm/llvm-project/issues/55490
  )
- [https://github.com/llvm/llvm-project/issues/55627](https://github.com/llvm/llvm-project/issues/55627
  )
- [https://github.com/llvm/llvm-project/issues/55644](https://github.com/llvm/llvm-project/issues/55644)
- [https://github.com/llvm/llvm-project/issues/55833](https://github.com/llvm/llvm-project/issues/55833)

It’s great to see that a number of these have been fixed already! Note that some of the bugs only happen when [global instruction selection](https://www.youtube.com/watch?v=S6SNs2ttdoA) is enabled — this is not yet enabled by default.

What have we learned? Perhaps surprisingly, a number of the bugs we found are not ARM-specific, but rather are in shared, non-target-specific LLVM backend code, which means that some of these bugs end up affecting multiple backends. Something else we learned (predictable, in retrospect) is that testing a backend is in an important sense more difficult than testing the middle end. Whereas most middle-end optimizations don’t care about the bitwidth of the instructions involved, the backends most certainly do care about this: instruction selection will be different at 8 vs 16 vs 32 vs 64 bits, different still at 19 bits, and even more different at 91 bits. So we have to run a lot more test cases through a backend than the middle end to get roughly equivalent coverage, and also we’re going to need to implement mutations that specifically change bitwidths in a test case. Does anyone actually care if the LLVM backends can deal with non-power-of-2 bitwidths? Turns out yes: in an optimized compile of LLVM itself, using LLVM, every integer width from 1 through 64 can be found. The largest integer that occurs when compiling LLVM using LLVM is 320 bits wide.

One might ask: Why are we doing bugfinding work? Why not focus all of our effort on developing our tooling to the point where it can prove that application codes of interest are being compiled correctly? There are two answers. The first is personal: bugfinding is a rewarding activity that lets us engage with the LLVM community in ways that we enjoy. In contrast, true formal verification work can feel isolated and also the endpoint is very difficult to reach: real codes invariably contain features that are hostile to formal methods. The final result, if reached, tends to be a bit of a letdown (“wow, it’s correct”). The second reason that we like to do bugfinding work is that it benefits all LLVM users, as opposed to a pure verification effort that benefits only the small subset of users who are willing to put formal methods into their workflow. But, of course, in the long run, translation validation of real codes is something we’re pursuing.

The AArch64 lifter is part of our larger program of working towards developer tools that deserve to be trusted. There’s plenty of work left to do on our lifter, and also some follow-on projects suggest themselves. First, it would be nice to derive our lifter from an existing formal semantics for AArch64 rather than writing it afresh from the documentation. Second, we’d like to support additional architectures. For example, we think that RISC-V will be pretty easy to support given the infrastructure we now have in place.

\[Our ARM lifter is not yet available on the [Alive2 Compiler Explorer instance](https://alive2.llvm.org/ce/).\]

\[This work is sponsored, in part, by [Woven Planet](https://www.woven-planet.global/en)\]
