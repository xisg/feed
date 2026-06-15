---
title: Writing Python inside your Rust code — Part 4
url: https://blog.m-ou.se/writing-python-inside-rust-4/
published: "2020-05-24T00:00:00Z"
feed: marabos
guid: https://blog.m-ou.se/writing-python-inside-rust-4/
---

# Writing Python inside your Rust code — Part 4

In this final part of the series,
we’ll explore a trick to make the behaviour of a macro depend
on whether it’s used as a statement or as part of an expression.
Using that, we’ll make the `python!{}` macro
more flexible to allow saving, reusing, and inspecting Python variables.
