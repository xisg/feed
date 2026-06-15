---
title: Handshake Patterns
url: https://without.boats/blog/handshake-patterns/
published: "2017-01-21T00:00:00Z"
feed: boats
guid: https://without.boats/blog/handshake-patterns/
---

# Handshake Patterns

The problem: defining a ‘handshake’ protocol between two traits You have a problem that decomposes in this way: you want any type which implements trait Alpha to be composable with any type which implements trait Omega…
That is, if Foo and Bar are both Alphas and Baz and Quux are both Omegas, you can compose Foo with Baz or Quux, and the same with Bar, and so on.
This is not a trivial problem.
