---
title: "The Music of Streams"
url: https://www.cs.dartmouth.edu/~doug/music.pdf
published: "2001-01-01T00:00:00Z"
feed: mcilroy
guid: https://www.cs.dartmouth.edu/~doug/music.pdf
---

# The Music of Streams

The Music of Streams

M. Douglas McIlroy

Dartmouth College, Hanover, New Hampshire 03755, USA

Abstract

Data streams make preeminent instruments for playing certain classical themes from analysis. Complex networks of processes, eﬀortlessly orchestrated by lazy evaluation, can enumerate terms of formal power series ad inﬁnitum. Expressed in a language like Haskell, working programs for power-series operations are tiny gems, because the natural programming style for data streams ﬁts the mathematics so well—often better than time-honored summation notation.

The cleverest copyist is the one whose music is performed with the most ease without the performer guessing why. — Jean Jacques Rousseau, Dictionary of Music

1 Overture

Like persistent folk tunes, the themes I intend to play here have been arranged for various ensembles over many years. Some performances have been angular, and some melodic, but all share the staying power of good music. The themes stick in mind, to be enjoyed again and again as each performance exposes new surfaces and depths.

My subject is “power-stream compositions,” a phrase intended to suggest a discipline of remarkable expressiveness and style, as well as compositionality. Algorithms on power streams are succinct, beautiful, and inevitable in their ﬁtness for the purpose. As motivating examples for stream-based computing, power streams are compelling in the same way that quicksort is for recursion.

I shall show power-stream compositions in two styles, with the unifying motif of lazy lists. Power series are represented as sequences of coeﬃcients. Operators on series acquire values in order from their inputs, compute result values in order, and deliver them as output. Input is never received ahead of the needs of the output. Such operators are obviously compositional.

Preprint submitted to Elsevier Preprint 2 July 2000

In the ﬁrst style, Horner form, a power series

f(z) = f0 + f1z + f2 z2 + f3 z3 + · · ·

= f0 + z(f1 + z(f2 + z(f3 + ...))) is viewed recursively as a head term f0 and a tail power series f that satisfy f(z) = f0 + zf(z). The series f is represented by the list [f0, f1, f2, f3, · · · ].

In the second style, Maclaurin form, a power series

f(z) = f(0) + f′(0)z + f′′(0)z2/2! + f(3)(0)z3/3! + · · · is viewed as a head term f(0) and a tail series, f′, of the same form as f. The tail is the derivative of f and satisﬁes

f(z) = f(0) + R z 0 f′(t)dt. The series f is represented by the list [f(0), f′(0), f′′(0), f(3)(0), · · · ].

The list [1, 1, 1, 1, · · · ] interpreted in Horner form is the geometric series 1 + z+z2+z3+· · ·. The same list interpreted in Maclaurin form is the exponential series 1 + z + z2/2! + z3/3! + · · ·.

The lazy-list formulation relegates concerns about sequencing and storage management to an evaluating engine. For example, in programming the usual convolution formula for the product of power series,

ˇ( P∞ i=0 fizi) × ( P∞ i=0 gizi) = P∞ i=0 zi Pj=i j=0 fjgi−j, one must keep book on a tangle of subscripts and on ever-growing arrays of input coeﬃcients. In the lazy list realization of Horner form, the program becomes a smooth one-liner: (f0 : f) × g@(g0 : g) = f0g0 : f0 × g + g × f The operators (:) and (@), taken from Haskell (Peterson and Hammond 1997), denote the list constructor and the “as” operator: g@(g0 : g) means operand g has structure (g0 : g).

1.1 Conventions

Polynomials may be represented as ﬁnite power series.

Operations on power series are deﬁned by rules that tell how to compute the head of the result and how recursively to compute the tail. In applying a rule, the ﬁrst subrule whose patterns match the operands is used. The anonymous identiﬁer names unused quantities. When a scalar operand occurs where a list is expected, the scalar is converted to a one-element list of the expected type. Thus, when f is a list of integers, 1 + f becomes [1] + f.

The rules are formal. There is no concern with analytic convergence or com-

putational divergence. Nor is there a guarantee that results will be uniquely represented; any result may end in an arbitrary sequence of zeros. However, any result given by the rules will agree with the analytically correct result when such exists.

2 First theme: Horner form

A series f(z) = f0 + f1z + f2 z2 + · · · may be viewed as a head f0 and a tail series f, so that

f(z) = f0 + z f(z), with the list representation

f = f0 : f.

2.1 Arithmetic

Negative. −[ ] = [ ]

−(f0 : f) = −f0 : −f

Sum. f + [ ] = f

[ ] + f = f

(f0 : f) + (g0 : g) = f0+g0 : f +g

Product. [ ] × = [ ]

× [ ] = [ ]

