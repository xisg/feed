---
title: 'Ringbahn: a safe, ergonomic API for io-uring in Rust'
url: https://without.boats/blog/ringbahn/
published: "2020-05-27T00:00:00Z"
feed: boats
guid: https://without.boats/blog/ringbahn/
---

# Ringbahn: a safe, ergonomic API for io-uring in Rust

In [my previous post,](../io-uring) I discussed the new io-uring interface for Linux, and how to create
a safe API for using io-uring from Rust. In the time since that post, I have implemented a prototype
of such an API. The crate is called [**ringbahn**](https://github.com/withoutboats/ringbahn), and it is intended to enable users to
perform IO on io-uring without any risk of memory unsafety.
