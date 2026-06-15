---
title: Notes on io-uring
url: https://without.boats/blog/io-uring/
published: "2020-05-06T00:00:00Z"
feed: boats
guid: https://without.boats/blog/io-uring/
---

# Notes on io-uring

Last fall I was working on a library to make a safe API for driving futures on top of an an io-uring
instance. Though I released bindings to liburing called [iou](https://github.com/withoutboats/iou), the futures integration, called
ostkreuz, was never released. I don’t know if I will pick this work up again in the future but
several different people have started writing other libraries with similar goals, so I wanted to
write up some notes on what I learned working with io-uring and Rust’s futures model. This post
assumes some level of familiarity with the io-uring API. A high level overview is provided in [this\
document](https://kernel.dk/io_uring.pdf).
