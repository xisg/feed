---
title: Shipping Const Generics in 2020
url: https://without.boats/blog/shipping-const-generics/
published: "2020-07-16T00:00:00Z"
feed: boats
guid: https://without.boats/blog/shipping-const-generics/
---

# Shipping Const Generics in 2020

It’s hard to believe that its been more than 3 years since I opened [RFC 2000](https://github.com/rust-lang/rfcs/pull/2000), which defined
the const generics for Rust. At the same time, reading the RFC thread, there’s also been a huge
amount of change in this area: for one thing, at the time the RFC was written, const fns weren’t
stable, and consts weren’t even being evaluated using miri yet. There’s been a lot of work over the
years on the const generics feature, but still nothing has shipped. However, I think we have defined
a very useful subset of const generics which is stable enough to ship in the near term.
