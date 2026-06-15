---
title: Origins and developments
url: https://graydon2.dreamwidth.org/315410.html
published: "2025-01-02T22:15:04Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:315410
---

# Origins and developments

Today I was asked -- as I am frequently asked! -- about the origins of something in Rust. In this case the origin of the "borrow" _terminology_. Not the idea, just the word. Who first used that _word_?

The short answer to this is "as far as I know, John Boyland, in [his 1999 paper](https://citeseerx.ist.psu.edu/document?repid=rep1&type=pdf&doi=ab3f13a598a9f4b91eaf5920fba033d72424f230) Alias killing: Unique variables without destructive reads" (later expanded-on in a more widely cited 2000 paper where he renamed it to the slightly less dramatic "alias burying").

John's paper was read by and [influenced the Cyclone folks](https://www.cs.umd.edu/projects/PL/cyclone/scp.pdf) somewhere between 2002 and 2007. And both of these works were read by and influenced Niko Matsakis when [he proposed](https://github.com/rust-lang/rust-wiki-backup/blob/6f5fc4b1e3ac754b58a269af6dffaf157c18b688/Proposal-for-regions.md) upgrading Rust's existing and somewhat half-baked alias checker into something resembling today's region/lifetime/borrowing system (at the time this was motivated by soundness issues emerging from interactions with Rust's at-the-time half-baked mutability rules, creating the so-called "Dan's Bug").

This was not impossible to find -- I know a few extra places to look mind you -- but digging it up produced a couple surprises to me:

- I thought -- and nearly answered the person asking me -- that one of us in the Rust project had coined the term "borrow" as an ergonomic adaptation, because "region" and "alias" are clunky and nonintuitive. This was a retcon on my part! Easy to imagine, but wrong.

- Even more interesting to me is how many _other_ works there are in the Related Works section of John's paper! Not just things I've seen before\[1\] (eg. Cyclone, Tofte & Talpin's ML Kit, Ada, Euclid\[2\]) but also a spectrum of delightfully weird others: Eiffel\*, Islands, Balloons, FAP, ESC with its "virgin, pivot and plenary" references, etc. This reference-set pulls at threads happening in multiple teams/projects/institutions, all through the 90s, 80s, even back into the 1970s, eg. [here is Reynolds grappling with it in 1978](https://dl.acm.org/doi/pdf/10.1145/512760.512766).

Now, I don't want to minimize Niko's work here at all -- he's responsible for doing the lion's share of detailed design and implementation work on Rust's borrowing system in particular, along with many many other aspects of the language -- but I do think this is an illustrative example of something we should all keep in mind: little-if-anything intellectual happens in a vacuum, few-if-any ideas spring forth fully formed. And we're all curiously willing to retcon the origins of such things. Even things we ourselves participated in!

(I am reminded of similar -- much weightier -- arguments that I have read about whether Darwin invented the concept of Evolution, or Einstein of Relativity, or Planck of Quantum Mechanics. On close inspection the person who gets cited is always part of a lengthy and ongoing research program spanning decades, never just "inventing stuff in isolation". But we are all very inclined to imagine it as so!)

* * *

\[1\]: I have elsewhere written about precedents-I-knew-about when working on Rust on my own, and designing the original somewhat half-baked move-semantics and mutable-alias-control systems:

- [precedents for "uniqueness" / "move-semantics" I'm pretty sure I knew about](https://www.reddit.com/r/rust/comments/le7m54/is_it_fair_to_say_rust_parentage_can_be_traced_to/gmb4zgc/)

- [precedents for "mutable-aliasing control" I'm pretty sure I knew about](https://www.reddit.com/r/rust/comments/t9972l/did_rust_first_introduce_the_ownership_concept/hztbsni/)

\[2\]: A genuinely odd coincidence is that the earliest reference I can find to anyone grappling with mutable aliasing control in a PL _design_ \-\- not just a complaint about how it breaks Hoare\[3\] logic -- is [Euclid](https://en.wikipedia.org/wiki/Euclid_(programming_language)), which was a multi-site / multi-collaborator project including the one and only Butler Lampson, but was fairly substantially focused in Toronto (Horning, Holt, Cordy) and published in 1977. I was born in 1977 and grew up in Toronto, so somehow this all seems like, I don't now, something that was in the air?

\[3\]: No relation. I know, right? How can this be?

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=315410) comments
