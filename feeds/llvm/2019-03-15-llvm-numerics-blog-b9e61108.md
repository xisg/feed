---
title: LLVM Numerics Blog
url: https://blog.llvm.org/2019/03/llvm-numerics-blog.html
published: "2019-03-15T15:47:00Z"
feed: llvm
guid: https://blog.llvm.org/2019/03/llvm-numerics-blog.html
---

# LLVM Numerics Blog

Keywords: Numerics, Clang, LLVM-IR, : 2019 LLVM Developers' Meeting, LLVMDevMtg.

The goal of this blog post is to start a discussion about numerics in LLVM – where we are, recent work and things that remain to be done.  There will be an informal discussion on numerics at the 2019 EuroLLVM conference next month. One purpose of this blog post is to refresh everyone's memory on where we are on the topic of numerics to restart the discussion.

In the last year or two there has been a push to allow fine-grained decisions on which optimizations are legitimate for any given piece of IR.  In earlier days there were two main modes of operation: fast-math and precise-math.  When operating under the rules of precise-math, defined by IEEE-754, a significant number of potential optimizations on sequences of arithmetic instructions are not allowed because they could lead to violations of the standard.

For example:

The Reassociation optimization pass is generally not allowed under precise code generation as it can change the order of operations altering the creation of NaN and Inf values propagated at the expression level as well as altering precision.

Precise code generation is often overly restrictive, so an alternative fast-math mode is commonly used where all possible optimizations are allowed, acknowledging that this impacts the precision of results and possibly IEEE compliant behavior as well.  In LLVM, this can be enabled by setting the unsafe-math flag at the module level, or passing the -funsafe-math-optimizations to clang which then sets flags on the IR it generates.  Within this context the compiler often generates shorter sequences of instructions to compute results, and depending on the context this may be acceptable.  Fast-math is often used in computations where loss of precision is acceptable.  For example when computing the color of a pixel, even relatively low precision is likely to far exceed the perception abilities of the eye, making shorter instruction sequences an attractive trade-off.  In long-running simulations of physical events however loss of precision can mean that the simulation drifts from reality making the trade-off unacceptable.

Several years ago LLVM IR instructions gained the ability of being annotated with flags that can drive optimizations with more granularity than an all-or-nothing decision at the module level.The IR flags in question are:

**nnan, ninf, nsz, arcp, contract, afn, reassoc, nsw, nuw, exact**.

