---
title: Unsafe Abstractions
url: https://without.boats/blog/unsafe-abstractions/
published: "2018-01-04T00:00:00Z"
feed: boats
guid: https://without.boats/blog/unsafe-abstractions/
---

# Unsafe Abstractions

Unsafety in Rust is often discussed in terms the primitive operations that can only be performed inside of unsafe blocks (such as dereferencing raw pointers and accessing mutable statics). I want to look at it from a different angle from these primitive operations, and instead focus on the capability to produce unsafe abstractions.
The general concept of unsafe abstractions An unsafe abstraction is a new abstraction which requires the unsafe keyword to apply to some context (this is an intentionally “abstract” definition, because as we will see there are several highly divergent forms of unsafe abstraction supported in Rust).
