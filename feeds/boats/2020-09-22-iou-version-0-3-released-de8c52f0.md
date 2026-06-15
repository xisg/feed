---
title: iou version 0.3 released
url: https://without.boats/blog/iou-0-3/
published: "2020-09-22T00:00:00Z"
feed: boats
guid: https://without.boats/blog/iou-0-3/
---

# iou version 0.3 released

Today I made a new release of the [iou](https://github.com/ringbahn/iou) library, which contains idiomatic Rust bindings to the
[liburing](https://github.com/axboe/liburing) library. This library allows users to manipulate the new [io-uring](https://kernel.dk/io_uring.pdf)
interface for asynchronous IO on Linux. For more context, you can read [my previous post on the\
first release of iou last year](https://without.boats/blog/iou).

This new release greatly expands the API of iou, introduces some valuable improvements, and contains
some breakages. I figured I would let this blog post serve as some basic release notes.
