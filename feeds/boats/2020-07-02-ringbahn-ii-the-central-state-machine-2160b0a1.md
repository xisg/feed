---
title: 'Ringbahn II: the central state machine'
url: https://without.boats/blog/ringbahn-ii/
published: "2020-07-02T00:00:00Z"
feed: boats
guid: https://without.boats/blog/ringbahn-ii/
---

# Ringbahn II: the central state machine

[Last time](../ringbahn) I wrote about [ringbahn](https://github.com/withoutboats/ringbahn), a safe API for using io-uring from Rust.
I wrote that I would soon write a series of posts about the mechanism that makes ringbahn work. In
the first post in that series, I want to look at the core state machine of ringbahn which makes it
memory safe. The key types involved are the [Ring](https://docs.rs/ringbahn/0.0.0-experimental.3/ringbahn/struct.Ring.html) and [Completion](https://docs.rs/crate/ringbahn/0.0.0-experimental.3/source/src/completion.rs) types.
