---
title: Using unwrap() in Rust is Okay
url: https://burntsushi.net/unwrap/
published: "2022-08-08T09:00:00Z"
feed: burntsushi
guid: https://burntsushi.net/unwrap/
---

# Using unwrap() in Rust is Okay

One day before Rust 1.0 was released, I published a
[blog post covering the fundamentals of error handling](https://blog.burntsushi.net/rust-error-handling/).
A particularly important but small section buried in the middle of the article
is named “ [unwrapping isn’t evil](https://blog.burntsushi.net/rust-error-handling/#a-brief-interlude-unwrapping-isnt-evil)”. That section briefly
described that, broadly speaking, using `unwrap()` is okay if it’s in
test/example code or when panicking indicates a bug.

I generally still hold that belief today. That belief is put into practice in
Rust’s standard library and in many core ecosystem crates. (And that practice
predates my blog post.) Yet, there still seems to be widespread confusion about
when it is and isn’t okay to use `unwrap()`. This post will talk about that
in more detail and respond specifically to a number of positions I’ve seen
expressed.

This blog post is written somewhat as a FAQ, but it’s meant to be read in
sequence. Each question builds on the one before it.

**Target audience**: Primarily Rust programmers, but I’ve hopefully provided
enough context that the principles espoused here apply to any programmer.
Although it may be tricky to apply an obvious mapping to languages with
different error handling mechanisms, such as exceptions.
