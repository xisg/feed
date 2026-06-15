---
title: 'Shifgrethor II: Notes on tracing garbage collectors'
url: https://without.boats/blog/shifgrethor-ii/
published: "2018-10-22T00:00:00Z"
feed: boats
guid: https://without.boats/blog/shifgrethor-ii/
---

# Shifgrethor II: Notes on tracing garbage collectors

In the previous post I said that in the second post in the series we’d talk about how rooting works. However, as I sat down to write that post, I realized that it would be a good idea to back up and give an initial overview of how a tracing garbage collector works - and in particular, how the underlying garbage collector in shifgrethor is implemented.
In the abstract, we can think of the memory of a Rust program with garbage collection as being divided into three sections: the stack, the “unmanaged” heap, and the “managed” heap.
