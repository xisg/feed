---
title: 'Async & Await in Rust: a full proposal'
url: https://without.boats/blog/async-await-final/
published: "2018-04-06T00:00:00Z"
feed: boats
guid: https://without.boats/blog/async-await-final/
---

# Async & Await in Rust: a full proposal

I’m really excited to announce the culmination of much of our work over the
last four months: a pair of RFCs for supporting async & await notation in Rust.
This will be very impactful for Rust in the network services space. The change
is proposed as two RFCs:

- **[RFC #2394:](https://github.com/rust-lang/rfcs/pull/2394)** which adds async & await notation to the
  language.
- **[RFC #2395:](https://github.com/rust-lang/rfcs/pull/2395)** which moves a part of the futures library into
  std to support that syntax.

These RFCs will enable basic async & await syntax support with the full range
of Rust features - including borrowing across yield points. The rest of this
blog post just covers the answers to some anticipated frequently asked
questions; for more details see the two RFCs.
