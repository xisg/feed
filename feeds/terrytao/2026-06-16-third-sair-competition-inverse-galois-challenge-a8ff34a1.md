---
title: "Third SAIR competition: inverse Galois challenge"
url: https://terrytao.wordpress.com/2026/06/16/third-sair-competition-inverse-galois-challenge/
published: "2026-06-16T16:11:57Z"
feed: terrytao
guid: http://terrytao.wordpress.com/?p=17779
---

# Third SAIR competition: inverse Galois challenge

I am happy to announce the
[third SAIR challenge](https://competition.sair.foundation/competitions/igp24/overview),
which is focused on obtaining numerical data for the infamous
[inverse Galois problem](https://en.wikipedia.org/wiki/Inverse_Galois_problem).
This is a collaborative project with the
[L-functions and modular forms database](https://www.lmfdb.org/)
(LMFDB), and is organized by
[John Jones](https://hobbes.la.asu.edu/),
[Jen Paulhus](https://www.jenpaulhus.com/),
[David Roe](https://math.mit.edu/~roed/),
[Andrew Sutherland](https://math.mit.edu/~drew/), and myself. The
challenge is somewhat similar to my own
[Equational Theories Project](https://github.com/teorth/equational_theories),
in that one is trying to complete a large mathematical data set in
a verified fashion, except that the target data set had an
existing mathematical interest. Also, the verification will be
done by
[MAGMA](<https://en.wikipedia.org/wiki/Magma_(computer_algebra_system)>)
(as well as [PARI/GP](https://en.wikipedia.org/wiki/PARI/GP))
rather than Lean.

Let me first quickly review the inverse Galois problem. Suppose
one has an irreducible polynomial
![{P(z)}](https://s0.wp.com/latex.php?latex=%7BP%28z%29%7D&bg=ffffff&fg=000000&s=0&c=20201002)
of one variable of some degree
![{n}](https://s0.wp.com/latex.php?latex=%7Bn%7D&bg=ffffff&fg=000000&s=0&c=20201002)
and integer coefficients; take for instance
![{P(z) = z^3 - 3z + 1}](https://s0.wp.com/latex.php?latex=%7BP%28z%29+%3D+z%5E3+-+3z+%2B+1%7D&bg=ffffff&fg=000000&s=0&c=20201002).
Then
![{P}](https://s0.wp.com/latex.php?latex=%7BP%7D&bg=ffffff&fg=000000&s=0&c=20201002)
will have
![{n}](https://s0.wp.com/latex.php?latex=%7Bn%7D&bg=ffffff&fg=000000&s=0&c=20201002)
distinct roots
![{\alpha_1,\dots,\alpha_n}](https://s0.wp.com/latex.php?latex=%7B%5Calpha_1%2C%5Cdots%2C%5Calpha_n%7D&bg=ffffff&fg=000000&s=0&c=20201002);
in this case the roots happen to be

![\displaystyle \alpha_1 = 2 \cos \frac{2\pi}{9}, \quad \alpha_2 = 2 \cos \frac{8\pi}{9}, \quad \alpha_3 = 2 \cos \frac{14 \pi}{9}.](https://s0.wp.com/latex.php?latex=%5Cdisplaystyle++%5Calpha_1+%3D+2+%5Ccos+%5Cfrac%7B2%5Cpi%7D%7B9%7D%2C+%5Cquad+%5Calpha_2+%3D+2+%5Ccos+%5Cfrac%7B8%5Cpi%7D%7B9%7D%2C+%5Cquad+%5Calpha_3+%3D+2+%5Ccos+%5Cfrac%7B14+%5Cpi%7D%7B9%7D.&bg=ffffff&fg=000000&s=0&c=20201002)
The roots generate some degree
![{n}](https://s0.wp.com/latex.php?latex=%7Bn%7D&bg=ffffff&fg=000000&s=0&c=20201002)
extension
![{\mathbb{Q}(\alpha_1,\dots,\alpha_n)}](https://s0.wp.com/latex.php?latex=%7B%5Cmathbb%7BQ%7D%28%5Calpha_1%2C%5Cdots%2C%5Calpha_n%29%7D&bg=ffffff&fg=000000&s=0&c=20201002)
of the rational numbers
![{\mathbb{Q}}](https://s0.wp.com/latex.php?latex=%7B%5Cmathbb%7BQ%7D%7D&bg=ffffff&fg=000000&s=0&c=20201002).
Any automorphism of this field extension must permute the roots
![{\alpha_1,\dots,\alpha_n}](https://s0.wp.com/latex.php?latex=%7B%5Calpha_1%2C%5Cdots%2C%5Calpha_n%7D&bg=ffffff&fg=000000&s=0&c=20201002),
and thus generates a subgroup of the permutation group
![{S_n}](https://s0.wp.com/latex.php?latex=%7BS_n%7D&bg=ffffff&fg=000000&s=0&c=20201002)
(defined up to relabeling of the roots), which we call the _Galois
group_ of
![{P}](https://s0.wp.com/latex.php?latex=%7BP%7D&bg=ffffff&fg=000000&s=0&c=20201002).
This is some subgroup of
![{S_n}](https://s0.wp.com/latex.php?latex=%7BS_n%7D&bg=ffffff&fg=000000&s=0&c=20201002)
that acts transitively on the
![{n}](https://s0.wp.com/latex.php?latex=%7Bn%7D&bg=ffffff&fg=000000&s=0&c=20201002)
roots (because each root generates the field). Typically, it is
all of
![{S_n}](https://s0.wp.com/latex.php?latex=%7BS_n%7D&bg=ffffff&fg=000000&s=0&c=20201002);
but occasionally it is smaller. For example, the particular cubic
polynomial
![{P}](https://s0.wp.com/latex.php?latex=%7BP%7D&bg=ffffff&fg=000000&s=0&c=20201002)
above has the special property that each root
![{\alpha_1,\alpha_2,\alpha_3}](https://s0.wp.com/latex.php?latex=%7B%5Calpha_1%2C%5Calpha_2%2C%5Calpha_3%7D&bg=ffffff&fg=000000&s=0&c=20201002)
individually generates the entire field
![{\mathbb{Q}(\alpha_1,\alpha_2,\alpha_3)}](https://s0.wp.com/latex.php?latex=%7B%5Cmathbb%7BQ%7D%28%5Calpha_1%2C%5Calpha_2%2C%5Calpha_3%29%7D&bg=ffffff&fg=000000&s=0&c=20201002),
thanks to the identities

![\displaystyle \alpha_2 = \alpha_1^2 - 2; \quad \alpha_3 = \alpha_2^2 - 2; \quad \alpha_1 = \alpha_3^2 - 2.](https://s0.wp.com/latex.php?latex=%5Cdisplaystyle++%5Calpha_2+%3D+%5Calpha_1%5E2+-+2%3B+%5Cquad+%5Calpha_3+%3D+%5Calpha_2%5E2+-+2%3B+%5Cquad+%5Calpha_1+%3D+%5Calpha_3%5E2+-+2.&bg=ffffff&fg=000000&s=0&c=20201002)
Because of this, the Galois group of
![{P}](https://s0.wp.com/latex.php?latex=%7BP%7D&bg=ffffff&fg=000000&s=0&c=20201002)
is the cyclic group
![{C_3}](https://s0.wp.com/latex.php?latex=%7BC_3%7D&bg=ffffff&fg=000000&s=0&c=20201002)
(or equivalently, the alternating group
![{A_3}](https://s0.wp.com/latex.php?latex=%7BA_3%7D&bg=ffffff&fg=000000&s=0&c=20201002)),
rather than the full symmetric group
![{S_3}](https://s0.wp.com/latex.php?latex=%7BS_3%7D&bg=ffffff&fg=000000&s=0&c=20201002).
(This is in contrast to, say,
![{P(z) = z^3 - 2}](https://s0.wp.com/latex.php?latex=%7BP%28z%29+%3D+z%5E3+-+2%7D&bg=ffffff&fg=000000&s=0&c=20201002),
whose roots
![{\alpha_1 = 2^{1/3}}](https://s0.wp.com/latex.php?latex=%7B%5Calpha_1+%3D+2%5E%7B1%2F3%7D%7D&bg=ffffff&fg=000000&s=0&c=20201002),
![{\alpha_2 = 2^{1/3} \omega}](https://s0.wp.com/latex.php?latex=%7B%5Calpha_2+%3D+2%5E%7B1%2F3%7D+%5Comega%7D&bg=ffffff&fg=000000&s=0&c=20201002),
![{\alpha_3 = 2^{1/3} \omega^2}](https://s0.wp.com/latex.php?latex=%7B%5Calpha_3+%3D+2%5E%7B1%2F3%7D+%5Comega%5E2%7D&bg=ffffff&fg=000000&s=0&c=20201002)
cannot be expressed as rational polynomials of each other, and
whose Galois group is all of
![{S_3}](https://s0.wp.com/latex.php?latex=%7BS_3%7D&bg=ffffff&fg=000000&s=0&c=20201002).)
In fact, in the cubic case, it turns out that the Galois group is
![{A_3}](https://s0.wp.com/latex.php?latex=%7BA_3%7D&bg=ffffff&fg=000000&s=0&c=20201002)
when the
[discriminant](https://en.wikipedia.org/wiki/Discriminant) is a
perfect square, and
![{S_3}](https://s0.wp.com/latex.php?latex=%7BS_3%7D&bg=ffffff&fg=000000&s=0&c=20201002)
otherwise.

More generally, we have

> **Problem 1 (Inverse Galois Problem)** Let
> ![{G}](https://s0.wp.com/latex.php?latex=%7BG%7D&bg=ffffff&fg=000000&s=0&c=20201002)
> be a transitive permutation group on
> ![{n}](https://s0.wp.com/latex.php?latex=%7Bn%7D&bg=ffffff&fg=000000&s=0&c=20201002)
> letters. Can
> ![{G}](https://s0.wp.com/latex.php?latex=%7BG%7D&bg=ffffff&fg=000000&s=0&c=20201002)
> be realized as the Galois group of some degree
> ![{n}](https://s0.wp.com/latex.php?latex=%7Bn%7D&bg=ffffff&fg=000000&s=0&c=20201002)
> irreducible polynomial
> ![{P(z)}](https://s0.wp.com/latex.php?latex=%7BP%28z%29%7D&bg=ffffff&fg=000000&s=0&c=20201002)
> with integer coefficients (after identifying the
> ![{n}](https://s0.wp.com/latex.php?latex=%7Bn%7D&bg=ffffff&fg=000000&s=0&c=20201002)
> roots of
> ![{P}](https://s0.wp.com/latex.php?latex=%7BP%7D&bg=ffffff&fg=000000&s=0&c=20201002)
> suitably with the
> ![{n}](https://s0.wp.com/latex.php?latex=%7Bn%7D&bg=ffffff&fg=000000&s=0&c=20201002)
> letters)?

The answer to this problem is known to be positive for
![{n \leq 23}](https://s0.wp.com/latex.php?latex=%7Bn+%5Cleq+23%7D&bg=ffffff&fg=000000&s=0&c=20201002),
with the single possible exception of the sporadic
[Mathieu group](https://en.wikipedia.org/wiki/Mathieu_group)![{M_{23}}](https://s0.wp.com/latex.php?latex=%7BM_%7B23%7D%7D&bg=ffffff&fg=000000&s=0&c=20201002):
[there are ![{4953}](https://s0.wp.com/latex.php?latex=%7B4953%7D&bg=ffffff&fg=000000&s=0&c=20201002) transitive permutation groups](https://www.lmfdb.org/GaloisGroup/stats)
on
![{n \leq 23}](https://s0.wp.com/latex.php?latex=%7Bn+%5Cleq+23%7D&bg=ffffff&fg=000000&s=0&c=20201002)
letters (cf. [OEIS A002106](https://oeis.org/A002106)), and for
![{4952}](https://s0.wp.com/latex.php?latex=%7B4952%7D&bg=ffffff&fg=000000&s=0&c=20201002)
of them, a polynomial has been located with that Galois group; see
[this database](https://galoisdb.math.upb.de/) of Klüners and
Malle. The problem of locating a polynomial with Galois group
![{M_{23}}](https://s0.wp.com/latex.php?latex=%7BM_%7B23%7D%7D&bg=ffffff&fg=000000&s=0&c=20201002)
is a notorious open problem, though this is likely to be quite a
difficult problem, and _not_ the objective of the SAIR challenge.

Instead, we will focus on “breadth” rather than “depth”, in order
to leverage the power of crowdsourcing and modern AI technologies.
It turns out that
[there are ![{25000}](https://s0.wp.com/latex.php?latex=%7B25000%7D&bg=ffffff&fg=000000&s=0&c=20201002) distinct transitive permutation groups on ![{n=24}](https://s0.wp.com/latex.php?latex=%7Bn%3D24%7D&bg=ffffff&fg=000000&s=0&c=20201002) letters](https://www.lmfdb.org/GaloisGroup/?n=24),
which are conventionally labeled from
![{24T1}](https://s0.wp.com/latex.php?latex=%7B24T1%7D&bg=ffffff&fg=000000&s=0&c=20201002)
(the cyclic group
![{C_{24}}](https://s0.wp.com/latex.php?latex=%7BC_%7B24%7D%7D&bg=ffffff&fg=000000&s=0&c=20201002))
to
![{24T25000}](https://s0.wp.com/latex.php?latex=%7B24T25000%7D&bg=ffffff&fg=000000&s=0&c=20201002)
(the permutation group
![{S_{24}}](https://s0.wp.com/latex.php?latex=%7BS_%7B24%7D%7D&bg=ffffff&fg=000000&s=0&c=20201002)).
The first stage of the challenge will be:

> **Problem 2 (First stage of SAIR challenge)** For as many of the
> groups
> ![{24Tt}](https://s0.wp.com/latex.php?latex=%7B24Tt%7D&bg=ffffff&fg=000000&s=0&c=20201002),
> ![{t = 1,\dots,25000}](https://s0.wp.com/latex.php?latex=%7Bt+%3D+1%2C%5Cdots%2C25000%7D&bg=ffffff&fg=000000&s=0&c=20201002),
> locate an integer polynomial with that Galois group (up to
> isomorphism). (Also of interest is to specify the number of real
> roots, and to keep the discriminant low; more on this later.)

The verification side of this problem is essentially solved: the
MAGMA computer algebra system can take any candidate polynomial
and locate its Galois group within seconds. The MAGMA team has
kindly granted SAIR a limited license to
[provide an API](https://competition.sair.foundation/competitions/igp24/api)
for contestants to calculate a certain number of Galois groups per
day without needing to purchase their own license, though of
course they are free to use their other computational tools to
also perform these calculations outside of the competition.

The LMFDB already has polynomials for 286 of the 25000 groups, so
there is plenty of remaining polynomials to claim in the
challenge.

For applications, it is of interest to track some other statistics
of a polynomial besides its Galois group. One of these is the
number
![{r}](https://s0.wp.com/latex.php?latex=%7Br%7D&bg=ffffff&fg=000000&s=0&c=20201002)
of real roots, which is a number between
![{0}](https://s0.wp.com/latex.php?latex=%7B0%7D&bg=ffffff&fg=000000&s=0&c=20201002)
and
![{n}](https://s0.wp.com/latex.php?latex=%7Bn%7D&bg=ffffff&fg=000000&s=0&c=20201002)
of the same parity as
![{n}](https://s0.wp.com/latex.php?latex=%7Bn%7D&bg=ffffff&fg=000000&s=0&c=20201002)
(and which has to be achievable as the number of fixed points of
one of the permutations in the Galois group, namely the one
corresponding to complex conjugation); in particular, this number
must be even in the degree
![{24}](https://s0.wp.com/latex.php?latex=%7B24%7D&bg=ffffff&fg=000000&s=0&c=20201002)
case. Combining the label
![{t}](https://s0.wp.com/latex.php?latex=%7Bt%7D&bg=ffffff&fg=000000&s=0&c=20201002)
of the Galois group with the number of roots
![{r}](https://s0.wp.com/latex.php?latex=%7Br%7D&bg=ffffff&fg=000000&s=0&c=20201002)
turns out to generate
![{165836}](https://s0.wp.com/latex.php?latex=%7B165836%7D&bg=ffffff&fg=000000&s=0&c=20201002)![{(t,r)}](https://s0.wp.com/latex.php?latex=%7B%28t%2Cr%29%7D&bg=ffffff&fg=000000&s=0&c=20201002)
pairs in degree
![{24}](https://s0.wp.com/latex.php?latex=%7B24%7D&bg=ffffff&fg=000000&s=0&c=20201002),
and the challenge is actually to attach polynomials to as many of
these pairs as possible. (The LMFDB has already done so for just
![{622}](https://s0.wp.com/latex.php?latex=%7B622%7D&bg=ffffff&fg=000000&s=0&c=20201002)
of these.)

Of course, there are infinitely many polynomials of degree
![{24}](https://s0.wp.com/latex.php?latex=%7B24%7D&bg=ffffff&fg=000000&s=0&c=20201002),
and any Galois group that is representable by one polynomial, will
be representable by infinitely many others (e.g., one could simply
translate the polynomial by an arbitrary integer shift). To avoid
creating an unusable database filled with uninteresting
polynomials, we will prioritize polynomials whose (absolute)
discriminant is as small as possible. (There are some technical
details as to how this discriminant is defined and computed; see
[this page](https://competition.sair.foundation/competitions/igp24/evaluation-setup)
for details). The way we have set things up, each
![{(t,r)}](https://s0.wp.com/latex.php?latex=%7B%28t%2Cr%29%7D&bg=ffffff&fg=000000&s=0&c=20201002)
pair will come with a leaderboard for the polynomials with the
smallest discriminants that have been located so far by
contestants, removing duplicates arising from trivial operations
such as translating the polynomial. Contestant team will be
awarded a score between
![{0}](https://s0.wp.com/latex.php?latex=%7B0%7D&bg=ffffff&fg=000000&s=0&c=20201002)
and
![{1}](https://s0.wp.com/latex.php?latex=%7B1%7D&bg=ffffff&fg=000000&s=0&c=20201002)
for each submitted polynomial based on how small their
discriminant is compared to the best known discriminant, and how
many other teams were also able to find a polynomial with that
![{(t,r)}](https://s0.wp.com/latex.php?latex=%7B%28t%2Cr%29%7D&bg=ffffff&fg=000000&s=0&c=20201002)
pair. Thus, pairs that are extremely easy to generate (such as
those associated to the full permutation group
![{S_{24}}](https://s0.wp.com/latex.php?latex=%7BS_%7B24%7D%7D&bg=ffffff&fg=000000&s=0&c=20201002))
will be worth only a negligible score (as every contestant will be
able to submit a polynomial for that pair), while pairs which are
difficult to locate a polynomial for will be worth more points.

For this competition, the unrestricted use of any sort of
computational tool, including AI, to locate the polynomials, are
expressly permitted; this first stage of the competition is a
“black box” challenge where we are not directly interested in
obtaining insights as to _how_ the polynomials are located, but
the sole objective is to resolve as much of the inverse Galois
challenge as possible. As such, the notorious uninterpretability
of modern AI is not a concern for this stage. However, we will
encourage contestants to share techniques with each other in order
to cover more ground, through
[the Zulip channel for this challenge](https://zulip.sair.foundation/#narrow/channel/21-Inverse-Galois-Problem-.28IGP24.29/topic/Welcome.20to.20IGP24/with/2411).

This first stage of the competition will close on August 15. After
this, we will launch a second stage (with details to be
determined) to focus on some set of candidate Galois groups that
could not be resolved by the first stage. Here we envisage a more
collaborative, conceptual, and human-driven effort in which the
role of AI tools may be more secondary, and with more of a focus
on creating mathematically interesting results rather than simply
trying to saturate a given benchmark. Stay tuned for more details!
