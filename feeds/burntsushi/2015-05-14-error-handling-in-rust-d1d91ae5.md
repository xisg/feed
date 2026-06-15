---
title: Error Handling in Rust
url: https://burntsushi.net/rust-error-handling/
published: "2015-05-14T10:47:26Z"
feed: burntsushi
guid: https://burntsushi.net/rust-error-handling/
---

# Error Handling in Rust

Like most programming languages, Rust encourages the programmer to handle
errors in a particular way. Generally speaking, error handling is divided into
two broad categories: exceptions and return values. Rust opts for return
values.

In this article, I intend to provide a comprehensive treatment of how to deal
with errors in Rust. More than that, I will attempt to introduce error handling
one piece at a time so that you’ll come away with a solid working knowledge of
how everything fits together.

When done naively, error handling in Rust can be verbose and annoying. This
article will explore those stumbling blocks and demonstrate how to use the
standard library to make error handling concise and ergonomic.

**Target audience**: Those new to Rust that don’t know its error handling
idioms yet. Some familiarity with Rust is helpful. (This article makes heavy
use of some standard traits and some very light use of closures and macros.)

**Update (2018/04/14)**: Examples were converted to `?`, and some text was
added to give historical context on the change.

**Update (2020/01/03)**: A recommendation to use
[`failure`](https://crates.io/crates/failure) was removed and replaced with
a recommendation to use either `Box<Error + Send + Sync>` or
[`anyhow`](https://crates.io/crates/anyhow).
