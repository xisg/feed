---
title: Pinned places
url: https://without.boats/blog/pinned-places/
published: "2024-07-23T00:00:00Z"
feed: boats
guid: https://without.boats/blog/pinned-places/
---

# Pinned places

In the previous [post](https://without.boats/blog/pin), I described the goal of Rust’s `Pin` type and the history of how it
came to exist. When we were initially developing this API in 2018, one of our explicit goals was the
limit the number of changes we would make to Rust, because we wanted to ship a “minimum viable
product” of async/await syntax as soon as possible. This meant that `Pin` is a type defined in the
standard library, without any syntactic or language support except for the ability to use it as a
method receiver. As I wrote in my previous post, in my opinion this is the source of a “complexity
cliff” when users have to interact with `Pin`.

We knew when we made this choice that pinned references would be harder to use and more confusing
than ordinary references, though I think we did underestimate just how much more challenging they
would be for most users. Our initial hope was that with async/await, pinning would disappear into
the background, because the `await` operator and the runtime’s `spawn` function would pin your
futures for you and you wouldn’t have to encounter it directly. As things played out, there are
still some cases where users must interact with pinned references, even when using async/await. And
sometimes users do need to “drop down” into a lower-level [register](https://without.boats/blog/the-registers-of-rust) to implement
`Future` themselves; this is when they truly encounter a huge complexity cliff: both the essential
complexity of implementing a state machine “by hand” and the additional complexity of understanding
the APIs to do with `Pin`.

My contention in my previous post was that the difficulties involved in this have very little to do
with the complexity inherent in the pinned typestate as a concept, or in pinned references as a way
of representing it, but instead arises from the fact that `Pin` is a pure library type without
support from the language. Users who deal with `Pin` are almost always doing something that is
totally memory safe, the problem is just that the idioms to do so with `Pin` are different from and
less clear than the idioms for doing so with ordinary references.

In this post, I want to propose a set of language changes - completely backward compatible with the
language as it exists and the async ecosystem built on `Pin` \- which will make interacting with
pinned references much more similar to interacting with ordinary references.
