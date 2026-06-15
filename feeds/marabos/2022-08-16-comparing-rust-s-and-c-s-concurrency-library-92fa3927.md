---
title: Comparing Rust's and C++'s Concurrency Library
url: https://blog.m-ou.se/rust-cpp-concurrency/
published: "2022-08-16T00:00:00Z"
feed: marabos
guid: https://blog.m-ou.se/rust-cpp-concurrency/
---

# Comparing Rust's and C++'s Concurrency Library

The concurrency features that are included in the Rust standard library
are quite similar to what was available in C++11: threads, atomics, mutexes, condition variables, and so on.
In the past few years, however, C++ has gained quite a few new concurrency related
features as part C++17 and C++20, with more proposals still coming in for future versions.

Let’s take some time to review C++ concurrency features,
discuss what their Rust equivalent could look like,
and what it’d take to get there.
