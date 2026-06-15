---
title: 'Alive2 Part 3: Things You Can and Can’t Do with Undef in LLVM'
url: https://blog.regehr.org/archives/1837
published: "2020-07-31T20:33:05Z"
feed: regehr
guid: https://blog.regehr.org/?p=1837
---

# Alive2 Part 3: Things You Can and Can’t Do with Undef in LLVM

\[Also see [Part 1](https://blog.regehr.org/archives/1722) and [Part 2](https://blog.regehr.org/archives/1737) in this series.\]

Let’s talk about these functions:

```
unsigned add(unsigned x) { return x + x; }
unsigned shift(unsigned x) { return x << 1; }
```

From the point of view of the C and C++ abstract machines, their behavior is equivalent: in a program you’re writing, you can always safely substitute add for shift, or shift for add.

If we compile them to LLVM IR with optimizations enabled, [LLVM will canonicalize both functions to the version containing the shift operator](https://gcc.godbolt.org/z/bGurwr). Although it might seem that this choice is an arbitrary one, it turns out there is a very real difference between these two functions:

```
define i32 @add(i32 %0) {
  %2 = add i32 %0, %0
  ret i32 %2
}
define i32 @shift(i32 %0) {
  %2 = shl i32 %0, 1
  ret i32 %2
}
```

At the LLVM level, [add can always be replaced by shift](https://alive2.llvm.org/ce/z/pje5Q7), but [it is not generally safe to replace shift with add](https://alive2.llvm.org/ce/z/iR2mfT). (In these linked pages, I’ve renamed the functions to src and tgt so that Alive2 knows which way the transformation is supposed to go.)

To understand why add can be replaced by shift, but shift cannot be replaced by add, we need to know that these functions don’t have 232 possible input values, but rather 232+2\. Analogously, an LLVM function taking an i1 (Boolean) argument has four possible input values, not the two you were probably expecting. These extra two values, undef and poison, cause a disproportionate amount of trouble because they follow different rules than regular values follow. This post won’t discuss [poison](https://llvm.org/docs/LangRef.html#poison-values) further. [Undef](https://llvm.org/docs/LangRef.html#undefined-values) sort of models uninitialized storage: it can resolve to any value of its type. Unless we can specifically prove that an LLVM value, such as the input to a function, cannot be undef, we must assume that it might be undef. Many compiler bugs have been introduced because compiler developers either forgot about this possibility, or else failed to reason about it correctly.

Let’s look at some things you’re allowed to do with undef. You can [leave it alone](https://alive2.llvm.org/ce/z/psNicd):

![](https://blog.regehr.org/wp-content/uploads/2020/07/Screen-Shot-2020-07-20-at-10.14.44-AM.png)

You can [replace it with zero](https://alive2.llvm.org/ce/z/DhPanX):

![](https://blog.regehr.org/wp-content/uploads/2020/07/Screen-Shot-2020-07-20-at-10.20.13-AM-1.png)

You can [replace it with any number you choose](https://alive2.llvm.org/ce/z/ECRrAS):

![](https://blog.regehr.org/wp-content/uploads/2020/07/Screen-Shot-2020-07-20-at-10.20.22-AM.png)

You can [replace it with the set of odd numbers](https://alive2.llvm.org/ce/z/3h6bFj):

![](https://blog.regehr.org/wp-content/uploads/2020/07/Screen-Shot-2020-07-20-at-10.20.30-AM.png)

You can [take a function that returns \[0..15\] and replace it by a function that returns \[3..10\]](https://alive2.llvm.org/ce/z/BK6JPt):

![](https://blog.regehr.org/wp-content/uploads/2020/07/Screen-Shot-2020-07-20-at-10.20.38-AM.png)

The common pattern here is that the new function returns a subset of the values returned by the old function. This is the essence of _refinement_, which is what every step of a correct compilation pipeline must do. (Though keep in mind that right now we’re looking at a particularly simple special case: a pure function that takes no arguments.) So what can’t you do with undef? This is easy: you can’t violate refinement. In other words, you can’t replace a (pure, zero-argument) function with a function that fails to return a subset of the values that is returned by the original one.

So, for example, you can’t [take a function that returns \[0..15\] and replace it by a function that returns \[9..16\]](https://alive2.llvm.org/ce/z/8VRq8E):

![](https://blog.regehr.org/wp-content/uploads/2020/07/Screen-Shot-2020-07-20-at-10.24.04-AM.png)

As we can see in the counterexample, Alive2 has correctly identified 16 as the problematic output. Returning \[9..15\] would have been fine, as would any other subset of \[0..15\].

When a function takes inputs, the refinement rule is: for every valuation of the inputs, the new function must return a subset of the values returned by the old function. Refinement gets more complicated for functions that touch memory; my colleagues Juneyoung and Nuno will cover that in a different post.

Now let’s return to the question from the top of this piece: why can we turn add into shift, but not shift into add? This is because every time an undef is used, the use can result in a different value ( [here “use” has a precise meaning from SSA](https://en.wikipedia.org/wiki/Static_single_assignment_form)). If undef is passed to the shift function, an arbitrary value is left-shifted by one position, resulting in an arbitrary even value being returned. On the other hand, the add function uses the undef value twice. When two different arbitrary values are added together, the result is an arbitrary value. This is a clear failure of refinement because the set of all 32-bit values fails to be a subset of the set of the even numbers in 0..232-1\. Why was undef designed this way? It is to give the compiler maximum freedom when translating undef. If LLVM mandated that every use of an undef returned the same value, then it would sometimes be forced to remember that value, in order to follow this rule consistently. The resulting value would sometimes occupy a register, potentially preventing something more useful from being done with that register. As of LLVM 10, this “remembering” behavior can be achieved using the freeze instruction. A frozen undef (or poison) value is arbitrary, but consistent. So if we want to transform the shift function into the add function in a way that respects refinement, [we can do it like this](https://alive2.llvm.org/ce/z/jGoHFy):

![](https://blog.regehr.org/wp-content/uploads/2020/07/Screen-Shot-2020-07-20-at-10.29.44-AM.png)

A general guideline is that an LLVM transformation can decrease the number of uses of a possibly-undef value, or leave the number of uses the same, but it cannot increase the number of uses unless the value is frozen first.

Instead of adding freeze instructions everywhere, one might ask whether it is possible to avoid undef-related problems by proving that a value cannot possibly be undef. This can be done, and as of recently [Juneyoung Lee has added a static analysis for this](https://github.com/llvm/llvm-project/blob/65c63eb69cc157b5bf257d6b91ecd758446ee5a1/llvm/lib/Analysis/ValueTracking.cpp#L4775-L4871). Alas, it is necessarily pretty conservative, it only reaches the desired conclusion under restricted circumstances.

Before concluding, it’s worth recalling that undefined behavior in an IR is not necessarily closely related to the source-level UB. In other words, we can compile a safe language to LLVM IR, and we can also compile C and C++ to IRs lacking undefined behavior. In light of this, we might ask whether undef, and its friend, poison, are worthwhile concepts in a compiler IR at all, given the headaches they cause for compiler developers. On one hand, we can find IR-like languages such as Java bytecode that lack analogous abstractions. On the other hand, as far as actual IRs for heavily optimizing compilers go, I believe all of the major ones have something more or less resembling poison and/or undef. The fact that a number of different compiler teams invented similar concepts suggests that there might be something fundamental going on here. An open question is to what extent these abstractions are worthwhile in an IR for a heavily-optimizing compiler middle-end that is only targeted by safe languages.

My colleagues and I believe that having both undef and poison in LLVM is overkill, and that undef can and should be removed at some point. This was not true in the past, but we believe that the new freeze instruction, in combination with poison values, provides adequate support for undef’s use cases.
