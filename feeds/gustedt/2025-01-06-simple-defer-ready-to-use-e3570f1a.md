---
title: Simple defer, ready to use
url: https://gustedt.wordpress.com/2025/01/06/simple-defer-ready-to-use/
published: "2025-01-06T15:30:00Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=4243
---

# Simple defer, ready to use

I have already [talked several times about `defer`](https://gustedt.wordpress.com/2024/09/11/braiding-the-spaghetti-implementing-defer-in-the-preprocessor/), the new feature that hopefully will make it into a future version of C. With this post I will concentrate on the here and now: how to apply that lifesaving feature with existing tools and compilers.

After briefly discussing the `defer` feature itself, again, I will show you a first implementation with gcc extensions, a second with C++ standard features and then I will discuss [a new proposal with for `defer`](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3434.htm) that has a syntax that is a tiny bit more constraining than what you may have seen so far.

## The defer feature itself

To remind you what `defer` is supposed to do, here is some toy code that uses three resources:

```
{                             // anchor block
    void* p = malloc(25);
    defer { free(p); };          // 1st defer

    mtx_lock(&mut);
    defer { mtx_unlock(&mut); }; // 2nd defer

    static uint64_t critical = 0;
    critical++;
    defer { critical--; };       // 3rd defer

    while (something) cnd_wait(&cond, &mut);
    ... use p under protection of mut ...
}

```

This code with three so-called deferred blocks is equivalent to the following

```
{
    void* p = malloc(25);

    mtx_lock(&mut);

    static uint64_t critical = 0;
    critical++;

    while (something) cnd_wait(&cond, &mut);
    ... use p under protection of mut ...

    { critical--;       } // from 3rd defer
    { mtx_unlock(&mut); } // from 2nd defer
    { free(p);          } // from 1st defer
}

```

only that the deferred blocks are even executed when the anchor block is left by a jump statement (such as a `break`, `continue`, `return` or even `goto`) that would be nested inside complicated `if`/ `else` conditionals. Thus using `defer` here ensures that

- `p`, `something` and `critical` are only used under the protection of the mutex `mut`,
- `critical` then holds the number of threads that are currently within that anchor block
- `mut` never remains locked whenever the anchor block is left.
- `*p` is deallocated whenever the anchor block is left.

## Implementing defer with gcc

With a minimal macro wrapper this feature works out of the box in gcc since at least two decades. Written with C23’s attribute feature the inner macro looks as simple as the following:

```
#define __DEFER__(F, V)      \
  auto void F(int*);         \
  [[gnu::cleanup(F)]] int V; \
  auto void F(int*)

```

Here the three lines of the macro are as follows:

- `auto void F(int*);` forward-declares a nested (local) function `F`.
- `[[gnu::cleanup(F)]] int V;` establishes this function as a cleanup handler of an auxiliary variable `V`
- The second `auto void F(int*)` then starts the definition of the local function which is completed with the user’s compound statement as the function body.

For this to work we have to provide unique names `F` and `V` such that several defer blocks may appear within the same anchor block. This is ensured by another very common extension `__COUNTER__`

```
#define defer __DEFER(__COUNTER__)
#define __DEFER(N) __DEFER_(N)
#define __DEFER_(N) __DEFER__(__DEFER_FUNCTION_ ## N, __DEFER_VARIABLE_ ## N)

```

That is basically it, a straight application of the `[[gnu::cleanup]]` feature that

- avoids the need for inventing safe identifiers,
- avoids the definition of an one-shot function with internal or external linkage far away from its use,
- integrates well and quite efficiently with the existing compiler infrastructure.

Indeed, when adding a bit more magic (such as `[[gnu::always_inline]]`) the assembly that is produced is very efficient and avoids function calls, trampolines and indirections. (See also [Omar Anson’s blog entry](https://omeranson.github.io/blog/2022/06/12/cleanup-attribute-in-C) on how efficient the cleanup attribute seems to be implemented in gcc.)

If you don’t like or have the C23 attribute syntax yet, you should easily be able to use gcc’s legacy `__attribute__((...))` syntax without problems.

## Implementing defer with C++

Yes! I think it would even make sense for C++, but what do I know.

At least the following implementation shows that the feature fits directly into C++’s model of binding actions to a scope. In fact, implementing the defer feature with the properties as desired in C++ is a student excercise. It can be done with a `template` class and lambdas.

```
template<typename T>
struct __df_st  : T {
  [[gnu::always_inline]]
  inline
  __df_st(T g) : T(g) {
    // empty
  }
  [[gnu::always_inline]]
  inline
  ~__df_st() {
    T::operator()();
  }
};

#define __DEFER__(V)  __df_st const V = [&](void)->void

```

Lambda expressions have a unique type for each expression. Such a lambda expression is taken here as a template parameter to the constructor `__df_st<T>::__df_st(T)`. The destructor `__df_st<T>::~__df_st()` of the variable then invokes the lambda when `V` leaves its scope. The `[&]` in

```
[&](void)->void { some code }

```

ensures that all outer variables are fully accessible at the point of execution of the lambda. Therefore, such an implementation provides the full functionality that we need for our first example from above.

Note, that the `__COUNTER__` pseudo-macro is also quite commonly implemented by C++ compilers and is now even proposed as an [addition to C++26](https://isocpp.org/files/papers/P3384R0.html). Thus with a similar macro as for gcc above we can implement the `defer` macro itself:

```
#define defer __DEFER(__COUNTER__)
#define __DEFER(N) __DEFER_(N)
#define __DEFER_(N) __DEFER__(__DEFER_VARIABLE_ ## N)

```

## A note on the syntax for defer

As we have seen above existing implementations (let’s also count the one for C++) use quite different features under the hood:

One uses a function declaration where a compound statement of the user code as in

```
    defer { mtx_unlock(&mut); };

```

ends up being a function body. Here the terminating `;` is superfluous, but doesn’t hurt much, either.

The other declares a expression that also needs `{}` to limit the user code and a terminating `;` because underneath it is an object declaration.

I think we should combine these different syntax requirements, such that implementors (or library providers) may easily start providing this feature with what they have. I have recently made a new proposal to the C standards committee in that sense:

[Even simpler defer for direct integration, N3434](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3434.htm)

It has the advantage, I think, that it is much easier and more direct that previous proposals and that it combines the possibility of implementing the feature in the language or in the library. Therefore the defer features is proposed as a “block item” in a compound statement (its anchor) and then defined via the syntax rules

> _defer-block:_

> > `defer` _deferred-block_ `;`

> _deferred-block:_

> > _compound-statement_
