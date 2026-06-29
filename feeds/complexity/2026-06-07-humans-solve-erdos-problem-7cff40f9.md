---
title: Humans Solve Erdos Problem!!
url: https://blog.computationalcomplexity.org/2026/06/humans-solve-erdos-problem.html
published: "2026-06-07T19:29:06Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-6135142595358018857
---

# Humans Solve Erdos Problem!!

(In 2008 I wrote a survey of some of the known sum-product
theorems,
see [here](https://www.cs.umd.edu/~gasarch/BLOGPAPERS/sp.pdf). Avi
Wigderson has a great slide-set on sum-product theorems and their
applications---the slides are on Avi's webpage of talks he has
given (all the talks are excellent) which
is [here](https://www.math.ias.edu/avi/talks). I had a prior post
on sum-product
theorems [here](https://blog.computationalcomplexity.org/2008/09/sum-product-theorems.html))

If \\(A\\) is a set then let

\\(A+A = \\{ x+y \ \\colon\\  x,y\\in A \\} \\),     \\(A\\cdot A
= \\{ xy \ \\colon \\  x,y\\in A \\} \\).

Let \\(A= \\{1,\\ldots,n\\} \\).

\\(\|A+A\| = \\Theta(n)  \\) which is small.

What about \\(\|A\\cdot A\|\\)?  By the prime number theorem there
are  \\( \\Theta(\\frac{n}{\\log n}) \\) primes in \\(A\\) hence
\\(\|A\\cdot A\| \\ge \\Omega(\\frac{n^2}{\\log^2n})\\) by taking
product of two primes.

Or better: look at  \\(\\{ xy \ \\colon \ x \\hbox{ a prime in }
\\{n/2,\\ldots,n\\} , y \\in \\{1,\\ldots,n/2\\} \\} \\).

This is a subset of  \\( A\\cdot A \\) of size  \\( \\Omega(
\\frac{n^2}{\\log n} ) \\).

Ford improved this to

\\\[ \|A\\cdot A\| = \\Theta\\biggl (\\frac{n^2}{(\\log
n)^a(\\log\\log n)^{3/2}} \\biggr ) \\\]

where  \\(a=1-\\frac{1+\\ln\\ln 2}{\\ln 2} \\sim 0.086\\). So
\\(\|A\\cdot A\|\\) is Large! (Ford's paper is
[here](https://arxiv.org/abs/math/0401223).)

Let \\(A= \\{2^1,\\ldots,2^n\\} \\).

\\(\|A+A\| = \\Theta(n^2) \\). Large!  \\(\|A\\cdot A\| =
\\Theta(n)  \\). Small!

Is it always the case that, for \\(A\\) a finite subset of
numbers, \\(\\max(\|A+A\|,\|A\\cdot A\|)\\) is large?

In 1976 Erdős made a series of conjectures:

For all \\(A\\subset N\\), A finite, \\(\\max(\|A+A\|,\|A\\cdot
A\|) \\ge \|A\|^{2-o(1)} \\).

For all \\(A\\subset Z\\), A finite,  \\(\\max(\|A+A\|,\|A\\cdot
A\|) \\ge  \|A\|^{2-o(1)}\\).

For all \\(A\\subset R\\), A finite,  \\(\\max(\|A+A\|,\|A\\cdot
A\|) \\ge \|A\|^{2-o(1) } \\).

For all  \\(A\\subset C\\), A finite, \\(\\max(\|A+A\|,\|A\\cdot
A\|) \\ge  \|A\|^{2-o(1) }\\).

Even though there are four of them (plural) these have come to be
called _The Sum-Product Conjecture_ (singular).

The paper appeared in the _Israel Journal of Math_ and is oddly
titled: *Problems and results on 3-progressions and related
topics.*(I could not find this paper online- if you can, email me
the pointer or pdf.)

In 1983 Erdős and Szemerédi made progress on the conjecture for
\\(Z\\) by showing the following two theorems (they combine it
into one).

1. There exists \\(c>0\\) such that, for all \\(n\\),  for all
   \\(A\\subset  Z \\), \\(\|A\|=n\\), \\( n^{1+c} <
   \\max\\{\|A+A\|,\|A\\cdot A\|\\}\\). The \\(c\\) was very
   small. In my survey I present a sequence of results where
   \\(c\\) gets bigger and bigger. More has been found since my
   survey, see the Wikipedia page on Sum-Product
   Theorems [here](https://en.wikipedia.org/wiki/Erd%C5%91s%E2%80%93Szemer%C3%A9di_theorem).

2. There exists \\(d>0\\) such that, for all \\(n\\), there exists
   \\(A\\subset Z\\), \\(\|A\|=n\\), such that 
   \\(\\max\\{\|A+A\|,\|A\\cdot A\|\\}  < n^2\\exp(\\frac{-c\\log
   n}{\\log\\log n})\\).

The paper appeared in a _Studies in Pure Mathematics_ volume in
memory of Paul Turán and is properly titled *On sums and products
of integers.* The paper
is [here](https://renyi.hu/~p_erdos/1983-18.pdf). For a sanity
check I worked out that this is an improvements on Ford's result,
so the set \\(A\\) in part 2 is better than \\(\\{1,\\ldots,n\\}
\\),
see [here](https://www.cs.umd.edu/~gasarch/BLOGPAPERS/fordves.pdf).

Because of the two papers, the conjecture is sometimes attributed
to Erdős and sometimes attributed to Erdős-Szemerédi.

Who should the conjecture be attributed to? It no longer matters
since humans  found a counterexample to the conjecture! In
particular Thomas Bloom, Will Sawin, Carl Schildkraut, and Dimitri
Zhelezov found a counterexample for the conjecture over \\(R\\)
(and hence also over \\(C\\)). Remarkably, they used a
thought-to-be-obsolete tool called _thinking._ Their paper
is [here](https://arxiv.org/abs/2605.28781)

The math world was shocked! An Erdős problem resolved by humans! 
One Abel prize winner was quoted as saying _We knew the day would
eventually come when humans could resolve Erdős_ *problems, but we
didn't know it would come this soon!*  Several math departments
now have plans for workshops on Human Alignment.

MY POINTS

1. The Sum-Product conjecture is a well known and interesting
   conjecture, so this is not some obscure problem invented to
   make humans look good.

2. Human-generated or Human-assisted? The announcement claims that
   it  was mostly human. I tend to agree since if it was done by
   AI they wouldn't hide it, they'd brag about it (see my
   post: [here](https://blog.computationalcomplexity.org/2025/10/if-you-use-ai-in-your-work-do-our-brag.html)).

3. AI may still be needed to clean up the proof. In the future, we
   will all use AI the way we currently use grad students,
   cleaning up what we do.

4. The final proof was readable. One concern about human proofs is
   that they would be unreadable and hard to verify. That was not
   the case here.

5. The ideas needed for the solution already existed; however:

a) The right combination was hard to find.

b) The relevant techniques used, algebraic number theory, are not
standard tools in this field (What field is the sum-product
conjecture in? If you have an opinion on this, leave a comment.
Future blog topic: what dictates what field a conjecture is in? a
theorem is in?)

c) It was widely believed that the Sum-Product conjecture was
true.

d) Contrast the difficulty of the following two statements

The Sum-Product Conjecture over \\(R\\) is false. This requires
finding a counterexample.

The Sum-Product Conjecture over \\(N\\) is true. If true this
would require a proof that covers all finite subsets of \\(N\\).

The first statement seems easier to prove.

It would be of interest to see if humans can prove the Sum-Product
Conjecture for \\(N\\).  Of course, it might not be true. In which
case it would be even more impressive to prove it.  My
undergraduates can not only prove \\(\\sqrt{2}\\) and
\\(\\sqrt{3}\\) are irrational but also that \\(\\sqrt{4}\\) is
irrational.

6. In the short term, this result and what it portends, is good:
   math problems we care about will be resolved with the help of
   humans, perhaps solely by humans.  But in the long run AI may
   lose the ability---or at least the patience---to do the proofs
   themselves.

See item 10-ONE for a counter thought.

7. Humans are good at combining known concepts.  Are they good at
   coming up with new ones?  Is AI?  The distinction between
   combining known ideas and coming up with new ideas is thin.

8. Two contrary lessons:

a) AI's should know many fields of mathematics so that they can
use ideas from one field in another, like the humans did.

b) AI should know some field of math really well so they may do
something new in it that current humans could not have done.

Whether the AI's choose to do (a) or (b) they should also spend
time attending seminars they do not understand, as humans do.

9. If  humans produce a new treatment for cancer that is better
   than what is known, we will not care that humans did it (though
   AI will check it).  Is mathematics similar? Do we care that a
   human did it?  Medicine values outcomes; mathematics also
   values understanding.

10. I suggest two futures:

ONE: While this human-generated (or human-assisted) result is
impressive, it will be a rare occurrence. This result was actually
a counterexample. The needed math was known. The result was
interesting. This is a perfect storm that we might not see again
for a while.

TWO: Paul Erdős lived in an earlier time before AI helped us do
math.  Without the help of AI (or even computers really) he made
conjectures, solved some of them with thinking, collaborated with
others (before email! before Zoom!) who also used thinking.  Does
the recent resolution of the sum-product conjecture mean we are
going back to that time? A time when people actually had to think?
For some that is a dream, for others a nightmare.
