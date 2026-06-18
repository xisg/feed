---
title: Respect the P v NP Problem
url: https://blog.computationalcomplexity.org/2026/06/respect-p-v-np-problem.html
published: "2026-06-10T18:50:08Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-5415610689180652747
---

# Respect the P v NP Problem

There are two ways to look at the P v NP problem, as a formal
mathematically defined conjecture as a Clay Millennium Prize
Problem, and as the more intuitive notion that everything
efficiently verifiable is efficiently computable and the
implications that has on our ability to compute.

I've
[written considerably](https://cacm.acm.org/research/fifty-years-of-p-vs-np-and-the-possibility-of-the-impossible/)
about how artificial intelligence has affected the latter. In
particular, how AI and other advances in computing have brought us
to this
[Optiland](https://blog.computationalcomplexity.org/2020/12/optiland.html)
of getting most of the good implications of P=NP while our
cryptographic codes remain unbreakable.

But now with the recent advances in AI-created and assisted
proofs, will AI change what we know about the formal mathematical
statement? Is an AI-generated proof of P ≠ NP around the corner?

No, it isn't.  I do not believe we will see a P v NP proof in my
lifetime proven by man or machine, separately or working together.

While the disproof of the Erdős unit distance problem is an
impressive AI achievement, keep in mind that for every AI math
proof there are hundreds of problems that we have tried to solve
with AI where we haven't seen progress. And there is a huge chasm
between Erdős combinatorial conjectures and the Clay Millennium
problems. AI will continue to improve, but there are limits.

People, particularly those outside of computational complexity,
don't realize how difficult a mathematical challenge this is.
Polynomial-time algorithms can work in strange and mysterious
ways. They don't have to respect the semantics of an NP-search
problem or do any searching at all. Bill gave me the following
"algorithm" for clique: Take the eigenvalues of the adjacency
matrix. For all we know, if there are two primes _p_ and _q_ such
that the _p_ th eigenvalue and the _q_ th eigenvalue differ by
more than 1/ _k_ then the graph has a _k_-clique. Of course this
doesn't work. But to prove P ≠ NP, you need to prove not only that
this algorithm doesn't work, but neither do any of the infinitely
other potential algorithms for solving NP-complete problems.

We simply know of no way to manage general polynomial-time
algorithms other than by simulating them. We know by
relativization that simulation and diagonalization will not work
to settle P v NP. Other attempts to understand polynomial time,
like circuit complexity, proof complexity and algebraic geometry
have gotten bogged down well below the full power of
polynomial-time. At this time we don't even have a viable approach
to settling the P v NP problem.

Don't waste your time trying a formal approach via Lean. (I'm
looking at
you [Dmitry Khanukov](https://cacm.acm.org/blogcacm/i-spent-a-year-asking-ai-to-solve-maths-hardest-problem-heres-what-happened/))
Computational complexity is very messy to formulate technically. I
can't get an AI willing to give me a full Lean-verified proof of
something trivial like P closed under complement, forget the PCP
theorem. If someone or something does come up with a P ≠ NP, it'll
be following the right intuitive approach, not a formalistic one.

At least start with something simpler, like showing BPP is in
subexponential time, or SAT doesn't have quadratic algorithms. You
won't succeed there either, even though these questions should be
galactically simpler than P ≠ NP.
