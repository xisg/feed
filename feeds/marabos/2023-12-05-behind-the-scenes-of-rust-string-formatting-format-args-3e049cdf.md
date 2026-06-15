---
title: 'Behind the Scenes of Rust String Formatting: format_args!()'
url: https://blog.m-ou.se/format-args/
published: "2023-12-05T00:00:00Z"
feed: marabos
guid: https://blog.m-ou.se/format-args/
---

# Behind the Scenes of Rust String Formatting: format_args!()

The [`fmt::Arguments`](https://doc.rust-lang.org/stable/std/fmt/struct.Arguments.html)
type is one of my favorite types in the Rust standard library.
It’s not particularly amazing, but it is a great building block that is indirectly used in nearly every Rust program.
This type, together with the [`format_args!()` macro](https://doc.rust-lang.org/stable/std/macro.format_args.html),
is the power behind `print!()`, `format!()`, `log::info!()` and many more text formatting macros,
both from the standard library and community crates.

In this blog post, we learn how it works, how it is implemented today, and how that might change in the future.
