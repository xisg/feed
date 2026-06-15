---
title: 'Async Methods I: generic associated types'
url: https://without.boats/blog/async-methods-i/
published: "2018-05-31T00:00:00Z"
feed: boats
guid: https://without.boats/blog/async-methods-i/
---

# Async Methods I: generic associated types

Async/await continues to move along swimmingly. We’ve accepted an RFC describing how the async/await syntax will work in Rust, and work is underway on implementing support for it in the compiler. We’re hopeful that users will be able to start experimenting with the syntax on nightly by early July.
The RFC for async/await didn’t address one important thing: async methods. It is very important for people defining libraries to be able to define traits that contain async functions, like this:
