---
title: Futures and Segmented Stacks
url: https://without.boats/blog/futures-and-segmented-stacks/
published: "2020-06-08T00:00:00Z"
feed: boats
guid: https://without.boats/blog/futures-and-segmented-stacks/
---

# Futures and Segmented Stacks

This is just a note on getting the best performance out of an async program.

The point of using async IO over blocking IO is that it gives the user program more control over
handling IO, on the premise that the user program can use resources more effectively than the kernel
can. In part, this is because of the inherent cost of context switching between the userspace and
the kernel, but in part it is also because the user program can be written with more specific
understanding of its exact requirements.
