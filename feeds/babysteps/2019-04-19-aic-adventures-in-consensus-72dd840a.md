---
title: 'AiC: Adventures in consensus'
url: https://smallcultfollowing.com/babysteps/blog/2019/04/19/aic-adventures-in-consensus/?utm_source=atom_feed
published: "2019-04-19T00:00:00Z"
feed: babysteps
guid: https://smallcultfollowing.com/babysteps/blog/2019/04/19/aic-adventures-in-consensus/
---

# AiC: Adventures in consensus

In the talk I gave at Rust LATAM, [I said](https://nikomatsakis.github.io/rust-latam-2019/#92) that the Rust project has
always emphasized **finding the best solution, rather than winning the**
**argument**. I think this is one of our deepest values. It’s also one
of the hardest for us to uphold.

Let’s face it – when you’re having a conversation, it’s easy to get
attached to specific proposals. It’s easy to have those proposals
change from “Option A” vs “Option B” to “ **my** option” and “ **their**
option”. Once this happens, it can be very hard to let **them** “win”
– even if you know that both options are quite reasonable.

This is a problem I’ve been thinking a lot about lately. So I wanted
to start an irregular series of blog posts entitled “Adventures in
consensus”, or AiC for short. These posts are my way of exploring the
topic, and hopefully getting some feedback from all of you while I’m
at it.

This first post dives into what a phrase like “finding the best
solution” even means (is there a best?) as well as the mechanics of
how one might go about deciding if you really have the “best”
solution. Along the way, we’ll see a few places where I think our
current process could do better.

### Beyond tradeoffs

Part of the challenge here, of course, is that often there is no
“best” solution. Different solutions are better for different things.

This is the point where we often talk about _tradeoffs_ – and
tradeoffs are part of it. But I’m also wary of the term. It often
brings to mind a simplistic, zero-sum approach to the problem, where
we can all too easily decide that we have to pick A or B and leave it
at that.

But often when we are faced with two irreconcilable options, A or B,
there is a third one waiting in the wings. This third option often
turns on some hidden assumption that – once lifted – allows us to
find a better overall approach; one that satisfies _both_ A _and_ B.

### Example: the `?` operator

I think a good example is the `?` operator. When thinking about error
handling, we seem at first two face two irreconcilable options:

- **Explicit error codes**, like in C, make it easy to see where
  errors can occur, but they require tedious code at the call site of
  each function to check for errors, when most of the time you just
  want to propagate the error anyway. This seems to favor **explicit**
  **reasoning** at the expense of **the happy path**.

  - (In C specifically, there is also the problem of it being easy to
    forget to check for errors in the first place, but let’s leave
    that for now.)
- **Exceptions** propagate the error implicitly, making the happy path
  clean, but making it very hard to see where an error occurs.

By now, a number of languages have seen that there is a third way – a
kind of “explicit exception”, where you make it very easy and
lightweight to propagate errors In Rust, we do this via the `?`
operator (which desugars to a match). In Swift ( [if I understand\
correctly](https://docs.swift.org/swift-book/LanguageGuide/ErrorHandling.html)) invoking a method that throws an exception is done
by adding a prefix, like `try foo()`. Joe Duffy describes a similar
mechanism in the midori language in [his epic article dissecting\
Midori error handling](http://joeduffyblog.com/2016/02/07/the-error-model/).

Having used `?` for a long time now, I can definitely attest that (a)
it is very nice to be able to propagate errors in a light-weight
fasion and (b) having the explicit marker is very useful. Many times
I’ve found bugs by scrutinizing the code for `?`, uncovering
surprising control flow I wasn’t considering.

### There is no free lunch

Of course, I’d be remiss if I didn’t point out that the discussion
over `?` was a really difficult one for our community. It was [one of\
the longest RFC threads in\
history](https://github.com/rust-lang/rfcs/pull/243), and one in which
the same arguments seemed to rise up again and again in a kind of
cycle. Moreover, we’re _still_ wrestling with what extensions (if any)
we might want to consider to the basic mechanism (e.g., `try` blocks,
perhaps `try` fns, etc).

I think part of the reason for this is that “the third option ain’t
free”. In other words, the `?` operator did a nice job of sidestepping
the dichotomy that seemed to be presented by previous options (clear
but tedious vs elegant but hidden), but it did so by coming into
contact with other principles. In this case, the primary debate was
over whether to consider some mechanism like Haskell’s `do` syntax for
working with monads.[1](#fn:1)

I think this is generally true. All the examples that I can come up
with where we’ve found a third option generally come at _some_ sort of
price – but often it’s a price we’re content to pay. In the case of
`?`, this means that we have some syntax in the language that is
dedicated to errors, when _perhaps_ it could have been more general
(but that might itself have come with limitations, or meant more
complexity elsewhere).

### Rust’s origin story

Overcoming tradeoffs is, in my view, the **core purpose** of Rust.
After all, the ur-tradeoff of them all is **control vs safety**:

- Control – let the programmer decide about memory layout,
  threads, runtime.
- Safety – avoid crashes.

This choice used to be embodied by having to decide between using C++
(and gaining the control, and thus often performance) or a
garbage-collected language like Java (and sacrifice control, often at
the cost of performance). Deciding [whether or not to use\
threads](https://blog.rust-lang.org/2015/04/10/Fearless-Concurrency.html)
was a similar choice between peril and performance.

Ownership and borrowing eliminated that tradeoff – but not for free!
They come with a steep learning curve, after all, and they impose some
limitations of their own. (Flattening the slope of that learning curve
– and extending the range of patterns that we accept – was of course
a major goal of the [non-lexical lifetimes](https://rust-lang.github.io/rfcs/2094-nll.html) effort, and I think
will continue to be an area of focus for us going forward.)

### Tradeoffs after all – but the right ones

So, even though I maligned tradeoffs earlier as simplistic thinking,
perhaps in the end it _does_ all come down to tradeoffs. Rust is
definitely a language for people who prefer to [measure twice and cut\
once](https://en.wiktionary.org/wiki/measure_twice_and_cut_once), and – for such folks – learning ownership and borrowing
has proven to be worth the effort (and then some!). But this clearly
isn’t the right choice for all people and all situations.

I guess then that the trick is being sure that you’re _trading the_
_right things_. You will probably have to trade _something_, but it may
not be the things you’re discussing right now.

### Mapping the solution space

When we talk about the RFC process, we always emphasize that the point
of RFC discussion **is not to select the best answer**; rather, the
point is to **map the solution space**. That is, to explore what the
possible tradeoffs are and to really look for alternatives. This
mapping process also means exploring the ups and downs of the current
solutions on the table.

### What does mapping the solution space really mean?

When you look at it, “mapping the solution space” is actually a really
complex task. There are a lot of pieces to it:

- **Identifying stakeholders:** figuring out who are the people
  affected by this change, for good or ill.
- **Clarifying motivations:** what exactly are we aiming to solve with
  a given proposal? It’s interesting how often this is left unstated
  (and, I suspect, not fully understood). Often we have a general idea
  of the problem, but we could sharpen it quite a bit. It’s also very
  useful to figure out which parts of the problem are most important.
- **Finding the pros and cons of the current proposals:** what works
  well with each solution and what are its costs.
- **Identifying new possibilities:** finding new ways to solve the motivations.
  Sometimes this may not solve the _complete_ problem we set out to attack,
  but only the most important part – and that can be a good thing, if it avoids
  some of the downsides.
- **Finding the hidden assumption(s):** This is in some way the same as
  identifying new possibilities, but I thought it was worth pulling
  out separately. There often comes a point in the design where you
  feel like you are faced with two bad options – and then you realize
  that **one of the design constraints you took as inviolate isn’t,**
  **_really_, all that essential**. Once you weaken that constraint, or
  drop it entirely, suddenly the whole design falls into place.

### Our current process mixes all of these goals

Looking at that list of tasks, is it any wonder that some RFC threads
go wrong? The current process doesn’t really try to separate out these
various tasks in any way or even to really **highlight** them. We sort of
expect people to “figure it out” on their own.

Worse, I think the current process often _starts with a particular_
_solution_. This encourages people to react to _that solution_. The RFC
author, then, is naturally prone to be defensive and to defend their
proposal. We are right away kicking things off with an “A or B”
mindset, where ideas belong to people, rather than the process. I
think ‘disarming’ the attachment of people to specific ideas, and
instead trying to focus everyone’s attention on **the problem space as**
**a whole**, is crucial.

Now, I am not advocating for some kind of “waterfall” process
here. **I don’t think it’s possible to cleanly separate each of the**
**goals above and handle them one at a time.** It’s always a bit messy
– you start with a fuzzy idea of the problem (and some stakeholders)
and you try to refine it. Then you take a stab at what a solution
might look like, which helps you to understand better the problem
itself, but which also starts to bring in more stakeholders. Figuring
out the pros and cons may spark new ideas. And so forth.

But just because we can’t use waterfall doesn’t mean we can’t give
more structure. Exploring what that might mean is one of the things
I hope to do in subsequent blog posts.

### Conclusion

Ultimately, this post is about the importance of being **thorough and**
**deliberate** in our design efforts. If we truly want to find the best
design – well, I shouldn’t say the best design. If we want to find
the **right design for Rust**, it’s often going to take time. This is
because we need to take the time to elaborate on the implications of
the decisions we are making, and to give time for a “third way” to be
found.

But – lo – even here there is a tradeoff. We are trading away
**time**, it seems, for **optimality**. And this clearly isn’t always
the right choice. After all, [“real artists ship”](http://wiki.c2.com/?RealArtistsShip). Often, there
comes a point where further exploration yields increasingly small
improvements (“diminishing returns”).

As we explore ways to improve the design process, then, we should try
to ensure we are covering the whole design space, but we also have to
think about knowing when to stop and move on to the next thing.

### Oh, one last thing…

Also, by the by, if you’ve not already read aturon’s 3-part series on
“ [listening](http://aturon.github.io/2018/05/25/listening-part-1/) [and](http://aturon.github.io/2018/06/02/listening-part-2/) [trust](http://aturon.github.io/2018/06/18/listening-part-3/)”, you should do so.

### Feedback

I’ve created a discussion thread on [the internals forum](https://internals.rust-lang.org/t/aic-adventures-in-consensus/9843)
where you can leave questions or comments. I’ll definitely read them
and I will try to respond, though I often get overwhelmed[2](#fn:2), so
don’t feel offended if I fail to do so.

### Footnotes

* * *

1. If you’d like to read more about the `?` decision, this \[summary comment\] tried to cover the thread and lay out the reasoning behind the ultimate decision.
\[summary comment\]: [https://github.com/rust-lang/rfcs/pull/243#issuecomment-172057844](https://github.com/rust-lang/rfcs/pull/243#issuecomment-172057844) [↩︎](#fnref:1)

2. So many things, so little time. [↩︎](#fnref:2)
