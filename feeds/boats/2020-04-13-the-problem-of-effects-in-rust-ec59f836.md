---
title: The problem of effects in Rust
url: https://without.boats/blog/the-problem-of-effects/
published: "2020-04-13T00:00:00Z"
feed: boats
guid: https://without.boats/blog/the-problem-of-effects/
---

# The problem of effects in Rust

In [a previous post](https://without.boats/blog/why-ok-wrapping/), I shortly discussed the concept of “effects” and the parallels
between them. In an unrelated post since then, [Yosh Wuyts](https://blog.yoshuawuyts.com/fallible-iterator-adapters/) writes about the
problem of trying to write fallible code inside of an iterator adapter that doesn’t support it. In a
[previous discussion](https://internals.rust-lang.org/t/idea-non-local-control-flow/11976), the users of the Rust Internals forum hotly discuss the notion of
closures which would maintain the so-called [“Tennant’s Correspondence Principle”](https://gafter.blogspot.com/2006/08/tennents-correspondence-principle-and.html) \- that is,
closures which support breaking to scopes outside of the closure, inside of the function they are in
(you can think of this is closures capturing their control flow environment in addition to capturing
variables).

I think it may not be obvious, but these discussions are all deeply related. They all arise from
what is, in my opinion, one of the biggest problems with the design of the Rust language: its
failure at 1.0 to give good support for handling common effects related to program control flow.
