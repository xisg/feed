---
title: Let futures be futures
url: https://without.boats/blog/let-futures-be-futures/
published: "2024-02-03T00:00:00Z"
feed: boats
guid: https://without.boats/blog/let-futures-be-futures/
---

# Let futures be futures

In the early-to-mid 2010s, there was a renaissance in languages exploring new ways of doing
concurrency. In the midst of this renaissance, one abstraction for achieving concurrent operations
that was developed was the “future” or “promise” abstraction, which represented a unit of work that
will maybe eventually complete, allowing the programmer to use this to manipulate control flow in
their program. Building on this, syntactic sugar called “async/await” was introduced to take futures
and shape them into the ordinary, linear control flow that is most common. This approach has been
adopted in many mainstream languages, a series of developments that has been controversial among
practitioners.

There are two excellent posts from that period which do a very good job of making the case for the
two sides of the argument. I couldn’t more strongly recommend reading each these posts in full:

- [Futures aren’t ersatz threads](https://monkey.org/~marius/futures-arent-ersatz-threads.html) by Marius Eriksen (April 2, 2013)
- [What color is your function?](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/) by Bob Nystrom (February 1, 2015)

The thesis of Eriksen’s post is that futures provide a fundamentally different model of concurrency
from threads. Threads provide a model in which all operations occur “synchronously,” because the
execution of the program is modeled as a stack of function calls, which block when they need to wait
for concurrently executing operations to complete. In contrast, by representing concurrent
operations as asynchronously completing “futures,” the futures model enabled several advantages
cited by Eriksen. These are the ones I find particularly compelling:

1. A function performing asynchronous operations has a different type from a “pure” function,
   because it must return a future instead of just a value. This distinction is useful because it
   lets you know if a function is performing IO or just pure computation, with far-reaching
   implications.
2. Because they create a direct representation of the unit of work to be performed, futures can be
   composed in multiple ways, both sequentially and concurrently. Blocking function calls can only
   be composed sequentially without starting a new thread.
3. Because futures can be composed concurrently, concurrent code can be written which more directly
   expresses the logic of what is occurring. Abstractions can be written which represent particular
   patterns of concurrency, allowing business logic to be lifted from the machinery of scheduling
   work across threads. Eriksen gives examples like a `flatMap` operator to chain many
   concurrent network requests after one initial network request.

Nystrom takes the counter-position. He starts by imagining a language in which all functions are
“colored,” either `BLUE`
or `RED`
. In his imaginary language, the important difference
between the two colors of function is that `RED`
functions can only be called from other `RED`
functions. He posits this distinction as a great frustration for users of the language,
because having to track two different kinds of functions is annoying and in his language `RED`
functions must be called using an annoyingly baroque syntax. Of course, what he’s referring to is
the difference between synchronous functions and asynchronous functions. Exactly what Eriksen cites
as an advantage of futures - that functions returning futures are different from functions that
don’t return futures - is for Nystrom it’s greatest weakness.

Some of the remarks Nystrom makes are not relevant to async Rust. For example, he says that if you
call a function of one color as if it were a function of the other, dreadful things could happen:

> When calling a function, you need to use the call that corresponds to its color. If you get it
> wrong … it does something bad. Dredge up some long-forgotten nightmare from your childhood like
> a clown with snakes for arms hiding under your bed. That jumps out of your monitor and sucks out
> your vitreous humour.

This is plausibly true of JavaScript, an untyped language with [famously ridiculous](https://www.destroyallsoftware.com/talks/wat) semantics,
but in a statically typed language like Rust, you’ll get a compiler error which you can fix and move
on.

One of his main points is also that calling a `RED`
function is much more “painful” than
calling a `BLUE`
function. As Nystrom later elaborates in his post, he is referring to the
callback-based API commonly used in JavaScript in 2015, and he says that async/await syntax resolves
this problem:

> \[Async/await\] lets you make asynchronous calls just as easily as you can synchronous ones, with the tiny
> addition of a cute little keyword. You can nest `await` calls in expressions, use them in
> exception handling code, stuff them inside control flow.

Of course, he also says this, which is the crux of the argument about the “function coloring
problem”:

> _But…_ you still have divided the world in two. Those async functions are easier to write, but
> _they’re still async functions._
>
> You’ve still got two colors. Async-await solves annoying rule #4: they make red functions not much
> worse to call than blue ones. But all of the other rules are still there.

Futures represent asynchronous operations differently from synchronous operations. For Eriksen, this
provides additional affordances which are the key advantage of futures. For Nystrom, this is just an
another hurdle to calling functions which return futures instead of blocking.

As you might expect if you’re familiar with this blog, I fall pretty firmly on the side of Eriksen.
So it has not been easy on me to find that Nystrom’s views have been much more popular with the sort
of people who comment on Hacker News or write angry, over-confident rants on the internet. A few
months ago I wrote a [post](https://without.boats/blog/why-async-rust) exploring the history of how Rust came to have the
futures abstraction and async/await syntax on top of that, as well as a follow-up
[post](https://without.boats/blog/a-four-year-plan) describing the features I would like to see added to async Rust to make it
easier to use.

Now I would like to take a step back and re-examine the design of async Rust in the context of this
question about the utility of the futures model of concurrency. What has the use of futures
actually gotten us in async Rust? I would like us to imagine that there could be a world in which
the difficulties of using futures have been mitigated or resolved & the additional affordances they
provide make async Rust not only just as easy to use as non-async Rust, but actually a _better_
experience overall.
