---
title: What I meant by the "soul of Rust"
url: https://smallcultfollowing.com/babysteps/blog/2022/09/19/what-i-meant-by-the-soul-of-rust/?utm_source=atom_feed
published: "2022-09-19T00:00:00Z"
feed: babysteps
guid: https://smallcultfollowing.com/babysteps/blog/2022/09/19/what-i-meant-by-the-soul-of-rust/
---

# What I meant by the "soul of Rust"

Re-reading my [previous post](https://smallcultfollowing.com/babysteps/blog/2022/09/18/dyn-async-traits-part-8-the-soul-of-rust/), I felt I should clarify why I called it the “soul of Rust”. The soul of Rust, to my mind, is definitely **not** being explicit about allocation. Rather, it’s about the struggle between a few key values — especially _productivity_ and _versatility_ [1](#fn:1) in tension with _transparency_. Rust’s goal has always been to _feel_ like a high-level but with the performance and control of a _low-level_ one. Oftentimes, we are able to find a [“third way” that removes the tradeoff](https://smallcultfollowing.com/babysteps/blog/2019/04/19/aic-adventures-in-consensus/), solving both goals pretty well. But finding those “third ways” takes time — and sometimes we just have to accept a certain hit to one value or another for the time being to make progress. It’s exactly at these times, when we have to make a difficult call, that questions about the “soul of Rust” starts to come into play. I’ve been thinking about this a lot, so I thought I would write a post that expands on the role of transparency in Rust, and some of the tensions that arise around it.

## Why do we value transparency?

From the [draft Rustacean Principles](https://rustacean-principles.netlify.app/how_rust_empowers/transparent.html):

> 🔧 Transparent: “you can predict and control low-level details”

The C language, famously, maps quite closely to how machines typically operate. So much so that people have sometimes called it “portable assembly”.[2](#fn:2) Both C++ and Rust are trying to carry on that tradition, but to add on higher levels of abstraction. Inevitably, this leads to tension. Operator overloading, for example, makes figuring out what `a + b` more difficult.[3](#fn:3)

## Transparency gives you control

Transparency doesn’t automatically give high performance, but it does give control. This helps when crafting your system, since you can set it up to do what you want, but it also helps when analyzing its performance or debugging. There’s nothing more frustrating than starting at code for hours and hours only to realize that the source of your problem isn’t anywhere in the code you can see — it lies in some invisible interaction that wasn’t made explicit.

## Transparency can cost performance

The flip-side of transparency is overspecification. The more directly your program maps to assembly, the less room the compiler and runtime have to do clever things, which can lead to lower performance. In Rust, we are always looking for places where we can be _less_ transparent in order to gain performance — but only up to a point. One example is struct layout: the Rust compiler retains the freedom to reorder fields in a struct, enabling us to make more compact data structures. That’s less transparent than C, but usually not in a way that you care about. (And, of course, if you want to specify the order of your fields, we offer the `#[repr]` attribute.)

## Transparency hurts versatility and productivity

The bigger price of transparency, though, is versatility. It forces everyone to care about low-level details that may not actually matter to the problem at hand[4](#fn:4). Relevant to dyn async trait, most async Rust systems, for example, perform allocations left and right. The fact that a particular call to an async function might invoke `Box::new` is unlikely to be a performance problem. For those users, selecting a `Boxing` adapter adds to the overall complexity they have to manage for very little gain. If you’re working on a project where you don’t _need_ peak performance, that’s going to make Rust less appealing than other languages. I’m not saying that’s _bad_, but it’s a fact.

## A zero-sum situation…

At this moment in the design of async traits, we are struggling with a core question here of “how versatile can Rust be”. Right now, it feels like a “zero sum situation”. We can add in something like `Boxing::new` to preserve transparency, but it’s going to cost us some in versatility — hopefully not too much.

## …for now?

I do wonder, though, if there’s a “third way” waiting somewhere. I hinted at this a bit in the previous post. At the moment, I don’t know what that third way is, and I think that requiring an explicit adapter is the most practical way forward. But it seems to me that it’s not a perfect sweet spot yet, and I am hopeful we’ll be able to subsume it into something more general.

Some ingredients that might lead to a ‘third way’:

- _With-clauses or capabilities:_ I am intrigued by the idea of \[with-clauses\] and the general idea of scoped capabilities. We might be able to think about the “default adapter” as something that gets specified via a with-clause?
- _Const evaluation:_ One of the niftier uses for const evaluation is for “meta-programming” that customizes how Rust is compiled. For example, we could potentially let you write a `const fn` that creates the vtable data structure for a given trait.
- _Profiles and portability:_ Can we find a better way to identify the kinds of transparency that you want, perhaps via some kind of ‘profiles’? I feel we already have ‘de facto’ profiles right now, but we don’t recognize them. “No std” is a clear example, but another would be the set of operating systems or architectures that you try to support. Recognizing that different users have different needs, and giving people a way to choose which one fits them best, might allow us to be more supportive of all our users — but then again, it might make it make Rust “modal” and more confusing.

### Comments?

Please leave comments in [this internals thread](https://internals.rust-lang.org/t/blog-series-dyn-async-in-traits-continues/17403). Thanks!

## Footnotes

* * *

1. I didn’t write about versatility in my original post: instead I focused on the hit to productivity. But as I think about it now, versatility is really what’s at play here — versatility really meant that Rust was useful for high-level things _and_ low-level things, and I think that requiring an explicit dyn adaptor is unquestionably a hit against being high-level. Interestingly, I put versatility _after_ transparency in the list, meaning that it was lower priority, and that seems to back up the decision to have some kind of explicit adaptor. [↩︎](#fnref:1)

2. At this point, some folks point out all the myriad subtleties and details that are actually hidden in C code. Hush you. [↩︎](#fnref:2)

3. I remember a colleague at a past job discovering that somebody had overloaded the `->` operator in our codebase. They sent out an angry email, “When does it stop? Must I examine every dot and squiggle in the code?” (NB: Rust supports overloading the deref operator.) [↩︎](#fnref:3)

4. Put another way, being transparent about one thing can make other things more obscure (“can’t see the forest for the trees”). [↩︎](#fnref:4)
