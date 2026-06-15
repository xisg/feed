---
title: Three problems of pinning
url: https://without.boats/blog/three-problems-of-pinning/
published: "2023-11-30T00:00:00Z"
feed: boats
guid: https://without.boats/blog/three-problems-of-pinning/
---

# Three problems of pinning

When we developed the [Pin](https://doc.rust-lang.org/std/pin/struct.Pin.html) API, our vision was that “ordinary users” - that is, users using
the “high-level” [registers](https://without.boats/blog/the-registers-of-rust/) of Rust, would never have to interact with it. We intended
that only users implementing Futures by hand, in the “low-level” register, would have to deal with
that additional complexity. And the benefit that would accrue to all users is that futures, being
immovable while polling, could store self-references in their state.

Things haven’t gone perfectly according to plan. The benefits of `Pin` have certainly been accrued -
everyone is writing self-referential async functions all the time, and low-level concurrency
primitives in all the major runtimes take advantage of `Pin` to implement intrusive linked lists
internally. But `Pin` still sometimes rears its ugly head into “high-level” code, and users are
unsurprisingly frustrated and confused when that happens.

In my experience, there a three main ways that this happens. Two of them can be solved by better
affordances for `AsyncIterator` (a part of why I have been pushing stabilizing this so hard!). The
third is ultimately because of a mistake that we made when we designed `Pin`, and without a breaking
change there’s nothing we could about it. They are:

1. Selecting a `Future` in a loop.
2. Calling `Stream::next`.
3. Awaiting a `Future` behind a pointer (e.g. a boxed future).
