---
title: Changing the rules of Rust
url: https://without.boats/blog/changing-the-rules-of-rust/
published: "2023-09-17T00:00:00Z"
feed: boats
guid: https://without.boats/blog/changing-the-rules-of-rust/
---

# Changing the rules of Rust

In Rust, there are certain API decisions about what is and isn’t sound that impact all Rust code.
That is, a decision was made to allow or not allow types which have certain safety requirements, and
now all users are committed to that decision. They can’t just use a different API with different
rules: _all_ APIs must conform to these rules.

These rules are determined through certain “marker” traits. If a safe API could do something to a
value of a type which some types don’t support, the API must be bound by that marker trait, so that
users can not pass values of those types which don’t support that behavior to that API. In contrast,
if Rust allows APIs to perform that behavior on any type, without any sort of marker trait bound,
then types which don’t support that behavior cannot exist.

I’m going to give three examples to show what I mean, each of which Rust has considered at different
points, though only the first one actually exists in Rust.
