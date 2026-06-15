---
title: Announcing Failure
url: https://without.boats/blog/announcing-failure/
published: "2017-11-16T00:00:00Z"
feed: boats
guid: https://without.boats/blog/announcing-failure/
---

# Announcing Failure

I’m really excited to announce a new crate I’ve been working on, called failure, and which I’ve just released to crates.io. Failure is a Rust library intended to make it easier to manage your error types. This library has been heavily influenced by learnings we gained from previous iterations in our error management story, especially the Error trait and the error-chain crate.
The Fail trait The core abstraction in failure is the Fail trait, a replacement for the existing std::error::Error trait.
