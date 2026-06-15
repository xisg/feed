---
title: Regex engine internals as a library
url: https://burntsushi.net/regex-internals/
published: "2023-07-05T09:06:00Z"
feed: burntsushi
guid: https://burntsushi.net/regex-internals/
---

# Regex engine internals as a library

Over the last several years, I’ve rewritten [Rust’s `regex`\
crate](https://github.com/rust-lang/regex/) to enable better internal composition, and to make it
easier to add optimizations while maintaining correctness. In the course of
this rewrite I created a new crate, [`regex-automata`](https://github.com/rust-lang/regex/tree/master/regex-automata), which exposes much
of the `regex` crate internals as their own APIs for others to use. To my
knowledge, this is the first regex library to expose its internals to the
degree done in `regex-automata` as a separately versioned library.

This blog post discusses the problems that led to the rewrite, how the rewrite
solved them and a guided tour of `regex-automata`’s API.

**Target audience**: Rust programmers and anyone with an interest in how one
particular finite automata regex engine is implemented. Prior experience with
regular expressions is assumed.
