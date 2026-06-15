---
title: The AsyncIterator interface
url: https://without.boats/blog/async-iterator/
published: "2023-03-22T00:00:00Z"
feed: boats
guid: https://without.boats/blog/async-iterator/
---

# The AsyncIterator interface

In [a previous post](https://without.boats/blog/the-registers-of-rust), I established the notion of “registers” - code in Rust can be
written in different registers, and it’s important to adequately support all registers. I
specifically discussed the low-level interface of the AsyncIterator trait, about which there is
currently a debate. The interface it currently has is a method called `poll_next`, which is a “poll”
method like `Future::poll`. Poll methods are very “low-level” and are harder to write correctly than
async functions. Some people would like to see `AsyncIterator` shifted to have an async next method,
simply the “asyncified” `Iterator` trait.
