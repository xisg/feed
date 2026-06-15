---
title: two mechanisms for dynamic type checks
url: https://wingolog.org/archives/2026/02/18/two-mechanisms-for-dynamic-type-checks
published: "2026-02-18T16:21:10Z"
feed: wingo
guid: https://wingolog.org/2026/02/18/two-mechanisms-for-dynamic-type-checks
---

# two mechanisms for dynamic type checks

Today, a very quick note on dynamic instance type checks in virtual
machines with single inheritance.

The problem is that given an object _o_ whose type is _t_, you want to
check if _o_ actually is of some more specific type _u_. To my knowledge, there are two sensible ways to implement these type checks.

### if the set of types is fixed: dfs numbering

Consider a set of types _T_ := { _t_, _u_, ...} and a set of edges _S_ :=
{< _t_ \|ε, _u_ >, ...} indicating that _t_ is the direct supertype of
_u_, or ε if _u_ is a top type. _S_ should not contain cycles and is
thus a direct acyclic graph rooted at ε.

First, compute a pre-order and post-order numbering for each _t_ in the
graph by doing a depth-first search over _S_ from ε. Something like
this:

```
def visit(t, counter):
    t.pre_order = counter
    counter = counter + 1
    for u in S[t]:
        counter = visit(u, counter)
    t.post_order = counter
    return counter

```

Then at run-time, when making an object of type _t_, you arrange to
store the type’s pre-order number (its _tag_) in the object itself. To
test if the object is of type _u_, you extract the tag from the object
and check if _tag_– _u_.pre\_order mod 2_n_ <
_u_.post\_order– _u_.pre\_order.

Two notes, probably obvious but anyway: one, you know the numbering for
_u_ at compile-time and so can embed those variables as immediates.
Also, if the type has no subtypes, it can be a simple equality check.

Note that this approach applies only if the set of types _T_ is fixed.
This is the case when statically compiling a WebAssembly module in a
system that doesn’t allow modules to be instantiated at run-time, like
[Wastrel](https://codeberg.org/andywingo/wastrel). Interestingly, it
can [also be the case in JIT compilers, when modeling types inside the\
optimizer](https://bernsteinbear.com/blog/toy-tbaa/?utm_source=rss).

### if the set of types is unbounded: the display hack

If types may be added to a system at run-time, maintaining a sorted set
of type tags may be too much to ask. In that case, the standard
solution is something I learned of as the _display hack_, but whose name
is apparently ungooglable. It is described in a 4-page technical note
by Norman H. Cohen, from 1991: [_Type-Extension Type Tests Can Be_\
_Performed In Constant_\
_Time_](https://dl.acm.org/doi/10.1145/115372.115297).

The basic idea is that each type _t_ should have an associated sorted
array of supertypes, starting with its top type and ending with _t_
itself. Each _t_ also has a _depth_, indicating the number of edges
between it and its top type. A type _u_ is a subtype of _t_ if
_u_\[ _t_.depth\]= _t_, if _u_.depth <= _t_.depth.

There are some tricks one can do to optimize out the depth check, but
it’s probably a wash given the check performs a memory access or two on
the way. But the essence of the whole thing is in Cohen’s paper; go
take a look!

Jan Vitek notes in a followup paper ( [_Efficient Type Inclusion Tests_](https://dl.acm.org/doi/pdf/10.1145/263700.263730)) that Christian
Queinnec discovered the technique around the same time. Vitek also mentions the DFS technique, but as prior art, apparently already deployed in DEC Modula-3 systems.
The term “display” was bouncing around in the 80s to describe some uses of
arrays; I learned it from Dybvig’s implementation of flat closures, who
learned it from Cardelli. I don’t know though where “display hack” comes from.

That’s it! If you know of any other standard techniques for type checks
with single-inheritance subtyping, do let me know in the comments.
Until next time, happy hacking!

_Addendum: Thanks to kind readers, I have some new references! Michael Schinz refers to [Yoav Zibin’s PhD thesis](https://yoav-zibin.github.io/homepage/publications/my-thesis.pdf) as a good overview. Alex Bradbury points to [a survey article by Roland Ducournau](https://dl.acm.org/doi/10.1145/1922649.1922655) as describing the DFS technique as “Schubert numbering”. CF Bolz-Tereick unearthed the [1983 Schubert paper](https://www.cs.rochester.edu/u/schubert/papers/type-part-color-and-time-relationships83.pdf), and [it is a weird one](https://mastodon.social/@wingo/116097769070475183). Still, I can’t but think that the DFS technique was known earlier; I have a 1979 graph theory book by Shimon Even that describes a test for “separation vertices” that is precisely the same, though it does not mention the application to type tests. Many thanks also to fellow traveller Max Bernstein for related discussions._
