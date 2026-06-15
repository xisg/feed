---
title: 'iou: Rust bindings for liburing'
url: https://without.boats/blog/iou/
published: "2019-11-08T00:00:00Z"
feed: boats
guid: https://without.boats/blog/iou/
---

# iou: Rust bindings for liburing

Today I’m releasing a library called iou. This library provides idiomatic Rust bindings to the C library called liburing, which itself is a higher interface for interacting with the io\_uring Linux kernel interface. Here are the answers to some questions I expect that may provoke.
What is io\_uring? io\_uring is an interface added to the Linux kernel in version 5.1. Concurrent with that, the primary maintainer of that interface has also been publishing a library for interacting with it called liburing.
