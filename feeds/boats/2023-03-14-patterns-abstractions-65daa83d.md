---
title: Patterns & Abstractions
url: https://without.boats/blog/patterns-and-abstractions/
published: "2023-03-14T00:00:00Z"
feed: boats
guid: https://without.boats/blog/patterns-and-abstractions/
---

# Patterns & Abstractions

This is the second post in an informal series commenting on the design of async Rust in 2023. In [my\
previous post](https://without.boats/blog/the-registers-of-rust), after a discussion of the “registers” in which control-flow effects could
be handled in Rust, I promised to turn my attention to recent proposals around a concept called
“keyword generics.” For reference, there are two posts by the current design team of Rust that are
my reference point for this commentary:

- **[Keyword Generics Progress Report: February 2023](https://blog.rust-lang.org/inside-rust/2023/02/23/keyword-generics-progress-report-feb-2023.html)**, a status update from the
  group working on “keyword generics”
- **[Trait transformers (send bounds, part 3)](https://smallcultfollowing.com/babysteps/blog/2023/03/03/trait-transformers-send-bounds-part-3/)**, a blog post about a related
  idea by Niko Matsakis

I’m not going to reiterate these blog posts at length, but in brief “keyword generics” is a proposal
to introduce a new kind of abstraction to Rust, to allow types to be abstracted over certain
effects. The examples have focused on `async` and `const` as the effects, but there is sometimes
discussion of `try` as well. Astute readers will notice this is an overlapping but not identical set
of effects to the effects I identified in my last post; I did not mention `const` as an effect, and
as far as I know the keyword generics working group has not devoted much or any time to considering
iteration as an effect.
