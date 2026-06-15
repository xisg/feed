---
title: Don't ask if a monorepo is good for you – ask if you're good enough for a monorepo
url: https://yosefk.com/blog/dont-ask-if-a-monorepo-is-good-for-you-ask-if-youre-good-enough-for-a-monorepo.html
published: "2019-07-30T00:00:00Z"
feed: yosefk
---

# Don't ask if a monorepo is good for you – ask if you're good enough for a monorepo

This is inspired by [Dan Luu's post](https://danluu.com/monorepo/) on the advantages of a single big repository
over many small ones. That post is fairly old, and I confess that I'm hardly up to date on the state of tooling, both for
managing multiple repos and for dealing with one big one. But I'm going to make an argument which I think mostly works
regardless of the state of tooling on any given day:

- Monorepo is great if you're really good, but absolutely terrible if you're not that good.
- Multiple repos, on the other hand, are passable for everyone – they're never great, but they're never truly terrible,
  either.

If you agree with the above, the choice is up to your personal philosophy. To me, for instance, it's a no-brainer – I'll
choose the passable thing which successfully withstands contact with apathetic mediocrity over the greater thing which falls
apart upon such contact in a heartbeat.

You might be different – you might believe in Good – and then you'll choose a monorepo, like Google, the ultimate force for
Good in technology (which is why they safeguard your personal data; you wouldn't want someone evil to have it – luckily, Google
can do no evil.) And I'm almost not kidding: the superpower which lets Google maintain the grassroots bureaucracy which I find
necessary to make monorepos work well is actually the same trait making you sufficiently delusional to chant, or at least to
have chanted "Don't Be Evil" entirely seriously. I don't have that. I am, to a first approximation, evil. [Worse is Better](https://yosefk.com/blog/what-worse-is-better-vs-the-right-thing-is-really-about.html).

But that's me – I'm not saying _you/your org_ are Not So Good, or Evil. I'm only saying that _you should be open to_
_the possibility,_ and that I don't see the implications of being Not So Good discussed as much as they deserve.

Why are monorepos terrible if you're not that good? Three reasons:

1. Branching in
2. Modularity out
3. Tooling strained

Let's discuss them in some detail.

## Branching: getting forked by your worst programmer

In a Good team, you don't have multiple concurrent branches from which actual product deliveries are produced, and/or where
most people get to maintain these branches simultaneously for a long time. And you certainly can't have branching due to
outright atrocities, like someone adding a feature by killing a feature – for example, making the app work on Android, but
destroying the ability to build for iOS in the process.

But in a not-so-good team... you get the idea.

What do you do when you have a branch working on Android and another branch working on iOS and you have deliveries on both
platforms? You postpone the merge, and keep the fork. For how long do you postpone the merge? For as long as is necessary for
the dumbass who caused the fork to fix their handiwork, in parallel with delivering more features (which likely results in
digging a deeper hole to climb out of afterwards.) And the dumbass might take months, years, or forever.

The question then becomes, _what was forked_?

In a multi-repo world, the repo maintained by the team with the dumbass on it got forked. In a monorepo world, _the entire_
_code base got forked, and the entire org is now held hostage by the dumbass._ And you might think that this will result in a
lot of pressure to fix the problem, and you'd be wrong, for the same reasons that high murder rates don't cure themselves by
people putting pressure on whomever to lower them to some equilibrium level common to all human societies.

Some places have higher than average murder rates, and some places have have higher than average fork rates. And I argue that
a lot of places have fork rates which combine into a complete disaster with a monorepo. And you might not even realize how bad
the fork rate is at your place, because multiple repos largely shield you from the consequences. Or, more tragically, you might
not realize how bad your fork rate is because your monorepo is in its first couple of years, and you're sowing what you'll reap
in its next couple of years, when you'll have more code, more deliveries and more dumbasses.

With multiple repos, if _you_ have your shit under control, and _your_ repos have a single release branch with
a single timeline, all you have to do is to test against both of the dumbass's branches. But with a monorepo, you need to
maintain your code in 2 branches, with a growing share of everybody else's code morphing incompatibly in those branches, simply
because they exist. And very soon it will be more than 2 because there's more than a single dumbass, and good luck to you.

## Modularity: demoted from a norm to an ideal

Norms are mundane, but they are what is. Ideals are lofty, but they are merely what should be (and typically isn't.) If you
want to actually _have_ something, you don't want it to be an ideal, like altruism – you want it to be a norm, like
wiping one's ass. If something is demoted from ass-wiping to altruism, that something will scarcely be found in the wild.

With multiple repos, modularity is the norm. It's not a _must_ \- you technically can have a repo depending on umpteen
other repos. But your teammates _expect_ to be able to work with their repo with a minimal set of dependencies. They
_don't like_ to have to clone lots of other repos, and to then worry about their versions _(in part because the_
_tooling support for this might be less than great)_.

In fact, a common multi-repo failure mode is that people expect _too few_ dependencies and make _too many repos_
_which are too small_ to host a _useful_ self-contained system. Note that this failure mode is not lethal. It kinda
sucks to have this over-modularity with benefits of independence which turn out to be imaginary upon a closer look, and to have
people treat what essentially are internal APIs with way too much reverence, just because two modules which are extremely
tightly coupled _conceptually_ are independent _technically,_ in terms of cloning/building/testing. But it doesn't
kill you.

With a monorepo, modularity is a mere ideal. Everybody clones the whole thing. You're not supposed to add gratuitous
dependencies, but it's very easy to add such a dependency in terms of cloning, building and versioning, and nobody objects to
the dependency being added the way they would if they needed to clone more repos.

Of course in a Good team, needless dependencies would be weeded out in code reviews, and a Culture would evolve over time
avoiding needless dependencies. In a not-so-good team, your monorepo will grow into a single giant ball of circular
dependencies. Note that adding dependencies is infinitely easier than untangling them, much like forking is easier than merging,
with the difference that the gut-felt urgency to merge ("I can't maintain all your damned branches any longer!!") is far greater
and far more backed by simple self-interest than the urgency to improve the dependency structure.

## Tooling: is yours better than the standard?

This part might age worse than the others, and might not be particularly up to date even now – what "standard" tools are
capable of changes over time. But generally speaking, a growing monorepo is likely to outgrow the standard version management
tools and methods, as well as _other_ tools and methods dealing with your revision controlled code.

Google used to have a FUSE driver to avoid copying hundreds of millions of source lines at a time, and instead getting the
files on demand, when a directory is cd'd into. Facebook used to hack on hg to make it fast on its large monorepo. Maybe already
today, or some day, a growing number of off-the-shelf tools will scale to infinite monorepos without such investments. But it
sounds reasonable that there will always be tools and workflows which you will struggle to make work with a large monorepo
(starting with some script doing find/grep.)

With a bunch of small monorepos, you work with a small overall number of source files in your working directory, so you don't
need to tell your tools, "don't try to deal with the whole thing – instead only search this subset, or use this index etc. etc."
And you have tools these days which kinda sorta let you manage the revisions of multiple repositories (for instance, there's
Google's Repo.) And I think the result is very, very far from a great experience _potentially_ afforded by a large
monorepo. But it also _never_ breaks down as badly as a large monorepo outgrowing the abilities of tools, as well as the
ability of your local toolsmiths to find creative workarounds for these growth pains.

## Summary

Don't ask if a monorepo is good for you – ask if you're good enough for a monorepo. Personally, I don't have the guts to bet
on the supply of Goodness in a given org to remain sufficiently large over time to consistently avert the potential disasters of
monorepos. But that's just my personal outlook; if you want to compliment me, don't call me "smart," and definitely don't call
me "good" – I know my limits in these areas, and I take far more pride in knowing these limits than in the limits themselves;
so, to compliment me, call me "pragmatic." Yet a culture worthy of a monorepo absolutely can exist – just make sure yours
actually is one of those, and don't mistake your ideals for your norms.
