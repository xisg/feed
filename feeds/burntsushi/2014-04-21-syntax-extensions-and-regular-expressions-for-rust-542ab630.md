---
title: Syntax extensions and regular expressions for Rust
url: https://burntsushi.net/rust-regex-syntax-extensions/
published: "2014-04-21T19:51:00Z"
feed: burntsushi
guid: https://burntsushi.net/rust-regex-syntax-extensions/
---

# Syntax extensions and regular expressions for Rust

**WARNING:** 2018-04-12: The code snippets for this post are no longer
available. This is just as well anyway, since they all depended on an unstable
internal compiler interface, which hasn’t existed for years.

A few weeks ago, I set out to add regular expressions to the
[Rust](http://www.rust-lang.org/)
distribution with an implementation and feature set heavily inspired by
[Russ Cox’s RE2](http://swtch.com/~rsc/regexp/).
It was just recently added to the
[Rust distribution](http://static.rust-lang.org/doc/master/regex/index.html).

This regex crate includes a syntax extension that compiles a regular expression
to native Rust code _when a Rust program is compiled_. This can be thought of
as “ahead of time” compilation or
something similar to [compile time function\
execution](http://en.wikipedia.org/wiki/Compile_time_function_execution).
These special natively compiled regexes have the _same exact_ API as regular
expressions compiled at runtime.

In this article, I will outline my implementation strategy—including code
samples on how to write a Rust syntax extension—and describe how I was able
to achieve an identical API between regexes compiled at compile time and
regexes compiled at runtime.