(f0 : f) × g@(g0 : g) = f0g0 : f0 × g + f × g

Quotient. [ ] / [ ] = error

[ ] / (0 : ) = error

[ ] / = [ ]

(0 : f) / (0 : g) = f / g

(f0 : f) / g@(g0 : g) = let q0 = f0/g0 in q0 : (f −q0 × g)/g To verify the product rule, write the series in head-tail style, then expand the product into a form that can be read in head-tail style:

f × g = (f0 + zf) × (g0 + zg) = f0g0 + z(f0 × g + f × g).

The last quotient rule expresses long division; it may be checked by multiplying through by g.

Diﬀerence and nonnegative integer power may be deﬁned in the usual way in terms of negative, sum and product.

2.2 Composition and reversion

Expanding the composition f ◦g of power series f and g in head-tail style gives

f(g) = f0 + g × f(g)

= f + (g0 + zg) × f(g)

= (f0 + g0f(g)) + zg × f(g). In general, the the head element of the output list cannot be calculated in ﬁnite time: unfolding the parenthesized subexpression leads to an inﬁnite sum for the head element. We can proceed, however, when g0 = 0 (second rule below), or when f is a polynomial (third rule).

Composition. [ ] ◦ = [ ]

(f0 : f) ◦g@(0 : g) = f0 : g × (f ◦g)

(f0 : f) ◦g@(g0 : g) = (f0 + g0f(g)) + (0 : g × f(g))

Reversion. revert (0 : f) = r where r = 0 : 1/(f ◦r) The latter rule, for reversion (functional inversion) of power series, may be developed by expanding the identity f(r(z)) = z, where r(z) is the inverse series and f(0) = 0.

The reversion rule is a tribute to the lucidity of power streams. Reversion is suﬃciently tricky to have inspired a considerable literature, rife with subscripts. One of the more concise treatments (Knuth, 1969) takes half a page of pseudocode to describe an algorithm equivalent to this working one-liner. The restriction to operands with head term zero may be relaxed for polynomial operands, as it was for composition.

2.3 Calculus

Let D be the diﬀerentiation operator. Since D(f0 + zf) = f + z(Df), we have

Derivative (slow). D( : f) = f + (0 : Df)

D = [ ] This formulation of the derivative takes O(n2) coeﬃcient-domain additions to compute n terms of the result. Trading away elegance, we may obtain an O(n) algorithm by counting terms and using the fact that Dzn = nzn−1. The

integral, deﬁned by R z 0 f(t)dt, may also be calculated by counting terms. (I do not know of a “slow integral” rule that hides the counting.)

Fast derivative. D[ ] = [ ]

D( : f) = (h 1 f) where

h [ ] = [ ]

h n (g0 : g) = ng0 : (h (n + 1) g)

Fast integral. R [ ] = [ ] R f = 0 : (h 1 f) where

h [ ] = [ ]

h n (g0 : g) = g0/n : (h (n + 1) g)

Example 1, elementary functions. Sine, cosine and exponential may be deﬁned as solutions of initial value problems,

D sin(x) = cos(x), sin(0) = 0

D cos(x) = −sin(x), cos(0) = 1

D exp(x) = exp(x), exp(0) = 1 Integration converts the diﬀerential equations to a neat program:

sin = R cos

cos = 1 − R sin

exp = 1 + R exp

Example 2, fast exponential. Calculation of n terms of exp(f(z)), where f(0) = 0, by the program exp ◦f takes O(n3) coeﬃcient-domain operations (Knuth, 1969). Gilles Kahn exploited the fact that exp(f) is a solution of y′ = yf′ with y(0) = 1 to get an O(n2) method:

exp ◦f = y where y = 1 + R (y × Df).

Example 3, square root. A power series f with constant term 1 has a square root q, of the same form, that satisﬁes q2 = f. Hence

2qDq = Df

Dq = Df/2q

q = 1 + R Df/2q from which follows the program

sqrt f@(1 : ) = q where q = 1 + R (Df/(2q)).

Example 4, tough tests. Classical identities provide a trove of tests of powerseries code. A simple test that exercises every basic operation is to check an initial segment of the series T deﬁned by

t = [0, 1]

arctan = R (1/(1 + t2))

tan = revert arctan

T = tan −sin / cos Every term of T should vanish.

2.4 Variation: multivariate series

Coeﬃcients in a power series may have any type for which the usual arithmetic functions are deﬁned and ﬁnite and for which multiplication is commutative. In particular coeﬃcients may be univariate polynomials, in which case the series is said to be in nested form.

Example 5, Pascal triangle. The generating function for the binomial coeﬃcients of order n is (1 + y)n. To enumerate the binomial coeﬃcients for all n, we may replace z in the geometric series 1+z +z2 +z3 +· · · by (1+y)z. That is, we compose the generating function for the geometric series, 1/(1−z), with the polynomial 0 + (1 + y)z:

