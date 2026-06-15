---
title: FuturesUnordered and the order of futures
url: https://without.boats/blog/futures-unordered/
published: "2024-02-18T00:00:00Z"
feed: boats
guid: https://without.boats/blog/futures-unordered/
---

# FuturesUnordered and the order of futures

In my [previous post](https://without.boats/blog/let-futures-be-futures), I wrote about the distinction between “multi-task”
and “intra-task” concurrency in async Rust. I want to open this post by considering a common pattern
that users encounter, and how they might implement a solution using each technique.

Let’s call this “sub-tasking.” You have a unit of work that you need to perform, and you want to
divide that unit into many smaller units of work, each of which can be run concurrently. This is
intentionally extremely abstract: basically every program of any significance contains an instance
of this pattern at least once (often many times), and the best solution will depend on the kind of
work being done, how much work there is, the arity of concurrency, and so on.

- Using **multi-task concurrency**, each smaller of work would be its own task. The user would spawn
  each of these tasks onto an executor. The results of the task would be collected with a
  synchronization primitive like a [channel](https://tokio.rs/tokio/tutorial/channels), or the tasks would be awaited together with
  a [JoinSet](https://docs.rs/tokio/latest/tokio/task/struct.JoinSet.html).

- Using **intra-task concurrency**, each smaller unit will be a future run concurrently within the
  same task. The user would construct all of the futures and then use a concurrency primitive like
  [join!](https://docs.rs/tokio/latest/tokio/macro.join.html) or [select!](https://docs.rs/tokio/latest/tokio/macro.select.html) to combine them into a single future, depending on the exact
  access pattern.


Each of these approaches has its advantages and disadvantages. Spawning multiple tasks requires that
each task be `'static`, which means they cannot borrow data from their parent task. This is
often a very annoying limitation, not only because it might be costly to use shared ownership
(meaning `Arc` and possibly `Mutex`), but also because even if it isn’t going to be problematic
in this context to use shared ownership, the design of Rust will make it much more annoying to manage than borrowing.(I’d love to see this change! Cheap shared ownership constructs like `Arc` and `Rc` should have
non-affine semantics so you don’t have to call clone on them.)

When you join multiple futures, they _can_ borrow from state outside of them within the same task,
but as I wrote in the previous post, you can only join a static number of futures. Users that don’t
want to deal with shared ownership but have a dynamic number of sub-tasks they need to execute are
left searching for another solution. Enter [FuturesUnordered](https://docs.rs/futures/latest/futures/stream/struct.FuturesUnordered.html).

`FuturesUnordered` is an odd duck of an abstraction from the futures library, which represents a
collection of futures as a `Stream` (in std parlance, an `AsyncIterator`). This makes it a lot like
tokio’s `JoinSet` in surface appearance, but unlike `JoinSet` the futures you push into it are not
spawned separately onto the executor, but are polled as the `FuturesUnordered` is polled. Much like
spawning a task, every future pushed into `FuturesUnordered` is separately allocated, so
representationally its very similar to multi-task concurrency. But because the `FuturesUnordered` is
what polls each of these futures, they don’t execute independently and they don’t need to be
`'static`. They can borrow surrounding state as long as the `FuturesUnordered` doesn’t outlive that
state.

In a sense, `FuturesUnordered` is a sort of hybrid between intra-task concurrency and multi-task
concurrency: you can borrow state from the same task like intra-task, but you can execute
arbitrarily many concurrent futures like multi-task. So it seems like a natural fit for the use case
I was just describing when the user wants that exact combination of features. But `FuturesUnordered`
has also been a culprit in some of the more frustrating bugs that users encounter when writing async
Rust. In the rest of this post, I want to investigate the reasons why that is.
