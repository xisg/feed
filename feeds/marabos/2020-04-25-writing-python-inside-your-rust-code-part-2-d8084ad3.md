---
title: Writing Python inside your Rust code — Part 2
url: https://blog.m-ou.se/writing-python-inside-rust-2/
published: "2020-04-25T00:00:00Z"
feed: marabos
guid: https://blog.m-ou.se/writing-python-inside-rust-2/
---

# Writing Python inside your Rust code — Part 2

In this part, we’ll extend our `python!{}`-macro to be able to seamlessly
use Rust variables in the Python code within.
We explore a few options, and implement two alternatives.
