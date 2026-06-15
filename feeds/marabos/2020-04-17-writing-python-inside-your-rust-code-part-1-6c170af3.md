---
title: Writing Python inside your Rust code — Part 1
url: https://blog.m-ou.se/writing-python-inside-rust-1/
published: "2020-04-17T00:00:00Z"
feed: marabos
guid: https://blog.m-ou.se/writing-python-inside-rust-1/
---

# Writing Python inside your Rust code — Part 1

About a year ago, I published a Rust crate called [inline-python](https://crates.io/crates/inline-python),
which allows you to easily mix some Python into your Rust code using a `python!{ .. }` macro.
In this series, I’ll go through the process of developing this crate from scratch.
