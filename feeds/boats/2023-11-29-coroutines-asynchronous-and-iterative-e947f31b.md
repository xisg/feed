---
title: Coroutines, asynchronous and iterative
url: https://without.boats/blog/coroutines-async-and-iter/
published: "2023-11-29T00:00:00Z"
feed: boats
guid: https://without.boats/blog/coroutines-async-and-iter/
---

# Coroutines, asynchronous and iterative

I wanted to follow up my [previous post](https://without.boats/blog/poll-next) with a small note elaborating on the use of
coroutines for asynchrony and iteration from a more abstract perspective. I realized the point I
made about `AsyncIterator` being the product of `Iterator` and `Future` makes a bit more sense if
you also consider the “base case” - a block of code that is neither asynchronous nor iterative.

It’s also an excuse to draw another fun ASCII diagram, and I’ve got to put that Berkeley Mono
license to good use.
