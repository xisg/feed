---
title: ripgrep is faster than {grep, ag, git grep, ucg, pt, sift}
url: https://burntsushi.net/ripgrep/
published: "2016-09-23T07:05:00Z"
feed: burntsushi
guid: https://burntsushi.net/ripgrep/
---

# ripgrep is faster than {grep, ag, git grep, ucg, pt, sift}

In this article I will introduce a new command line search tool,
[`ripgrep`](https://github.com/BurntSushi/ripgrep),
that combines the usability of
[The Silver Searcher](https://github.com/ggreer/the_silver_searcher)
(an [`ack`](http://beyondgrep.com/) clone) with the
raw performance of GNU grep. `ripgrep` is fast, cross platform (with binaries
available for Linux, Mac and Windows) and written in
[Rust](https://www.rust-lang.org).

`ripgrep` is available on
[Github](https://github.com/BurntSushi/ripgrep).

We will attempt to do the impossible: a fair benchmark comparison between
several popular code search tools. Specifically, we will dive into a series of
25 benchmarks that substantiate the following claims:

- For both searching single files _and_ huge directories of files, no other
  tool obviously stands above `ripgrep` in either performance or correctness.
- `ripgrep` is the only tool with proper Unicode support that doesn’t make
  you pay dearly for it.
- Tools that search many files at once are generally _slower_ if they use
  memory maps, not faster.

As someone who has worked on text search in Rust in their free time for the
last 2.5 years, and as the author of both `ripgrep` and
[the underlying regular expression engine](https://github.com/rust-lang-nursery/regex),
I will use this opportunity to provide detailed insights into the performance
of each code search tool. No benchmark will go unscrutinized!

**Target audience**: Some familiarity with Unicode, programming and some
experience with working on the command line.

**NOTE**: I’m hearing reports from some people that `rg` isn’t as fast as I’ve
claimed on their data. I’d love to help explain what’s going on, but to do
that, I’ll need to be able to reproduce your results. If you
[file an issue](https://github.com/BurntSushi/ripgrep/issues)
with something I can reproduce, I’d be happy to try and explain it.
