---
title: Generators with UnpinCell
url: https://without.boats/blog/generators-with-unpin-cell/
published: "2024-10-25T00:00:00Z"
feed: boats
guid: https://without.boats/blog/generators-with-unpin-cell/
---

# Generators with UnpinCell

In July, I described a way to make pinning [more ergonomic](https://without.boats/blog/pinned-places) by integrating it more
fully into the language. Last week, I develoepd that idea [further](https://without.boats/blog/unpin-cell) with the notion of
`UnpinCell`: a wrapper type that lets a user take an `&pin mut UnpinCell<T>` and produce an `&mut T`, similar to how other cells let a user take a shared reference to the cell and produce a mutable
reference to its contents. I believe that this notion can also solve the biggest outstanding issues
facing [generators](https://without.boats/blog/generators): the fact that the `Iterator` interface does not permit
self-referential values.

As I wrote in my [explanation](https://without.boats/blog/pin) of Pin’s design, the biggest advantage that Pin had over other
design ideas was that it was a trivially backward compatible way of introducing a contract that an
object will never be moved. But this meant that a trait could only opt into that contract using the
new interface; traits that existed before Pin and don’t opt into that contract cannot be implemented
by types that have self-referential values. The most problematic trait here is `Iterator`, because
generators (functions that evaluate to iterators in the same way async functions evaluate to
futures) would ideally support self-referential values just like async functions do. So long as the
interface for `Iterator` takes a mutable reference and not a pinned mutable reference, implementers
must assume the iterator can be moved around and therefore can’t be self-referential.
