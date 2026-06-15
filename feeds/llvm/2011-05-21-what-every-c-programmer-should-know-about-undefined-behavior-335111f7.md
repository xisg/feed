---
title: 'What Every C Programmer Should Know About Undefined Behavior #3/3'
url: https://blog.llvm.org/2011/05/what-every-c-programmer-should-know_21.html
published: "2011-05-21T00:48:00Z"
feed: llvm
guid: https://blog.llvm.org/2011/05/what-every-c-programmer-should-know_21.html
---

# What Every C Programmer Should Know About Undefined Behavior #3/3

In [Part 1](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html) of the series, we took a look at undefined behavior in C and showed some cases where it allows C to be more performant than "safe" languages. In [Part 2](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know_14.html), we looked at the surprising bugs this causes and some widely held misconceptions that many programmers have about C. In this article, we look at the challenges that compilers face in providing warnings about these gotchas, and talk about some of the features and tools that LLVM and Clang provide to help get the performance wins, while taking away some of the surprise.

 Translation available in: [Japanese](http://blog-ja.intransient.info/2011/06/c-33.html)

## Why can't you warn when optimizing based on undefined behavior?

People often ask why the compiler doesn't produce warnings when it is taking advantage of undefined behavior to do an optimization, since any such case might actually be a bug in the user code. The challenges with this approach are that it is 1) likely to generate far too many warnings to be useful - because these optimizations kick in all the time when there is no bug, 2) it is really tricky to generate these warnings only when people want them, and 3) we have no good way to express (to the user) how a series of optimizations combined to expose the opportunity being optimized. Lets take each of these in turn:

**It is "really hard" to make it actually useful**

Lets look at an example: even though invalid type casting bugs are frequently exposed by type based alias analysis, it would not be useful to produce a warning that "the optimizer is assuming that P and P\[i\] don't alias" when optimizing "zero\_array" (from [Part #1 of our series](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html)).

```
float *P;
 void zero_array() {
 int i;
 for (i = 0; i < 10000; ++i)
 P[i] = 0.0f;
}

```

Beyond this "false positive" problem, a logistical problem is that the optimizer doesn't have enough information to generate a reasonable warning. First of all, it is working on an already-abstract representation of the code ( [LLVM IR](http://llvm.org/docs/LangRef.html)) which is quite different from C, and second, the compiler is highly layered to the point where the optimization trying to "hoist a load from P out of the loop" doesn't know that TBAA was the analysis that resolved the pointer alias query. Yes, this is "the compiler guy whining" part of the article :), but it really is a hard problem.

**It is hard to generate these warnings _only_ when people want them**

Clang implements numerous warnings for simple and obvious cases of undefined behavior, such as out of range shifts like "x << 421". You might think that this is a simple and obvious thing, but it turns out that this is hard, because [people don't want to get warnings about undefined behavior in dead code](http://llvm.org/bugs/show_bug.cgi?id=5544) (see also [the duplicates](http://llvm.org/bugs/show_bug.cgi?id=6933)).

This dead code can take several forms: a macro that expands out in a funny way when when passed a constant, we've even had complaints that we warn in cases that would require [control flow analysis](http://llvm.org/bugs/show_bug.cgi?id=9322) of switch statements to prove that cases are not reachable. This is not helped by the fact that switch statements in C are [not necessarily properly structured](http://en.wikipedia.org/wiki/Duff's_device).

The solution to this in Clang is a growing infrastructure for handling "runtime behavior" warnings, along with code to prune these out so that they are not reported if we later find out that the block is unexecutable. This is something of an arms race with programmers though, because there are always idioms that we don't anticipate, and doing this sort of thing in the frontend means that it doesn't catch every case people would want it to catch.

**Explaining a series of optimizations that exposed an opportunity**

If the frontend has challenges producing good warnings, perhaps we can generate them _from the optimizer_ instead! The biggest problem with producing a useful warning here is one of data tracking. A compiler optimizer includes dozens of optimization passes that each change the code as it comes through to canonicalize it or (hopefully) make it run faster. If the inliner decides to inline a function, this may expose other opportunities for [optimizing away an "X\*2/2"](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html#signed_overflow), for example.

While I've given [relatively simple and self-contained examples](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html#optimizations) to demonstrate these optimizations, most of the cases where they kick in are in code coming from macro instantiation, inlining, and other abstraction-elimination activities the compiler performs. The reality is that humans don't commonly write such silly things directly. For warnings, this means that in order to relay back the issue to the users code, the warning would have to reconstruct exactly how the compiler got the intermediate code it is working on. We'd need the ability to say something like:

> `
> warning: after 3 levels of inlining (potentially across files with Link Time Optimization), some common subexpression elimination, after hoisting this thing out of a loop and proving that these 13 pointers don't alias, we found a case where you're doing something undefined. This could either be because there is a bug in your code, or because you have macros and inlining and the invalid code is dynamically unreachable but we can't prove that it is dead.`

Unfortunately, we simply don't have the internal tracking infrastructure to produce this, and even if we did, the compiler doesn't have a user interface good enough to express this to the programmer.

Ultimately, undefined behavior is valuable to the optimizer because it is saying "this operation is invalid - you can assume it never happens". In a case like "\*P" this gives the optimizer the ability to reason that P cannot be NULL. In a case like "\*NULL" (say, after some constant propagation and inlining), this allows the optimizer to know that the code must not be reachable. The important wrinkle here is that, because it cannot solve the halting problem, the compiler cannot know whether code is actually dead (as the C standard says it must be) or whether it is a bug that was exposed after a (potentially long) series of optimizations. Because there isn't a generally good way to distinguish the two, almost all of the warnings produced would be false positives (noise).

## Clang's Approach to Handling Undefined Behavior

Given the sorry state that we're in when it comes to undefined behavior, you might be wondering what Clang and LLVM are doing to try to improve the situation. I mentioned a couple of them already: the [Clang Static Analyzer](http://clang-analyzer.llvm.org/), [Klee project](http://klee.llvm.org/), and the `-fcatch-undefined-behavior` flag are useful tools for tracking down some classes of these bugs. The problem is that these aren't as widely used as the compiler is, so anything we can do directly in the compiler offers even higher goodness than doing it in these other tools. Keep in mind though that the compiler is limited by not having dynamic information and by being limited to what it can without burning lots of compile time.

Clang's first step to improve the world's code is to turn on a whole lot more warnings by default than other compilers do. While some developers are disciplined and build with " `-Wall -Wextra`" (for example), many people don't know about these flags or don't bother to pass them. Turning more warnings on by default catches more bugs more of the time.

The second step is that Clang generates warnings for many classes of undefined behavior (including dereference of null, oversized shifts, etc) that are obvious in the code to catch some common mistakes. Some of the caveats are mentioned above, but these seem to work well in practice.

The third step is that the LLVM optimizer generally takes much less liberty with undefined behavior than it could. Though the standard says that any instance of undefined behavior has completely unbound effects on the program, this is not a particularly useful or developer friendly behavior to take advantage of. Instead, the LLVM optimizer handles these optimizations in a few different ways (the links describe rules of LLVM IR, not C, sorry!):

1. Some cases of undefined behavior are silently transformed into implicitly trapping operations if there is a good way to do that. For example, with Clang, this C++ function:





   ```
   int *foo(long x) {
    return new int[x];
   }

   ```



   compiles into this X86-64 machine code:





   ```
   __Z3fool:
    movl $4, %ecx
    movq %rdi, %rax
    mulq %rcx
    movq $-1, %rdi # Set the size to -1 on overflow
    cmovnoq %rax, %rdi # Which causes 'new' to throw std::bad_alloc
    jmp __Znam

   ```



   instead of the code GCC produces:





   ```
   __Z3fool:
    salq $2, %rdi
    jmp __Znam # Security bug on overflow!

   ```



   The difference here is that we've decided to invest a few cycles in preventing a potentially [serious integer overflow bug](http://cert.uni-stuttgart.de/advisories/calloc.php) that can lead to buffer overflows and exploits (operator new is typically fairly expensive, so the overhead is almost never noticable). The GCC folks have been aware of this [since at least 2005](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=19351) but haven't fixed this at the time of this writing.
2. Arithmetic that [operates on undefined values](http://llvm.org/docs/LangRef.html#undefvalues) is considered to produce a undefined value instead of producing undefined behavior. The distinction is that undefined values can't format your hard drive or produce other undesirable effects. A useful refinement happens in cases where the arithmetic would produce the same output bits given any possible instance of the undefined value. For example, the optimizer assumes that the result of "undef & 1" has zeros for its top bits, treating only the low bit as undefined. This means that ((undef & 1) >> 1) is defined to be 0 in LLVM, not undefined.
3. Arithmetic that dynamically executes an undefined operation (such as a signed integer overflow) generates a logical [trap value](http://llvm.org/docs/LangRef.html#trapvalues) which poisons any computation based on it, but that does not destroy your entire program. This means that logic downstream from the undefined operation may be affected, but that your entire program isn't destroyed. This is why the optimizer ends up deleting code that operates on uninitialized variables, for example.
4. Stores to null and calls through null pointers are turned into a \_\_builtin\_trap() call (which turns into a trapping instruction like "ud2" on x86). These happen all of the time in optimized code (as the result of other transformations like inlining and constant propagation) and we used to just delete the blocks that contained them because they were "obviously unreachable".



   While (from a pedantic language lawyer standpoint) this is strictly true, we quickly learned that people do occasionally dereference null pointers, and having the code execution just fall into the top of the next function makes it very difficult to understand the problem. From the performance angle, the most important aspect of exposing these is to squash downstream code. Because of this, clang turns these into a runtime trap: if one of these is actually dynamically reached, the program stops immediately and can be debugged. The drawback of doing this is that we slightly bloat code by having these operations and having the conditions that control their predicates.
5. The optimizer does go to some effort to "do the right thing" when it is obvious what the programmer meant (such as code that does "\*(int\*)P" when P is a pointer to float). This helps in many common cases, but you really don't want to rely on this, and there are lots of examples that you might think are "obvious" that aren't after a long series of transformations have been applied to your code.
6. Optimizations that don't fall into any of these categories, such the zero\_array and set/call examples in Part #1 are optimized as described, silently, without any indication to the user. We do this because we don't have anything useful to say, and it is very uncommon for (buggy) real-world code to be broken by these optimizations.

One major area of improvement we can make is with respect to trap insertion. I think it would be interesting to add an (off-by-default) warning flag that would cause the optimizer to warn whenever it generates a trap instruction. This would be extremely noisy for some codebases, but could be useful for others. The first limiting factor here is the infrastructure work to make the optimizer produce warnings: it doesn't have useful source code location information unless debugging information is turned on (but this could be fixed).

The other, more significant, limiting factor is that the warning wouldn't have any of the "tracking" information to be able to explain that an operation is the result of unrolling a loop three times and inlining it through four levels of function calls. At best we'll be able to point out the file/line/column of the original operation, which will be useful in the most trivial cases, but is likely to be extremely confusing in other cases. In any event, this hasn't been a high priority for us to implement because a) it isn't likely to give a good experience b) we won't be able to turn it on by default, and c) is a lot of work to implement.

## Using a Safer Dialect of C (and other options)

A final option you have if you don't care about "ultimate performance", is to use various compiler flags to enable dialects of C that eliminate these undefined behaviors. For example, using the `-fwrapv` flag eliminates undefined behavior that results from signed integer overflow (however, note that it does **not** eliminate possible integer overflow security vulnerabilities). The `-fno-strict-aliasing` flag disables Type Based Alias Analysis, so you are free to ignore these type rules. If there was demand, we could add a flag to Clang that implicitly zeros all local variables, one that inserts an "and" operation before each shift with a variable shift count, etc. Unfortunately, there is no tractable way to **completely** eliminate undefined behavior from C without breaking the ABI and completely destroying its performance. The other problem with this is that you're not writing C anymore, you're writing a similar, but non-portable dialect of C.

If writing code in a non-portable dialect of C isn't your thing, then the `-ftrapv` and `-fcatch-undefined-behavior` flags (along with the other tools mentioned before) can be useful weapons in your arsenal to track down these sorts of bugs. Enabling them in your debug builds can be a great way to find related bugs early. These flags can also be useful in production code if you are building security critical applications. While they provide no guarantee that they will find all bugs, they do find a useful subset of bugs.

Ultimately, the real problem here is that C just isn't a "safe" language and that (despite its success and popularity) many people do not really understand how the language works. In its decades of evolution prior to standardization in 1989, C migrated from being a "low level systems programming language that was a tiny layer above PDP assembly" to being a "low level systems programming language, trying to provide decent performance by _breaking many people's expectations_". On the one hand, these C "cheats" almost always work and code is generally more performant because of it (and in some cases, _much_ more performant). On the other hand, the places where C cheats are often some of the most surprising to people and typically strike at the worst possible time.

C is much more than a portable assembler, sometimes in very surprising ways. I hope this discussion helps explain some of the issues behind undefined behavior in C, at least from a compiler implementer's viewpoint.

- [Chris Lattner](http://nondot.org/sabre/)
