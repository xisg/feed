---
title: 'Propane: an experimental generator syntax for Rust'
url: https://without.boats/blog/propane/
published: "2020-08-06T00:00:00Z"
feed: boats
guid: https://without.boats/blog/propane/
---

# Propane: an experimental generator syntax for Rust

I’ve just released a new crate called [propane](https://crates.io/crates/propane), which is a library for writing **generator functions**. It can only run on nightly:

```rust
#![feature(generators, generator_trait, try_trait)]

#[propane::generator]
fn fizz_buzz() -> String {
 for x in 1..101 {
 match (x % 3 == 0, x % 5 == 0) {
 (true, true) => yield String::from("FizzBuzz"),
 (true, false) => yield String::from("Fizz"),
 (false, true) => yield String::from("Buzz"),
 (..) => yield x.to_string(),
 }
 }
}

```
