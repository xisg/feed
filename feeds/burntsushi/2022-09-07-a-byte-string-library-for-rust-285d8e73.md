---
title: A byte string library for Rust
url: https://burntsushi.net/bstr/
published: "2022-09-07T14:00:00Z"
feed: burntsushi
guid: https://burntsushi.net/bstr/
---

# A byte string library for Rust

[`bstr`](https://docs.rs/bstr/1.*) is a byte string library for Rust and [its 1.0 version has\
just been released](https://github.com/BurntSushi/bstr/releases/tag/1.0.0)! It provides string oriented operations on
arbitrary sequences of bytes, but is most useful when those bytes are UTF-8. In
other words, it provides a string type that is UTF-8 by _convention_, where as
Rust’s built-in string types are _guaranteed_ to be UTF-8.

This blog will briefly describe the API, do a deep dive on the motivation for
creating `bstr`, show some short example programs using `bstr` and conclude
with a few thoughts.

**Target audience**: Rust programmers with some background knowledge of UTF-8.
