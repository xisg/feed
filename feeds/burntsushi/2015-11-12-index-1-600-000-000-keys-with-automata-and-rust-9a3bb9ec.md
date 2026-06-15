---
title: Index 1,600,000,000 Keys with Automata and Rust
url: https://burntsushi.net/transducers/
published: "2015-11-12T02:45:00Z"
feed: burntsushi
guid: https://burntsushi.net/transducers/
---

# Index 1,600,000,000 Keys with Automata and Rust

It turns out that finite state machines are useful for things other than
expressing computation. Finite state machines can also be used to compactly
represent ordered sets or maps of strings that can be searched very quickly.

In this article, I will teach you about finite state machines as a _data_
_structure_ for representing ordered sets and maps. This includes introducing
an implementation written in Rust called the
[`fst` crate](https://github.com/BurntSushi/fst).
It comes with
[complete API documentation](https://burntsushi.net/rustdoc/fst/).
I will also show you how to build them using a simple command line tool.
Finally, I will discuss a few experiments culminating in indexing over
1,600,000,000 URLs (134 GB) from the
[July 2015 Common Crawl Archive](http://blog.commoncrawl.org/2015/08/july-2015-crawl-archive-available/).

The technique presented in this article is also how
[Lucene represents a part of its inverted\
index](http://blog.mikemccandless.com/2010/12/using-finite-state-transducers-in.html).

Along the way, we will talk about memory maps, automaton intersection with
regular expressions, fuzzy searching with Levenshtein distance and streaming
set operations.

**Target audience**: Some familiarity with programming and fundamental data
structures. No experience with automata theory or Rust is required.