Their exact meaning is described in the [LLVM Language Reference Manual](https://llvm.org/docs/LangRef.html#fast-math-flags%22%3ELLVM%20Language%20Reference%20Manual%3C/a%3E).   When all the flags are are **enabled**, we get the current fast-math behavior.  When these flags are **disabled**, we get precise math behavior.  There are also several options available between these two models that may be attractive to some applications.  In the past year several members of the LLVM community worked on making IR optimizations passes aware of these flags.  When the unsafe-math module flag is not set these optimization passes will work by examining individual flags, allowing fine-grained selection of the optimizations that can be enabled on specific instruction sequences.  This allows vendors/implementors to mix fast and precise computations in the same module, aggressively optimizing some instruction sequences but not others.

We now have good coverage of IR passes in the LLVM codebase, in particular in the following areas:

\\* Intrinsic and libcall management

\\* Instruction Combining and Simplification

\\* Instruction definition

\\* SDNode definition

\\* GlobalIsel Combining and code generation

\\* Selection DAG code generation

\\* DAG Combining

\\* Machine Instruction definition

\\* IR Builders (SDNode, Instruction, MachineInstr)

\\* CSE tracking

\\* Reassociation

\\* Bitcode

There are still some areas that need to be reworked for modularity, including vendor specific back-end passes.

The following are some of the contributions mentioned above from the last 2 years of open source development:

[https://reviews.llvm.org/D45781](https://reviews.llvm.org/D45781) : MachineInst support mapping SDNode fast math flags for support in Back End code generation

[https://reviews.llvm.org/D46322](https://reviews.llvm.org/D46322) : \[SelectionDAG\] propagate 'afn' and 'reassoc' from IR fast-math-flags

[https://reviews.llvm.org/D45710](https://reviews.llvm.org/D45710) : Fast Math Flag mapping into SDNode

[https://reviews.llvm.org/D46854](https://reviews.llvm.org/D46854) : \[DAG\] propagate FMF for all FPMathOperators

[https://reviews.llvm.org/D48180](https://reviews.llvm.org/D48180) : updating isNegatibleForFree and GetNegatedExpression with fmf for fadd

[https://reviews.llvm.org/D48057:](https://reviews.llvm.org/D48057:) easing the constraint for isNegatibleForFree and GetNegatedExpression

[https://reviews.llvm.org/D47954](https://reviews.llvm.org/D47954) : Utilize new SDNode flag functionality to expand current support for fdiv

[https://reviews.llvm.org/D47918](https://reviews.llvm.org/D47918) : Utilize new SDNode flag functionality to expand current support for fma

[https://reviews.llvm.org/D47909](https://reviews.llvm.org/D47909) : Utilize new SDNode flag functionality to expand current support for fadd

[https://reviews.llvm.org/D47910](https://reviews.llvm.org/D47910) : Utilize new SDNode flag functionality to expand current support for fsub

[https://reviews.llvm.org/D47911](https://reviews.llvm.org/D47911) : Utilize new SDNode flag functionality to expand current support for fmul

[https://reviews.llvm.org/D48289](https://reviews.llvm.org/D48289) : refactor of visitFADD for AllowNewConst cases

[https://reviews.llvm.org/D47388](https://reviews.llvm.org/D47388) : propagate fast math flags via IR on fma and sub expressions

[https://reviews.llvm.org/D47389](https://reviews.llvm.org/D47389) : guard fneg with fmf sub flags

[https://reviews.llvm.org/D47026](https://reviews.llvm.org/D47026) : fold FP binops with undef operands to NaN

[https://reviews.llvm.org/D47749](https://reviews.llvm.org/D47749) : guard fsqrt with fmf sub flags

[https://reviews.llvm.org/D46447](https://reviews.llvm.org/D46447) : Mapping SDNode flags to MachineInstr flags

[rL334970: \[NFC\] make MIFlag accessor functions consistant with usage model](https://reviews.llvm.org/rL334970)

[rL338604: \[NFC\] small addendum to r334242, FMF propagation](https://reviews.llvm.org/rL338604)

[https://reviews.llvm.org/D50195](https://reviews.llvm.org/D50195) : extend folding fsub/fadd to fneg for FMF

[https://reviews.llvm.org/rL339197](https://reviews.llvm.org/rL339197) : \[NFC\] adding tests for Y - (X + Y) --> -X

[https://reviews.llvm.org/D50417](https://reviews.llvm.org/D50417) : \[InstCombine\] fold fneg into constant operand of fmul/fdiv

[https://reviews.llvm.org/rL339357](https://reviews.llvm.org/rL339357) : extend folding fsub/fadd to fneg for FMF

[https://reviews.llvm.org/D50996](https://reviews.llvm.org/D50996) : extend binop folds for selects to include true and false binops flag intersection

[https://reviews.llvm.org/rL339938](https://reviews.llvm.org/rL339938) : add a missed case for binary op FMF propagation under select folds

[https://reviews.llvm.org/D51145](https://reviews.llvm.org/D51145) : Guard FMF context by excluding some FP operators from FPMathOperator

[https://reviews.llvm.org/rL341138](https://reviews.llvm.org/rL341138) : adding initial intersect test for Node to Instruction association

[https://reviews.llvm.org/rL341565](https://reviews.llvm.org/rL341565) : in preparation for adding nsw, nuw and exact as flags to MI

[https://reviews.llvm.org/D51738](https://reviews.llvm.org/D51738) : add IR flags to MI

[https://reviews.llvm.org/D52006](https://reviews.llvm.org/D52006) : Copy utilities updated and added for MI flags

[https://reviews.llvm.org/rL342598](https://reviews.llvm.org/rL342598) : add new flags to a DebugInfo lit test

[https://reviews.llvm.org/D53874](https://reviews.llvm.org/D53874) : \[InstSimplify\] fold 'fcmp nnan oge X, 0.0' when X is not negative

[https://reviews.llvm.org/D55668](https://reviews.llvm.org/D55668) : Add FMF management to common fp intrinsics in GlobalIsel

[https://reviews.llvm.org/rL352396](https://reviews.llvm.org/rL352396) : \[NFC\] TLI query with default(on) behavior wrt DAG combines for fmin/fmax target…

[https://reviews.llvm.org/rL316753](https://reviews.llvm.org/rL316753) (Fold fma (fneg x), K, y -> fma x, -K, y)

[https://reviews.llvm.org/D57630](https://reviews.llvm.org/D57630) : Move IR flag handling directly into builder calls for cases translated from Instructions in GlobalIsel

[https://reviews.llvm.org/rL332756](https://reviews.llvm.org/rL332756) : adding baseline fp fold tests for unsafe on and off

[https://reviews.llvm.org/rL334035](https://reviews.llvm.org/rL334035) : NFC: adding baseline fneg case for fmf

[https://reviews.llvm.org/rL325832](https://reviews.llvm.org/rL325832) : \[InstrTypes\] add frem and fneg with FMF creators

[https://reviews.llvm.org/D41342](https://reviews.llvm.org/D41342) : \[InstCombine\] Missed optimization in math expression: simplify calls exp functions

[https://reviews.llvm.org/D52087](https://reviews.llvm.org/D52087) : \[IRBuilder\] Fixup CreateIntrinsic to allow specifying Types to Mangle.

[https://reviews.llvm.org/D52075](https://reviews.llvm.org/D52075) : \[InstCombine\] Support (sub (sext x), (sext y)) --> (sext (sub x, y)) and (sub (zext x), (zext y)) --> (zext (sub x, y))

[https://reviews.llvm.org/rL338059](https://reviews.llvm.org/rL338059) : \[InstCombine\] fold udiv with common factor from muls with nuw

Commit: e0ab896a84be9e7beb59874b30f3ac51ba14d025 : \[InstCombine\] allow more fmul folds with ‘reassoc'

Commit: 3e5c120fbac7bdd4b0ff0a3252344ce66d5633f9 : \[InstCombine\] distribute fmul over fadd/fsub

[https://reviews.llvm.org/D37427](https://reviews.llvm.org/D37427) : \[InstCombine\] canonicalize fcmp ord/uno with constants to null constant

[https://reviews.llvm.org/D40130](https://reviews.llvm.org/D40130) : \[InstSimplify\] fold and/or of fcmp ord/uno when operand is known nnan

[https://reviews.llvm.org/D40150](https://reviews.llvm.org/D40150) : \[LibCallSimplifier\] fix pow(x, 0.5) -> sqrt() transforms

[https://reviews.llvm.org/D39642](https://reviews.llvm.org/D39642) : \[ValueTracking\] readnone is a requirement for converting sqrt to llvm.sqrt; nnan is not

[https://reviews.llvm.org/D39304](https://reviews.llvm.org/D39304) : \[IR\] redefine 'reassoc' fast-math-flag and add 'trans' fast-math-flag

[https://reviews.llvm.org/D41333](https://reviews.llvm.org/D41333) : \[ValueTracking\] ignore FP signed-zero when detecting a casted-to-integer fmin/fmax pattern

[https://reviews.llvm.org/D5584](https://reviews.llvm.org/D5584) : Optimize square root squared (PR21126)

[https://reviews.llvm.org/D42385](https://reviews.llvm.org/D42385) : \[InstSimplify\] (X \* Y) / Y --> X for relaxed floating-point ops

[https://reviews.llvm.org/D43160](https://reviews.llvm.org/D43160) : \[InstSimplify\] allow exp/log simplifications with only 'reassoc’ FMF

[https://reviews.llvm.org/D43398](https://reviews.llvm.org/D43398) : \[InstCombine\] allow fdiv folds with less than fully 'fast’ ops

[https://reviews.llvm.org/D44308](https://reviews.llvm.org/D44308) : \[ConstantFold\] fp\_binop AnyConstant, undef --> NaN

[https://reviews.llvm.org/D43765](https://reviews.llvm.org/D43765) : \[InstSimplify\] loosen FMF for sqrt(X) \* sqrt(X) --> X

[https://reviews.llvm.org/D44521](https://reviews.llvm.org/D44521) : \[InstSimplify\] fp\_binop X, NaN --> NaN

[https://reviews.llvm.org/D47202](https://reviews.llvm.org/D47202) : \[CodeGen\] use nsw negation for abs

[https://reviews.llvm.org/D48085](https://reviews.llvm.org/D48085) : \[DAGCombiner\] restrict (float)((int) f) --> ftrunc with no-signed-zeros

[https://reviews.llvm.org/D48401](https://reviews.llvm.org/D48401) : \[InstCombine\] fold vector select of binops with constant ops to 1 binop (PR37806)

[https://reviews.llvm.org/D39669](https://reviews.llvm.org/D39669) : DAG: Preserve nuw when reassociating adds

[https://reviews.llvm.org/D39417](https://reviews.llvm.org/D39417) : InstCombine: Preserve nuw when reassociating nuw ops

[https://reviews.llvm.org/D51753](https://reviews.llvm.org/D51753) : \[DAGCombiner\] try to convert pow(x, 1/3) to cbrt(x)

[https://reviews.llvm.org/D51630](https://reviews.llvm.org/D51630) : \[DAGCombiner\] try to convert pow(x, 0.25) to sqrt(sqrt(x))

[https://reviews.llvm.org/D53650](https://reviews.llvm.org/D53650) : \[FPEnv\] Last BinaryOperator::isFNeg(...) to m\_FNeg(...) changes

[https://reviews.llvm.org/D54001](https://reviews.llvm.org/D54001): \[ValueTracking\] determine sign of 0.0 from select when matching min/max FP

[https://reviews.llvm.org/D51942](https://reviews.llvm.org/D51942): \[InstCombine\] Fold (C/x)>0 into x>0 if possible

[https://llvm.org/svn/llvm-project/llvm/trunk@348016](https://llvm.org/svn/llvm-project/llvm/trunk@348016) : \[SelectionDAG\] fold FP binops with 2 undef operands to undef

[http://llvm.org/viewvc/llvm-project?view=revision&revision=346242](http://llvm.org/viewvc/llvm-project?view=revision&revision=346242) : propagate fast-math-flags when folding fcmp+fpext, part 2

[http://llvm.org/viewvc/llvm-project?view=revision&revision=346240](http://llvm.org/viewvc/llvm-project?view=revision&revision=346240): propagate fast-math-flags when folding fcmp+fpext

[http://llvm.org/viewvc/llvm-project?view=revision&revision=346238](http://llvm.org/viewvc/llvm-project?view=revision&revision=346238) : \[InstCombine\] propagate fast-math-flags when folding fcmp+fneg, part 2

[http://llvm.org/viewvc/llvm-project?view=revision&revision=346169](http://llvm.org/viewvc/llvm-project?view=revision&revision=346169): \[InstSimplify\] fold select (fcmp X, Y), X, Y

[http://llvm.org/viewvc/llvm-project?view=revision&revision=346234](http://llvm.org/viewvc/llvm-project?view=revision&revision=346234): propagate fast-math-flags when folding fcmp+fneg

[http://llvm.org/viewvc/llvm-project?view=revision&revision=346147](http://llvm.org/viewvc/llvm-project?view=revision&revision=346147): \[InstCombine\] canonicalize -0.0 to +0.0 in fcmp

[http://llvm.org/viewvc/llvm-project?view=revision&revision=346143](http://llvm.org/viewvc/llvm-project?view=revision&revision=346143): \[InstCombine\] loosen FP 0.0 constraint for fcmp+select substitution

[http://llvm.org/viewvc/llvm-project?view=revision&revision=345734](http://llvm.org/viewvc/llvm-project?view=revision&revision=345734) : \[InstCombine\] refactor fabs+fcmp fold; NFC

[http://llvm.org/viewvc/llvm-project?view=revision&revision=345728](http://llvm.org/viewvc/llvm-project?view=revision&revision=345728): \[InstSimplify\] fold 'fcmp nnan ult X, 0.0' when X is not negative

[http://llvm.org/viewvc/llvm-project?view=revision&revision=345727](http://llvm.org/viewvc/llvm-project?view=revision&revision=345727): \[InstCombine\] add assertion that InstSimplify has folded a fabs+fcmp; NFC

While multiple people have been working on finer-grained control over fast-math optimizations and other relaxed numerics modes, there has also been some initial progress on adding support for _more_ constrained numerics models. There has been considerable progress towards adding and enabling constrained floating-point intrinsics to capture FENV\_ACCESS ON and similar semantic models.

These experimental constrained intrinsics prohibit certain transforms that are not safe if the default floating-point environment is not in effect. Historically, LLVM has in practice basically “split the difference” with regard to such transforms; they haven’t been explicitly disallowed, as LLVM doesn’t model the floating-point environment, but they have been disabled when they caused trouble for tests or software projects. The absence of a formal model for licensing these transforms constrains our ability to enable them. Bringing language and backend support for constrained intrinsics across the finish line will allow us to include transforms that we disable as a matter of practicality today, and allow us to give developers an easy escape valve (in the form of FENV\_ACCESS ON and similar language controls) when they need more precise control, rather than an ad-hoc set of flags to pass to the driver.

We should discuss these new intrinsics to make sure that they can capture the right models for all the languages that LLVM supports.

Here are some possible discussion items:

- Should specialization be applied at the call level for edges in a call graph where the caller has special context to extend into the callee wrt to flags?
- Should the inliner apply something similar to calls that meet inlining criteria?
- What other part(s) of the compiler could make use of IR flags that are currently not covered?
- What work needs to be done regarding code debt wrt current areas of implementation.
