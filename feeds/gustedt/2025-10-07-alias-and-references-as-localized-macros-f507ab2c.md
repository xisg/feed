---
title: Alias and references as localized macros
url: https://gustedt.wordpress.com/2025/10/07/alias-and-references-as-localized-macros/
published: "2025-10-07T13:03:27Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=4586
---

# Alias and references as localized macros

Since the last meeting of the C committee I am struggling with the idea of aliases as they are proposed in a long series of papers by JeanHeyd.

> [Transparent Aliases](https://www.open-std.org/JTC1/SC22/WG14/www/docs/n3689.htm)

I am struggling with this for several reasons, but most importantly is that I have the impression that it is a heavy gun pointed on a problem that merely seems to be about tooling, or at least is presented as such. And then, there is this rejection of macro solutions that drives me nuts. That looks just like bashing to me, evacuation a possible solution without giving if the necessary thoughts.

```
_Alias(memcpy, my_meme);

```

(the proposed syntax is a bit different, but let’s not go into that for now)

This declares an identifier `memcpy` for the rest of the scope that is as good as using `my_meme` everywhere. This then could be used for configuring a source for example with a `#if` cascade that chooses the replacement according to some word size, target architecture capacity or so. Once compiled, the resulting binary would then always use the same function `my_meme`, even if the system’s `memcpy` is linked dynamically.

If we see this as a simple replacement of identifiers (which it isn’t) we could be tempted to just declare a macro `memcpy` that resolves to the concrete choice I want to make at that point in the source. And that is indeed the way that is foreseen by the C library: any C library function may be implemented as a macro that points to the stuff that we need.

## Problems with the usual macro approaches

There are several problems with this approach.

First, we’d have to ensure that macro quacks like a function _descriptor_, that is

- usages with parenthesis and arguments should result in a call,
- usages without parenthesis should convert to a function pointer,
- applying the `&` operator should convert to a function pointer,
- applying the `*` operator should give back the function descriptor.

So, first the only choice that leaves for a macro is to have it “object-like”, that is without parenthesis in the definition.

Then, there is the problem of locality of definition. We’d want to choose at the point of definition, to which other symbol my macro resolves. A simple macro replacement takes whatever symbol `my_meme` is defined at the point of macro _invocation_. That could be something completely different than what the coder of the macro had in mind.

Another problem is the one of scope. If I define my replacement just within a code block, I wouldn’t want the macro to pollute the code that follows that block and randomly replace things there.

Add to that the usual problem of macros, that they also replace names in other name spaces, for example structure members, or where the symbol would locally be used otherwise, for example a function parameter named `memcpy`. But that is clearly something that we can’t change easily. It is inherent to the macro approach and C programmers should know how to deal with this.

## A serious approach to provide aliases with macros

A spelled out solution to do aliases with macros could look as follows

```
auto const __memcpy_alias_7854 = &(my_meme);
#define memcpy /*space*/ (*__memcpy_alias_7854)
...
#undef memcpy // only if used inside a {} block

```

So first of all, we provide ourselves with an immutable variable that holds the address of the symbol which we are after. Then we ensure that the identifier `memcpy` will always resolve back to the original function descriptor. Now

- `memcpy(something)` will call `my_meme`.
- `memcpy` without parenthesis is `(*__memcpy_alias_7854)` a function descriptor that decays to a function pointer, just as a direct use of `my_meme` would.
- `&memcpy` is `&(*__memcpy_alias_7854)` where `&*` cancel out and the result is a function pointer, just as a direct use of `my_meme` would.
- `*memcpy` is `*(*__memcpy_alias_7854)` which first decays `(*__memcpy_alias_7854)` to the function pointer and then applies `*` to result in a function descriptor, again, just as a direct use of `*my_meme` would.

So in all, this approach solves most of the inherent problems of doing this with a macro.

Some remarks and observations:

- The ergonomics of that approach could be better. You’d have to implement two or three consistent lines, watch about the space between the name and the opening parenthesis, and in addition you need to invent a random, unique name for the pointer. Not beautiful.
- This uses C23’s `auto` feature. If you don’t have that (yet) you’d have to know your type or use an extension such as gcc’s `__typeof__` for the declaration.
- You’d probably like the auxiliary variable to be as “optimizable” as possible. So in block scope `register` would be in order. If you are in file scope, you wouldn’t want the auxiliary name pollute your linker space, so you’d probably have that as `static`.
- Any decent modern compiler with some optimization switched on should be able to make the use of the function pointer transparent and optimize it away completely.

So, if you are willing to accept the bad ergonomics, this is a solution for the problem.

## An encapsulation with eĿlipsis

Enters eĿlipsis, obviously I can’t let such a solution out, without programming some improvement into my preprocessor. In the next release, there is a new header `<ellipsis-reference.h>` that implements a macro `_Alias` that follows that idea: it introduces a local alias for an addressable lvalue or function descriptor and defines a macro that is bound to the current C scope. So the behavior is as-if when this is used inside some pair of `{}`, an `#undef` of the macro is inserted at the closing brace.

The second argument (and possible further arguments all together) have to expand to an lvalue or function descriptor. Consider the following example with a user function `my_strlen`.

```
char* fun(char x[]) {
   _Alias(strlen, my_strlen);
   …
   size_t len = strlen(x);
   …
}

```

Here all invocations of the `strlen` object-like macro inside the function are replaced by `(*_Alias_strlen_34)` as follows. (The `34` is just a number to ensure that the identifier is unique.)

```
char* fun(char x[]) {
   [[maybe_unused]] register auto const _Alias_strlen_34 = &(__my_strlen);
   …
   size_t len = (*_Alias_strlen_34)(x);
   …
}

```

Note that in addition to the above, this is automatically annotated with `[[maybe_unused]]`, so unused aliases in headers wouldn’t trigger warnings, and with `register` such that the address of `_Alias_strlen_34` can never be taken. Unfortunately the latter does not work in file scope, so there `static` is used instead.

## Possible use cases for `_Alias`

The use of `_Alias` goes far beyond the initial motivation for finding a tool to freeze architectural choices consistently to a given situation. Even with JeanHeyd’s feature we could give the name of a variable for the RHS and thus have an alias refer to an object. With eĿlipsis we can even do more, something like

```
char* fan(double (*Ap)[42]) {
   _Alias(A, *Ap);
   …
   for (size_t j = 0; j < 23; ++j) {
       for (size_t i = 0; i < _Countof(A); i++) {
           A[i] += i;
       }
       ... do something else ...
   }
}

```

This would provide us with the possibility to see a function parameter as some sort of reference and in particular handle parameter arrays in a convenient way.

I am not a fan of that approach as shown above, because it is, as the name indicates, an alias and not a reference. An alias provides a new name for an existing feature but it does not ensure that the feature is not accessed through other channels. Much of C and its optimization capabilities come from the fact that it avoids such aliases and that the compiler can safely assume that they have the only view on an object. In the example above the line

```
           A[i] += i;

```

might allow for very different optimizations, when there is a guarantee that the view through `A` is the only one, than when it is not.

## Lvalue references

So a reference (as provided by many programming languages other than C) is more than an alias, it guarantees

- that there is an object, and
- that, in the given scope, the access to this object is unique.

These assumptions allow the compiler to do a lot of optimizations that otherwise would not be possible.

As you probably know, C does not have that concept, and this is a conscious choice. Many C programmers become very passionate about this, the mere possibility of aliasing or hiding pointers from view can put us into crisis mode.

## A macro version of lvalue references for function parameters

The eĿlipsis header `<ellipsis-reference.h>` has another macro for parameter declarations that avoids some of these problems namely `_LVparam`. Other than `_Alias` it is just applied to the name of a parameter. Taken the `fan` example from above that would be

```
char* fan(double _LVparam(A)[42]) {
   …
   for (size_t j = 0; j < 23; ++j) {
       for (size_t i = 0; i < _Countof(A); i++) {
           A[i] += i;
       }
       ... do something else ...
   }
}

```

that is expanded to something like

```
char* fan(double (_LVparam_A_16[restrict const static 1])[42]) {
   …
   for (size_t j = 0; j < 23; ++j) {
       for (size_t i = 0; i < _Countof((*_LVparam_A_16)); i++) {
           (*_LVparam_A_16)[i] += i;
       }
       ... do something else ...
   }
}

```

So almost the same as before, but not quite. The difference is that the `_LVparam_A_16` pointer parameter is given as an array with additional annotations. (Remember that C has the crude rule that the innermost array bound of a _parameter_ is rewritten to a pointer.)

- `restrict` says that the pointer is the only possible way to access the pointed-to object inside the function.
- `static 1` says that the pointed-to object has at least one element. So in particular, the pointer is never null.

So with these additional annotations, `A` now is really a reference to a vector object. But be careful, whereas the compiler may now make a uniqueness assumption inside the function, there is no guarantee in C that this property is checked on the caller side. Compilers are getting better with this, but there are situations where they just can’t tell. Use your favorite static analyzer to obtain the most possible guarantees.

The `_LVparam` feature has the necessary properties to be applicable to real code, namely the macro definition of the parameter name extends from the definition to

- the closing brace of the function body if the function declaration is also a definition;
- to the terminating semicolon if the declaration is a forward declaration of the function.

## A macro version of lvalue pseudo-references for non-parameter objects

Unfortunately, the `static 1` notation as used above only works for array parameters (that are then rewritten to pointers). For references that we would like to define inside functions such a feature is not available. Nevertheless, we can use a similar feature for local lvalue references, `_LVlocal`,

```
double _LVlocal(A)[len] = calloc(sizeof A, 1);

```

This expands to something similar to

```
double (*restrict const _LVlocal_A_12)[len]
   = calloc(sizeof(*_LVlocal_A_12), 1);
if (!LVlocal_A_12) exit(EXIT_FAILURE);

```

Here we directly declare a pointer and again, `restrict` makes the promise that this pointer is unique. Unfortunately, there is currently no direct syntax to also ensure that the pointer is non-null. So the tool inserts the check automatically at the next semicolon in the source.

If we then even add an automatic destruction that triggers when the scope is left

```
double _LVlocal(A)[len] = calloc(sizeof A, 1);
defer { free(&A); };

```

we have implemented a stack-safe version of a variable length array, VLA that checks automatically if the allocation succeeded. This solution needs that your compiler implements the new `defer` feature or that you are using the one from eĿlipsis.

Another possible solution for the problem that the allocation of VLAs may not succeed would be not to provide the implicit test as it is implemented here, but to leave it to the user code to test for it simply by testing if `&A` is null.

## Conclusion

C already has the semantics to cover features such as JeanHeyd’s proposed `_Alias` and more generally to implement references of objects or VLA that include runtime checks instead of smashing your stack. I personally am not yet completely convinced that I want any of this in my code, but we have it now in eĿlipsis. These features are not completely equivalent to proper language features (macro declarations in eĿlipsis still can’t shaddow one another, for example) but it comes quite close and should be good for at least 90% of the use cases.

So please give it a try if you like these kind of things.
