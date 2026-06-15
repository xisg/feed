---
title: Another look at the pinning API
url: https://without.boats/blog/rethinking-pin/
published: "2018-08-22T00:00:00Z"
feed: boats
guid: https://without.boats/blog/rethinking-pin/
---

# Another look at the pinning API

A few months ago we introduced the concept of “pinned” references - pointers which “pin” the data they refer to in a particular memory location, guaranteeing that it will never move again. These are an important building block for certain patterns that had previously been hard for Rust to handle, like self-referential structs and intrusive lists, and we’ve in the process of considering stabilizing the API.
One thing has always nagged about the API we have right now though: the proliferation of different reference types that it implies.
