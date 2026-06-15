---
title: A brief apology of Ok-Wrapping
url: https://without.boats/blog/why-ok-wrapping/
published: "2020-04-07T00:00:00Z"
feed: boats
guid: https://without.boats/blog/why-ok-wrapping/
---

# A brief apology of Ok-Wrapping

I’ve _long_ been a proponent of having some sort of syntax in Rust for writing functions which
return results which “ok-wrap” the happy path. This is has also always been a feature with very
vocal, immediate, and even emotional opposition from many of our most enthusiastic users. I want to
write, in one place, why I think this feature would be awesome and make Rust much better.

I don’t want to get into the details too much of the specific proposal, but here’s a sketch of one
way this _could_ work (there are a number of variables). We would add a syntactic modifier to the
signature of a function, like this:

```rust
fn foo() -> usize throws io::Error {
 //..
}

```

This function returns `Result<usize, io::Error>`, but internally the `return` expressions return a
value of type `usize`, not the Result type. They are “Ok-wrapped” into being `Ok(usize)`
automatically by the language. If users wish to throw an error, a new `throw` expression is added
which takes the error side (the type after `throws` in the signature). The `?` operator would behave
in this context the same way it behaves in a function that returns `Result`.