(1/[1, −1]) ◦[[0], [1, 1]] The result is the Pascal triangle,

[[1], [1, 1], [1, 2, 1], [1, 3, 3, 1], [1, 4, 6, 4, 1], · · · ].

2.4.1 Homogeneous form

It is often convenient to group terms of a bivariate power series in x and y by total degree, with the nth group being a homogeneous polynomial of degree n. The groups may be carried as coeﬃcients in a power series in z:

f(x, y, z) = (f00) + (f10x + f01y)z + (f20x2 + f11xy + f02y2)z2 + · · · Since the degree of any variable in a term is determinable from the degree of the other two, we can elide any one variable—i.e. set the variable to 1. Since elision commutes with addition and multiplication, there is a direct isomorphism between the ring of nested-form series f(1, y, z) and the ring of homogeneous-form series f(x, y, 1). We can exploit this isomorphism to compute with homogeneous-form series.

The following rules take partial derivatives of a homogeneous-form series. The y-derivative of the homogeneous polynomial f1(x, y) is calculated by

applying the ordinary derivative operator to the isomorph f1(1, y). The xderivative works on the isomorph f1(x, 1), whose terms come in reverse order. So that the ﬁrst term that operator D sees is the term of x-degree zero, f1 must have exactly n + 1 terms, where n is the total degree of f1.

Partial derivative Dy( : f@(f1 : )) = (D f1) : (Dy f)

Dy = [ ]

Dx( : f@(f1 : )) = (reverse · D · reverse)f1 : (Dx f)

Dx = [ ] The operator (·) is ordinary functional composition. It is distinguished from (◦), which applies to sequences, not functions.

Example 6, more tough tests. The identities ex+y = exey and Dxex+y = ex+y

yield test functions, T1 and T2, that should evaluate to zero.

x = [[0], [1, 0]]

y = [[0], [0, 1]]

T1 = (exp ◦(x + y)) −(exp ◦x) × (exp◦y)

T2 = Dx(exp ◦(x + y)) −Dy(exp◦(x + y)) These tests are truly vigorous exercises that take O(n5) rational operations to carry out to n terms.

3 Second theme: Maclaurin form

In Maclaurin form, a power series is represented by the list of derivatives evaluated at the origin:

[f(0), f′(0), f′′(0), f(3)(0), · · · ]. The representation of the derivative f′ is the tail of the representation of f. Writing f0 = f(0), we have

f = f0 : f′

Operator (:) is the ordinary list constructor, decorated as a reminder that it now constructs Maclaurin form.

Derivative and integral operators shift sequences left or right. Some arithmetic rules and the reversion rule are the same as for Horner form. Only distinctive rules are given here. In particular, rules for empty lists are omitted. The rules for arithmetic and composition are direct consequences of the usual formulas for derivative of product and quotient, and the chain rule, (f(g))′ = f′(g)g′.

Derivative. D( :f′) = f′

Integral. R f = 0 : f

Product. f@(f0 : f′) × g@(g0 : g′) = f0g0 : (f × g′ + g × f′)

Quotient. f@(f0 : f′) / g@(g0 : g′) = q where q = f0/g0 : (f′ −q × g′)/g

Composition. (f0 : f′) ◦g@(0 : g′) = f0 : (f′ ◦g) × g′

Conversion from Horner to Maclaurin form (hm) and vice versa (mh) are straightforward.

Conversions. hm[ ] = [ ]

hm(f@(f0 : )) = f0 : hm(Df)

mh[ ] = [ ]

mh(f0 :f′) = f0 + R mh(f′)

3.1 Behavioral diﬀerential equations

Relations written in the form y = y0 : y′ are called behavioral diﬀerential equations (Rutten, 1999), or BDEs.

Example 7, elementary functions. From the diﬀerential equations for the elementary functions (Example 1) we read oﬀthese BDEs:

sin = 0 : cos

cos = 1 : −sin

exp = 1 : exp Executing any of these functions—a trivial exercise—yields a sequence of derivatives evaluated at the origin. The sequence for the cosine is ˇ[1, 0, −1, 0, 1, 0, −1, 0, · · · ].

3.2 Doing math

The power-stream idea, though born in computing, is fruitful in mathematics, too. Derivations of mathematical relationships among objects deﬁned by BDEs can be quite direct and elegant.

Example 8, reciprocal of the exponential. Verify the identity 1/ex = e−x, knowing only that ex denotes the solution of exp = 1 : exp. Let

q = 1/ exp

= 1 : 1′ −q exp′ / exp Quotient rule

= 1 : −q exp / exp Example 7, Derivative rule

= 1 : −q

z = [0, 1]

r = exp ◦(−z)

= (1 : exp) ◦(−z) Example 7

