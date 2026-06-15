---
title: 'Ringbahn III: A deeper dive into drivers'
url: https://without.boats/blog/ringbahn-iii/
published: "2020-10-01T00:00:00Z"
feed: boats
guid: https://without.boats/blog/ringbahn-iii/
---

# Ringbahn III: A deeper dive into drivers

In [the previous post](../ringbahn-ii) in this series, I wrote about how the core state machine of
[ringbahn](https://github.com/ringbahn/ringbahn) is implemented. In this post I want to talk about another central concept in
ringbahn: **“drivers”**, external libraries which determine how ringbahn schedules IO operations
over an io-uring instance.
