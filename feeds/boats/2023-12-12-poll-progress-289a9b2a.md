---
title: poll_progress
url: https://without.boats/blog/poll-progress/
published: "2023-12-12T00:00:00Z"
feed: boats
guid: https://without.boats/blog/poll-progress/
---

# poll_progress

Last week, Tyler Mandry published an interesting [post](https://tmandry.gitlab.io/blog/posts/for-await-buffered-streams/) about a problem that the Rust
project calls “Barbara battles buffered streams.” Tyler does a good job explaining the issue, but
briefly the problem is that the buffering adapters from the futures library ( `Buffered` and
`BufferUnordered`) do not interact well with `for await` if the processing in the body is
asynchronous (i.e. if it contains any `await` expressions).

I think we can better understand the problem if we examine it visually. First, let’s consider the
control flow that occurs when a user processes a normal, non-asynchronous `Iterator` using a for
loop:

```fallback
 ┌── SOME ────────────────┐
 ╔═══════════════╗ ╔═══════▼═══════╗
 ║ ║▐▌ ║ ║▐▌
 ──────▶ NEXT ║▐▌ ║ LOOP BODY ║▐▌
 ║ ║▐▌ ║ ║▐▌
 ╚════════════▲══╝▐▌ ╚═══════════════╝▐▌
 ▀▀│▀▀▀▀▀▀▀▀▀│▀▀▀▀▘ ▀▀▀▀▀▀▀│▀▀▀▀▀▀▀▀▀▘
 │ └───────────────────┘
 └── NONE ──────────────────────────────▶

```

The for loop first calls the iterator’s `next` method, and then passes the resulting item (if there
is one) to the loop body. When there are no more items, it exits the loop.
