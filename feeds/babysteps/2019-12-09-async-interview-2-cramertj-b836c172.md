---
title: 'Async Interview #2: cramertj'
url: https://smallcultfollowing.com/babysteps/blog/2019/12/09/async-interview-2-cramertj/?utm_source=atom_feed
published: "2019-12-09T00:00:00Z"
feed: babysteps
guid: https://smallcultfollowing.com/babysteps/blog/2019/12/09/async-interview-2-cramertj/
---

# Async Interview #2: cramertj

For the second [async interview](http://smallcultfollowing.com/babysteps/blog/2019/12/09/async-interview-2-cramertj/), I spoke with Taylor Cramer – or
cramertj, as I’ll refer to him. cramertj is a member of the compiler
and lang teams and was – until recently – working on Fuchsia at
Google. He’s been a key player in Rust’s Async I/O design and in the
discussions around it. He was also responsible for a lot of the
implementation work to make `async fn` a reality.

### Video

You can watch the [video](https://youtu.be/NF_qyiypnOs) on YouTube. I’ve also embedded a copy here
for your convenience:

### Spreading this out over a few posts

So, cramertj and I had a long conversation, with a lot of technical
detail. I was trying to get this blog post finished by last Friday but
it took a lot of time! I decided it’s probably too much material to
post in one go, so I’m going to break up the blog post into a few
pieces (I’ll post the whole video though).

The blog post is mostly covering what cramertj had to say, though in
some cases I’m also adding in various bits of background information
or my own editorialization. I’m trying to mark it when I do that. =)

### On Fuchsia

We kicked off the discussion talking a bit about the particulars of
the Fuchsia project. Fuchsia is a microkernel architecture and thus a
lot of the services one finds in a typical kernel are implemented as
independent Fuchsia processes. These processes are implemented in Rust
and use Async I/O.

### Fuchsia uses its own unique executor and runtime

Because Fuchsia is not a unix system, its kernel primitives, like
sockets and events, work quite differently. Fuchsia therefore uses its
own custom executor and runtime, rather than building on a separate
stack like tokio or async-std.

### Fuchsia benefits from interoperability

Even though Fuchsia uses its own executor, it is able to reuse a lot
of libraries from the ecosystem. For example, Fuchsia uses Hyper for
its HTTP parsing. This is possible because Hyper offers a generic
interface based on traits that Fuchsia can implement.

In general, cramertj feels that the best way to achieve interop is to
offer trait-based interfaces. There are other projects, for example,
that offer feature flags (e.g., to enable “tokio” compatibilty etc),
but this tends to be a suboptimal way of managing things, at least for
libraries.

For one thing, offer features means that support for systems like fuschia
must be “upstreamed” into the project, whereas offering traits means that
downsteam systems can implement the traits themselves.

In addition, using features to choose between alternatives can cause
problems across larger dependency graphs. Features are always meant to
be “additive” – i.e,. you can add any number of them – but features
that choose between backends tend to be exclusive – i.e., you must
choose at most one. This is a problem because cargo likes to take the
union of all features across a dependency graph, and so having
exclusive features can lead to miscompilations when things are
combined.

### Background topic: futures crate

cramertj and I next talked some about the futures crate. Before going
much further into that, I want to give a bit of background on the
futures crate itself and how its setup.

The futures crate has been very carefully setup to permit its
components to evolve with minimal breakage and incompatibility across
the ecosystem. However, my experience from talking to people has been
that there is a lot of confusion as to how the futures crate is setup
and why, and just how much they can rely on things not to change. So I
want to spend a bit of time documenting _my_ understanding the setup
and its motivations.

Historically, the [`futures`](https://github.com/rust-lang-nursery/futures-rs/) crate has served as a kind of
experimental “proving ground” for various aspects of the future
design, including the `Future` trait itself (which is now in std).

Currently, the futures crate is at version 0.3, and it offers a number of
different categories of functionality:

- key traits like [`Stream`](https://docs.rs/futures-core/0.3.1/futures_core/stream/trait.Stream.html), [`AsyncRead`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncRead.html), and [`AsyncWrite`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncWrite.html)
- key primitives like \[“async-aware” locks\]
  - traditional locks
- “extension” traits like [`FutureExt`](https://docs.rs/futures/0.3.1/futures/future/trait.FutureExt.html), [`StreamExt`](https://docs.rs/futures/0.3.1/futures/stream/trait.StreamExt.html), [`AsyncReadExt`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncReadExt.html), and so forth

  - these traits offer convenient combinator methods like `map` that
    are not part of the corresponding base traits
- useful macros like [`join!`](https://docs.rs/futures/0.3.1/futures/macro.join.html) or [`select!`](https://docs.rs/futures/0.3.1/futures/macro.select.html)
- useful bits of code such as a [`ThreadPool`](https://docs.rs/futures/0.3.1/futures/executor/struct.ThreadPool.html) for “off-loading” heavy computations

In fact, the first item in that list (“key traits”) is quite distinct
from the remaining items. In particular, if you are writing a library,
those key traits are things that you might well like to have in your
public interface. For example, if you are writing a parser that
operates on a stream of data, it might take a [`AsyncRead`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncRead.html) as its
data source (just as a synchronous parser would take a [`Read`](https://doc.rust-lang.org/std/io/trait.Read.html)).

The remaining items on the list fall _generally_ into the category of
“implementation details”. They ought to be “private” dependencies of
your crate. For example, you may use methods from [`FutureExt`](https://docs.rs/futures/0.3.1/futures/future/trait.FutureExt.html)
internally, but you don’t require other crates to use them; similarly
you may [`join!`](https://docs.rs/futures/0.3.1/futures/macro.join.html) futures internally, but that is not something that
would show up in a function signature.

### the futures crate is really a facade

One thing you’ll notice if you look more closely at the [`futures`](https://github.com/rust-lang-nursery/futures-rs/)
crate is that it is in fact composed of a number of smaller crates.
The [`futures`](https://github.com/rust-lang-nursery/futures-rs/) crate itself simply ’re-exports’ items from these
other crates:

- [`futures-core`](https://crates.io/crates/futures-core) – defines the [`Stream`](https://docs.rs/futures-core/0.3.1/futures_core/stream/trait.Stream.html) trait (also the `Future`
  trait, but that is an alias for std)
- [`futures-io`](https://crates.io/crates/futures-io) – defines the [`AsyncRead`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncRead.html) and [`AsyncWrite`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncWrite.html) traits
- [`futures-util`](https://crates.io/crates/futures-util) – defines extension traits like [`FutureExt`](https://docs.rs/futures/0.3.1/futures/future/trait.FutureExt.html)
- …

The goal of this facade is to permit things to evolve without forcing
semver-incompatible changes. For example, if the [`AsyncRead`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncRead.html) trait
should evolve, we might be forced to issue a new major version of
[`futures-io`](https://crates.io/crates/futures-io) and thus ultimately issue a new [`futures`](https://github.com/rust-lang-nursery/futures-rs/) release
(say, 0.4). However, the version number of [`futures-core`](https://crates.io/crates/futures-core) remains
unchanged. This means that if your crate only depends on the
[`Stream`](https://docs.rs/futures-core/0.3.1/futures_core/stream/trait.Stream.html) trait, it will be interoperable across both [`futures`](https://github.com/rust-lang-nursery/futures-rs/) 0.3
and 0.4, since both of those versions are in fact re-exporting the
same [`Stream`](https://docs.rs/futures-core/0.3.1/futures_core/stream/trait.Stream.html) trait (from [`futures-core`](https://crates.io/crates/futures-core), whose version has not
changed).

In fact, if you are a library crate, it probably behooves you to avoid
depending on the [`futures`](https://github.com/rust-lang-nursery/futures-rs/) crate at all, and instead to declare
finer-grained dependencies; this will make it very clear whe you need
to declare a new semver release yourself.

### cramertj: the best place for “standard” traits is in std

So, background aside, let me return to my discussion with
cramertj. One of the points that cramertj is that the only “truly
standard” place for a trait to live is libstd. Therefore, cramertj
feels like the next logical step for traits like [`Stream`](https://docs.rs/futures-core/0.3.1/futures_core/stream/trait.Stream.html) or
[`AsyncRead`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncRead.html) is to start moving them into the standard library. Once
they are there, this would be the strongest possible signal that
people can rely on them not to change.

### we can move to libstd without breakage

You may be wondering what it would mean if we moved one of the traits
from the [`futures`](https://github.com/rust-lang-nursery/futures-rs/) crate into libstd – would things in the
ecosystem that are currently using [`futures`](https://github.com/rust-lang-nursery/futures-rs/) have to update? The
answer is no, not necessarily.

Presuming that some trait from [`futures`](https://github.com/rust-lang-nursery/futures-rs/) is moved wholesale into
libstd (i.e., without _any_ modification), then it is possible for us
to simply issue a new _minor version_ of the [`futures`](https://github.com/rust-lang-nursery/futures-rs/) crate (and
the appropriate subcrate). This new minor version would change from
defining a trait (say, `Stream`) to re-exporting the version from std.

As a concrete example, if we moved [`AsyncRead`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncRead.html) from [`futures-io`](https://crates.io/crates/futures-io)
to libstd (as cramertj advocates for later on), then we would issue a
`0.3.2` release of [`futures-io`](https://crates.io/crates/futures-io). This release would replace `trait AsyncRead` with a `pub use` that re-exports [`AsyncRead`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncRead.html) from std. Now,
any crate in the ecosystem that previously depended on `0.3.1` can be
transparently upgraded to `0.3.2` (it’s a semver-compatibly change,
after all)[1](#fn:1), and suddenly all references to
[`AsyncRead`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncRead.html) would be referencing the version from std. (This is, in
fact, exactly what happened with the futures trait; in 0.3.1., it is
simply [re-exported from libcore](https://docs.rs/futures-core/0.3.1/src/futures_core/future.rs.html#7).)

### on the extension traits

One of the interesting points that cramertj made, though not until
later in the interview, is that when it comes to futures there are a
number of “smaller design decisions” one might make when it comes to
combinators. For example, consider a function like [`Stream::filter`](https://docs.rs/futures/0.3.1/futures/stream/trait.StreamExt.html#method.filter).
As defined in the future crates, this function returns a “future to a
boolean”, so it has a signature like:

```rust
impl FnMut(&Item) -> impl Future<Output = bool>

```

This is effectively an async closure; I’ll summarize what cramertj had
to say about async closures in one of the upcoming blog
posts. However, you might plausibly wish instead to have a signature
that just returns a boolean directly, like so:

```rust
impl FnMut(&Item) -> bool

```

For this reason, cramertj felt that it may make sense not to add these
sorts of utilities into the standard library (or at least not yet),
and instead to leave those extension traits in “user space”. Maybe when we have
more experience we’ll be able to say what the best definition would be for
the standard library.

(If I may editorialize, I do think it’s important that we add these
sorts of helper methods to std eventually; even if there’s no single
best choice, we should make some decisions, because it’ll be quite
annoying to force everything to pull in utility crates for simple
things.)

### upcoming posts

OK, that wraps it up for the first post. I have two more coming. In
the next post, we’ll discuss the design of the [`Stream`](https://docs.rs/futures-core/0.3.1/futures_core/stream/trait.Stream.html), [`AsyncRead`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncRead.html),
and [`AsyncWrite`](https://docs.rs/futures/0.3.1/futures/io/trait.AsyncWrite.html) traits, and what we might want to change there. In
the final post, we’ll discuss async closures.

## Comments?

There is a [thread on the Rust users forum](https://users.rust-lang.org/t/async-interviews/35167/) for this series.

* * *

1. This change relies on the fact that cargo will generally not compile two distinct minor versions of a crate; so all crates that depend on `0.3.1` would be compiled against `0.3.2`. [↩︎](#fnref:1)