= 1 : (exp′ ◦(−z)) × (−z)′ Composition rule

= 1 : (exp ◦(−z)) × (−1) Example 7, Derivative rule

= 1 : −r Since q and r satisfy the same behavioral diﬀerential equation, the desired result holds. Notice that the calculation deals with inﬁnite objects without using sums or limits. It instead uses properties of the ﬁxed points of BDEs.

Example 9, Bell numbers. The numbers Bn count distinct ways to distribute n distinguishable objects among indistinguishable boxes (Bell 1934). They have an exponential generating function

B(z) = P∞ n=0 Bnzn/n! = eez−1

Thus the Bell numbers are produced by the program

bell = exp ◦(exp−1)

= exp ◦((1 : exp) −1) Example 7

= exp ◦(0 : exp) It is a pleasant exercise to show that

bell = 1 : bell × exp is equivalent; both programs produce

[1, 1, 2, 5, 15, 52, 203, 877, 4140, 21147, · · · ].

4 Coda

4.1 Complexity

Eﬃciency can vary dramatically with algorithm. Section (2.3) gives O(n2) and O(n) rules for the derivative. Example 2 gives O(n3) and O(n2) algorithms for calculating the exponential of a series. More dramatically, the product op-

erators for Maclaurin and Horner forms respectively take O(2n) and O(n2) coeﬃcient-domain operations to compute n terms. Evidently it would be advantageous to compute the product of Maclaurin-form series by bypassing (Melzak 1983) to Horner form and back. On the other hand, it takes O(n) coeﬃcient-domain operations to compute n terms of the elementary functions in either form.

4.2 Power streams as system tests

Besides exercising the basic rules, the “tough tests” of Examples 4 and 6 also severely stress implementations of lazy-list languages, and of streamprocessing systems similar to CSP (Hoare, 1985). Example 4 alone has shown up bugs or severe capacity limitations in several systems and languages in which I have tried them. Fortunately, all were promptly ﬁxed by responsive implementers.

4.3 Real Haskell

While the programs above deviate from pure Haskell only in minor typographic ways, they omit details of operator overloading for the sake of accessibility. The full story of how to make the code work in Haskell is given by McIlroy (1999).

4.4 Inﬂuences

Power-stream compositions, once mainly an oral tradition, have begun to take their place in the literature, Two remarkable papers, by Karczmarczuk (1997) and by Pavlovi´c and Escard´o (1998), exploit the two forms. Karczmarczuk deals mainly with special functions of mathematical physics in Horner form. Pavlovi´c and Escard´o examine Maclaurin form and related representations from an algebraic standpoint, and advocate power streams as a productive formalism for calculus. McIlroy discusses Horner form as an application of Haskell, and Hehner (1993) integrates the subject with the logic of programs. Rutten (1999) relates Maclaurin form to automata theory.

4.5 Resolution

As the music of power streams, always fascinating, has been arranged for successively more agile instruments, the melodic form has become purer and more alluring. Early performances of power streams resembled concerts of street cries, in which transactions were shouted out among producers and consumers: “Get me a term, oh/ Here is your term, oh”—a striking but stolid form. Scored now for a well tuned organ of lazy evaluation, the genre has, I trust, become worthy of performance for that prince of computing, Edsger W. Dijkstra.

References

[1] Bell, E. T. 1934. Exponential numbers. American Mathematical Monthly 41: 411419.

[2] Hehner, E. C. R. 1993. A Practical Theory of Programming, 143-148. Springer.

[3] Hoare, C. A. R. 1985. Communicating Sequential Processes. Prentice-Hall.

[4] Karczmarczuk, J. 1997. Generating power of lazy semantics. Theoretical Computer Science, 9: 203–219.

[5] Knuth, D. E. 1969. The Art of Computer Programming, Volume 2, Seminumerical Algorithms, §4.7. Addison-Wesley.

[6] McIlroy, M. D. 1999. Functional pearl: power series, power serious. Journal of Functional Programming, 9: 323–335; also at http://www.cs.dartmouth.edu/~doug/haskell/pearl.ps.gz.

[7] Melzak, Z. A. 1983. Bypasses: a Simple Approach to Complexity. John Wiley & Sons.

[8] Pavlovi´c, D. and Escard´o, M. H. 1998. Calculus in coinductive form, Proceedings of the 13th Annual IEEE Symposium on Logic in Computer Science, 408–417. IEEE Computer Science Society Press.

[9] Peterson, J. and Hammond, K. (editors) 1997. Report on the Programming Language Haskell. Available from http://www.haskell.org.

[10] Rutten, J. J. M. M. 1999. Automata, power series, and coinduction: taking input derivatives seriously. Proceedings of ICALP ’99, J. Wiedermann, P. van Emde Boas and M. Nelson, editors, 645–654. Springer LNCS 1644.
