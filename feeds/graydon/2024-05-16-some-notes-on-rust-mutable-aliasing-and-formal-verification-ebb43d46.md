---
title: Some notes on Rust, mutable aliasing and formal verification
url: https://graydon2.dreamwidth.org/312681.html
published: "2024-05-16T03:15:11Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:312681
---

# Some notes on Rust, mutable aliasing and formal verification

Recently [Boats wrote a blog post](https://without.boats/blog/references-are-like-jumps/) about Rust, mutable aliasing, and the sad story of local reasoning over many decades of computer science. I recommend that post and agree with its main points! Go read it! But I also thought I'd add a little more detail to an area it's less acutely focused on: formal methods / formal verification.

**TL;DR**: support for _local_ reasoning is a big factor in the ability to do _automated_ reasoning about programs. Formal verification involves such reasoning. Rust supports it better than many other imperative systems languages -- even some impure functional ones! -- and formal verification people are excited and building tools presently. This is not purely by accident, and is worth understanding as part of what makes Rust valuable beyond "it doesn't need a GC".

The rest of this post is just unpacking and giving details of one or more of the above assertions, which I'll try to address in order of plausible interestingness to the present, but I will also throw in some history because I kinda can't help myself.

**What even are formal verification tools?**

[Formal methods](https://en.wikipedia.org/wiki/Formal_methods) or [formal verification](https://en.wikipedia.org/wiki/Formal_verification) tools are tools that _prove_ properties hold over large (possibly infinite) amounts of a program's entire possible state space, rather than testing the program on individual paths through that state space. They use _computational logics_ for this, sometimes in very complex ways. They are a very old field in computer science, going back at least 50 years. A certain contingent -- including the famous [Tony Hoare](https://en.wikipedia.org/wiki/Tony_Hoare) (no relation) -- have maintained since the late 60s that we _really ought_ to be proving all of our programs correct, and that it will be tractable to do so any day now (or perhaps already is if we tried harder).

There's way too much to say about formal verification to summarize in a blog post, but [I gave a talk recently covering some of it](http://venge.net/assets/talks/formal-world.pdf). It does _sometimes_ work, but it's not yet made it anywhere near as big as its proponents have always wanted, and there's good reason to assume it will always remain relatively niche. Some of the limited uptake of formal verification is socio-economic (discussed in more depth that talk) and some of it is strictly computer-science-theoretic (see for example [Ron Pressler's writing on the subject](https://pron.github.io/posts/correctness-and-complexity)) but within those limits, certain small niches like safety-critical code are sometimes willing to tolerate the limits in order to experience the rewards.

In the _best_ case -- and this is actually approached, sometimes, on some codebases, by some tools today -- the developer experience can be something like "you write an `assert!()` in your code, and the IDE puts a green squiggly underline when it can prove that that assert always holds over all possible executions, and a red squiggly underline if it has a counterexample possible-execution where the assert won't hold". Absolute _magic_.

**How does local reasoning relate to formal verification?**

Formal verification involves inferring, propagating, resolving and relating formulas in sound computational logics (eg. [Hoare logic](https://en.wikipedia.org/wiki/Hoare_logic)) that characterize a program's possible states. These formulas start with the literals and variables in a program and are then combined by rules (eg. [Dijkstra's predicate-transformer rules](https://en.wikipedia.org/wiki/Predicate_transformer_semantics)) driven by the composite syntactic and semantic forms of the programming language, including function applications, variable assignments, conditional branching, sequential composition and so forth.

The rules for combining logical formulas across variable assignments typically have to modify all formulas involving the modified variable to correctly account for the change. This is a problem if the underlying programming language has uncontrolled mutable aliasing: potentially _every logical formula_ might change when you make an assignment through a reference, because the logic has no idea what other variables the reference might write-to in a given run of the program. Essentially "all bets are off" at any write through a pointer.

There are various partial workarounds -- reasoning about type disjointness, or about sets of disjoint possible-aliases, eventually the development of a whole family of logics (eg. [separation logic](https://en.wikipedia.org/wiki/Separation_logic)) that account for the disjointness of portions of the heap and the reachable "footprints" of functions -- but for a very, very long time the field has been held back by programming languages with too much mutable aliasing to be tractable.

**And Rust makes this better?**

Yes. Rust references are subject to the "shared-xor-mutable" rule, which means a write through a reference _doesn't_ invalidate any other variables in existing formulas. This makes it much easier to write a tool that verifies Rust code. Don't just believe me! Here's formal verification hacker Xavier Denis saying as much:

> rust's avoidance of shared mutable state has deep consequences; when we formally verify programs in Rust we can use FOL and avoid separation logic since the type system protects us from mutable aliasing, while this is not true in caml despite being 'functional'
>
> — xavxav (@xldenis) [May 14, 2024](https://twitter.com/xldenis/status/1790297114519404692?ref_src=twsrc%5Etfw)

**What Rust formal verification projects are there?**

Tons! And they're very exciting. [Prusti](https://github.com/viperproject/prusti-dev), [Kani](https://github.com/model-checking/kani), [Crux-Mir](https://github.com/GaloisInc/crucible/tree/master/crux-mir), [Aeneas](https://github.com/AeneasVerif/aeneas), [Flux](https://github.com/flux-rs/flux), [Creusot](https://github.com/creusot-rs/creusot), [MIRAI](https://github.com/facebookexperimental/MIRAI), [Hax](https://github.com/hacspec/hax), [RustHorn](https://github.com/hopv/rust-horn), [Rustproof](https://github.com/Rust-Proof/rustproof), [Verus](https://github.com/verus-lang/verus) .. plus lots of [supporting crates and projects in computational logic](https://github.com/newca12/awesome-rust-formalized-reasoning).

**Was Rust designed for formal verification?**

_Kinda_. There are two fairly different ways in which this is partly true, one usually forgotten and one possibly non-obvious: typestate and borrow checking. I will discuss both briefly below. Both are what we'd at best call "lightweight" formal methods, in the sense that the annotation burden is comparatively low, the word "logic" doesn't make much of an appearance, and the verification is fast and reliable, integrated into the normal workflow of programming. Indeed, some people call type systems themselves "lightweight formal methods", and we had no shortage of people on the team who were comfortable (much more comfortable than me!) with this sort of thing.

Moreover, even before the project was overrun by MIT and CMU folks (j/k I love you all), I was generally aware of Hoare logics when doing early Rust, and the extent to which mutable aliasing was a problem for them, and that limiting mutable aliasing helped. I was aware that pure-functional languages, affine languages, or those with un-observable writes (eg. with Copy-on-Write) could propagate logical formulas more readily. I was aware Butler Lampson and Jim Horning's "verification-oriented" [Euclid systems language](https://en.wikipedia.org/wiki/Euclid_(programming_language)), for example, had shipped pointers with disjointness guarantees to support local reasoning as far back as the late 70s. I even registered and maintained the domain name `rust-proof.org` early on, thinking it might wind up coming in handy if we went further in that direction.

But in a broader sense it is not true that Rust was "designed for" formal verification. I was _not_ totally fluent in Dijkstra's approach to automatic generation of verification condition formulas in Hoare logics, and I was not closely following the ongoing revolutions in either separation logic or [CDCL SAT solvers](https://en.wikipedia.org/wiki/Conflict-driven_clause_learning) that were pushing the boundary of that approach. I had (wrongly) convinced myself well before Rust that the approach was a bit of a weird dead end, and believed that the future of formal methods was either in stepwise refinement calculi like [B-method](https://en.wikipedia.org/wiki/B-Method), or interactive dependent-type or higher-order-logic provers like [Rocq (nee Coq)](https://en.wikipedia.org/wiki/Coq_(software)) or [HOL](https://en.wikipedia.org/wiki/HOL_(proof_assistant)), or other sorts of pure-functional hybrid logic languages like [ACL2](https://en.wikipedia.org/wiki/ACL2). I did not think any of these was likely to be usable for the broad audience of systems programmers I was aiming Rust at, and was happy to limit its formal verification ambitions to something I could explain to normal programmers, like a typestate system.

(And I was actually a bit annoyed at how involved the analysis got for the borrow checker; it's long passed my ability to accurately describe what it's doing, and I apologize for the somewhat mangled explanations to follow.)

**Typestate**

A usually-forgotten chapter in Rust's history is that it initially had a typestate system. [Typestate systems](https://en.wikipedia.org/wiki/Typestate_analysis) let you annotate a program's variables with n-ary predicates that get checked dynamically when they're initially established, but are also subject to static analysis of their propagation and validity as values flow through the program. For example you can mark a function as requiring a given predicate on its arguments, and then it becomes an obligation for the caller to have established somewhere before the call (or a requirement they can place on their caller, etc.)

Eventually people figured out that you can emulate typestates fairly well (though a bit awkwardly) with a combination of affine types and phantom types, both of which Rust supports, so the typestate feature was dropped from the language, but this was a part of the original design that lived quite a ways into the project's life, and was part of the motivation for control over mutable aliasing. If you ever see mention of [Hermes](https://en.wikipedia.org/wiki/Hermes_(programming_language)) and NIL in Rust's influences, this is what is meant: these languages have typestates and prohibit mutable aliasing in order to support it.

**Borrow checking**

Borrow checking is the other motivation, and it's an interesting case because it both requires _and provides_ local reasoning support. It's a key actor in the story! Its main purpose is to prove that any given access through a `&` or `&mut` reference is accessing valid memory. To do this it sets out to prove the property that reference lifetimes are shorter than referent lifetimes: a pointer always points to something live.

But it turns out mere lifetime nesting isn't enough, because Rust is a language with non-uniform representation: not every pointer points to a separate heap cell, but rather a `&` or `&mut` reference can point into the middle of some existing value, and that might be a value like an enum that can change size or shape when it's overwritten. So to ensure memory safety, the borrow checker also needs to know that a referenced value with a variable size or shape _doesn't change size or shape while referenced_, and so it _also_ enforces the "shared-xor-mutable" property on references. And somewhat coincidentally, this rule in turn provides quite strong local reasoning for _any_ property of Rust programs you might want to formally verify.

While earlier versions of Rust had a much more limited version of `&` and `&mut` references -- they were "second class", could not be returned from functions or embedded in structs, only passed as arguments -- they still provided strong support for local reasoning, for the same memory-safety reasons. But their limited expressivity meant the machinery required to check and enforce their properties was much simpler than the full glory of the modern borrow checker: there was a [simple path-and-type-disjointness pass called the "alias checker"](https://github.com/rust-lang/rust/commit/beda82ddf1f482f286a8d9af3402626dc56d6fea#diff-966e2bfbe831a658c4a89ea9bad0a12ec9d85d6dbf46318e7bd59f2e297c2946).

**How is this related to GC?**

One way to look at reference counting is as a sort of eager, optimized form of garbage collection that works in the case that you have strictly acyclic data (or can tolerate leaking cycles). And one way to look at the `&` and `&mut` reference types in Rust is as a way to _further_ optimize reference counting, all the way down to nothing: if you prove the referent outlives the reference, you can avoid reference counting altogether. Of course the `&` and `&mut` reference types do other things too -- for example they let you abstract over the difference between reference-counted heap allocation pointers and pointers to the interior of the stack or other objects -- but to a first approximation, when people call these references (and the supporting alias-and-later-borrow-checking pass) "compile-time garbage collection" that is not too far from the point. Strong local reasoning support just comes along for the ride.

Conversely, languages that _have_ a GC have unfortunately often not felt it necessary to bother having strong local reasoning support. Java for example has a GC and also uncontrolled mutable aliasing, so weak local reasoning and consequently more-challenging formal verification. If you write to an object in Java every other variable referencing that object "sees" the change, so any logical formula that held over all those variables might be invalidated by the write unless you use some fancy method of tracking possible write-sets or separating the heap.

**Didn't Rust once have a full, tracing, not-just-reference-counting GC?**

Yep! Rust had reference counting with a backup tracing GC on _part_ of its memory graph in order to help support the use case of modeling the DOM in a web browser, which was considered an important use case by some early Mozilla reviewers. As it happened, we couldn't get the tracing part performing well enough to take over from the reference counting part entirely, and also people weren't using these shared pointer types as much in idiomatic Rust as they were using the unique pointer types. So we removed the tracing GC from the shared-mutable type, and then eventually pushed both the unique pointer (Box) and shared pointer (Rc and Arc) types _and_ a mutable-cell type (RefCell) into the standard library.

What's important to understand is that, as with Rust's Rc type today, the shared pointer graph in Rust-with-a-GC was not responsible for the whole pointer graph. It statically differentiated "shared and immutable" parts of the memory graph from the "shared and mutable" parts, and put the shared-and-mutable stuff behind runtime borrow checks to prevent simultaneous mutable aliasing with `&` references. Essentially it functioned like `Rc<RefCell<T>>` today does, which an attentive reader will note can still be tied into reference-cycles, and lacking a GC such a cycle will now just leak! So just imagine that type, but with a cycle collector attached to it automatically by the language runtime to prevent leaks, and a little syntax sugar. You could revive this today if you wanted, and there are even a few Rust libraries that do.

**Wait, does that mean `Rc<RefCell<T>>` is a mutable alias?**

It's fair to view it that way! You can also look at unsafe Rust as a way to get mutable aliasing. Arguably so is access to stuff outside the program's memory, such as performing IO against a shared filesystem or whatever; at least if you were trying to do automated reasoning about such things. The main difference with other languages is in the _ubiquity_ of such problematic mutable aliases. The typical Rust program has either no refcells, or a very small number, just as it has either no unsafe code or a very small amount; so it's easier to reason about them as a special case and, say, exclude them from the variables being analyzed or develop [specialized rules for them](https://arxiv.org/abs/2405.08372).

Moreover the RefCell type does a dynamic check for the shared-xor-mutable rule, so _from the perspective of the borrow checker in particular_ it's safe (and sound) to assume any execution that actually occurs (i.e. that does not panic) upholds the shared-xor-mutable rule. Put another way: contrary to some wild claims circulating on the internet, `Rc<RefCell<T>>` does not "turn off" the borrow checker, it just moves one of the checks from compile time to runtime. Turning it off would let you break memory safety (eg by taking a reference to an enum then overwriting it with a different-shaped value and reading back garbage); but that's still not possible. The worst that can happen when using `Rc<RefCell<T>>` is a panic.

But from the perspective of a different verifier looking at different properties then yes, a formula that includes an `Rc<RefCell<T>>` might be invalidated by a write to some other `Rc<RefCell<T>>` pointing to the same value, which might defeat formal verification of the properties of interest.

In fact early versions of Rust didn't support this sort of thing _at all_. It made all shared-mutable allocations Copy-on-Write, so writing to a shared-mutable allocation made the write private, the other observers of the shared allocation would continue to see the old value. This is a fairly common design choice for languages trying to maintain local reasoning while accepting the need to do mutation sometimes. Rust copied it from a fairly old language called [Newsqueak](https://en.wikipedia.org/wiki/Newsqueak), but you will see it also in newer languages like [Swift](https://en.wikipedia.org/wiki/Swift_(programming_language)). Unfortunately like pure-functional programming, it's something users of lower-level systems languages tend to reject due to both its unpredictable performance and the fact that some programs really do _want_ some small amounts of mutable aliasing.

**How is this related to threading?**

Threading in the presence of mutable aliasing usually makes formal verification much harder; or put another way it makes local reasoning much less plausible than just "normal" sequential mutable aliasing. Now you don't just have variable assignments invalidating logical formulas, you have _everything that a concurrent thread might write to_ potentially being invalidated _at any time_. A lot of formal verification tools limit themselves to a single-threaded model of the language they're working with.

**Is this why Rust has Fearless Concurrency (tm)?**

To an extent. Rust built up a lot of machinery to restrict the ways concurrent threads can interact in order both to make multithreaded programming in Rust less error-prone in general, but _also_ to just preserve the local reasoning required by its "lightweight formal verification" passes, mentioned above, specifically the borrow checker. The borrow checker couldn't rely on anything if it was racing with other threads everywhere. Neither could typestate.

Again, thinking about analogies to GC, this is in some ways similar to how runtime automatic memory management -- refcounting or tracing GC -- often has to work harder in the presence of threads, with write barriers or collection restarts or atomic operations or such.

Earlier versions of Rust didn't actually support multiple threads touching the same memory at all. It was trying to be an Actor Language, so limited communication to messages sent over channels, and leveraged the Copy-on-Write machinery to make shallow copies of messages sent between tasks in the same thread, and deep copies when sending between threads.

**Phew, that is a lot of notes**

Yeah, this is paraphrasing a bunch from earlier conversations but I've wanted to write about it publicly for a long time. Thanks Boats for the motivation!

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=312681) comments
