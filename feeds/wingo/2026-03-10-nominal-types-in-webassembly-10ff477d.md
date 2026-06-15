---
title: nominal types in webassembly
url: https://wingolog.org/archives/2026/03/10/nominal-types-in-webassembly
published: "2026-03-10T08:19:34Z"
feed: wingo
guid: https://wingolog.org/2026/03/10/nominal-types-in-webassembly
---

# nominal types in webassembly

Before the managed data types extension to WebAssembly was incorporated
in the standard, there was a huge debate about type equality. The end
result is that if you have two types in a Wasm module that look the
same, like this:

```
(type $t (struct i32))
(type $u (struct i32))

```

Then they are for all intents and purposes equivalent. When a Wasm
implementation loads up a module, it has to partition the module’s types
into equivalence classes. When the Wasm program references a given type
by name, as in `(struct.get $t 0)` which would get the first field of
type `$t`, it maps `$t` to the equivalence class containing `$t` and
`$u`. See the [spec](https://webassembly.github.io/spec/core/valid/conventions.html#rolling-and-unrolling), for more details.

This is a form of _structural type equality_. Sometimes this is what you
want. But not always! Sometimes you want _nominal types_, in which no
type declaration is equivalent to any other. WebAssembly doesn’t have
that, but it has something close: _recursive type groups_. In fact, the
type declarations above are equivalent to these:

```
(rec (type $t (struct i32)))
(rec (type $u (struct i32)))

```

Which is to say, each type is in a group containing just itself. One
thing that this allows is self-recursion, as in:

```
(type $succ (struct (ref null $succ)))

```

Here the struct’s field is itself a reference to a `$succ` struct, or
null (because it’s `ref null` and not just `ref`).

To allow for mutual recursion between types, you put them in the same `rec`
group, instead of each having its own:

```
(rec
 (type $t (struct i32))
 (type $u (struct i32)))

```

Between `$t` and `$u` we don’t have mutual recursion though, so why
bother? Well `rec` groups have another role, which is that they are the
unit of structural type equivalence. In this case, types `$t` and `$u`
are not in the same equivalence class, because they are part of the same
`rec` group. Again, see [the spec](https://webassembly.github.io/spec/core/valid/conventions.html#defined-types).

Within a Wasm module, `rec` gives you an approximation of nominal
typing. But what about between modules? Let’s imagine that `$t`
carries important capabilities, and you don’t want another module to be
able to forge those capabilities. In this case, `rec` is not enough:
the other module could define an equivalent `rec` group, construct a
`$t`, and pass it to our module; because of isorecursive type equality,
this would work just fine. What to do?

### cursèd nominal typing

I said before that Wasm doesn’t have nominal types. That was true in
the past, but no more! The [nominal typing\
proposal](https://github.com/WebAssembly/exception-handling/blob/main/proposals/exception-handling/Exceptions.md)
was incorporated in the standard last July. Its vocabulary is a bit
odd, though. You have to define your data types with the [`tag` keyword](https://webassembly.github.io/spec/core/syntax/types.html#tag-types):

```
(tag $v (param $secret i32))

```

Syntactically, these data types are a bit odd: you have to declare
fields using `param` instead of `field` and you don’t have to wrap the
fields in `struct`.

They also omit some features relative to isorecursive structs, namely
subtyping and mutability. However, sometimes subtyping is not
necessary, and one can always assignment-convert mutable fields, wrapping them in mutable structs as needed.

To construct a nominally-typed value, the mechanics are somewhat
involved; instead of `(struct.new $t (i32.const 42))`, you use [`throw`](https://webassembly.github.io/spec/core/exec/instructions.html#xref-syntax-instructions-syntax-instr-control-mathsf-throw-x):

```
(block $b (result (ref exn))
 (try_table
  (catch_all_ref $b)
  (throw $v (i32.const 42)))
 (unreachable))

```

Of course, as this is a new proposal, we don’t yet have precise type
information on the Wasm side; the new instance instead is returned as
the top type for nominally-typed values, `exn`.

To check if a value is a `$v`, you need to write a bit of code:

```
(func $is-v? (param $x (ref exn)) (result i32)
  (block $yep (result (ref exn))
   (block $nope
    (try_table
     (catch_ref $v $yep)
     (catch_all $nope)
     (throw_ref (local.get $x))))
   (return (i32.const 0)))
  (return (i32.const 1)))

```

Finally, field access is a bit odd; unlike structs which have
`struct.get`, nominal types receive all their values via a `catch`
handler.

```
(func $v-fields (param $x (ref exn)) (result i32)
  (try_table
   (catch $v 0)
   (throw_ref (local.get $x)))
  (unreachable))

```

Here, the `0` in the `(catch $v 0)` refers to the function call itself:
all fields of `$v` get returned from the function call. In this case
there’s only one, othewise a get-fields function would return multiple
values. Happily, this accessor preserves type safety: if `$x` is not
actually `$v`, an exception will be thrown.

Now, sometimes you want to be quite strict about your nominal type
identities; in that case, just define your `tag` in a module and don’t
export it. But if you want to enable composition in a principled way,
not just subject to the randomness of whether another module happens to
implement a type structurally the same as your own, the nominal typing
proposal also gives a preview of [type\
imports](https://github.com/WebAssembly/proposal-type-imports/blob/main/proposals/type-imports/Overview.md).
The facility is direct: you simply export your `tag` from your module,
and allow other modules to import it. Everything will work as expected!

### fin

Friends, as I am sure is abundantly clear, this is a troll post :) It’s
not wrong, though! All of the facilities for nominally-typed structs
without subtyping or field mutability are present in the
exception-handling proposal.

The context for this work was that I was updating
[Hoot](https://spritely.institute/hoot/) to use the newer version of
Wasm exception handling, instead of the pre-standardization version. It
was a nice change, but as it introduces the `exnref` type, it does open
the door to some funny shenanigans, and I find it hilarious that the
committee has been hemming and hawwing about type imports for 7 years
and then goes and ships it in this backward kind of way.

Next up, exception support in
[Wastrel](https://codeberg.org/andywingo/wastrel), as soon as I can
figure out where to allocate type tags for this new nominal typing
facility. Onwards and upwards!
