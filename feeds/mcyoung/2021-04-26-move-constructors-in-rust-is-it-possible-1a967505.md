---
title: "Move Constructors in Rust: Is it possible?"
url: /2021/04/26/move-ctors
published: "2021-04-26T00:00:00Z"
feed: mcyoung
guid: /2021/04/26/move-ctors
---

# Move Constructors in Rust:

Is it possible?

I’ve been told I need to write this idea down – I figure this
one’s a good enough excuse to start one of them programming blogs.

_TL;DR_ You _can_ move-constructors the Rust! It requires a few
macros but isn’t much more outlandish than the `async` pinning
state of the art. A prototype of this idea is implemented in my
[`moveit`](https://crates.io/crates/moveit) crate.

## [The Interop Problem](#the-interop-problem)

Rust is the best contender for a C++ replacement; this is not even
a question at this point[1](#fn:1). It’s a high-level language
that provides users with appropriate controls over memory, while
also being memory safe. Rust accomplishes by codifying C++ norms
and customs around ownership and lifetimes into its type system.

Rust has an ok[2](#fn:2) FFI story for C:

```c highlight
void into_rust();

void into_c() {
  into_rust();
}

```

[C](#code:1)

```rust highlight
extern "C" {
  fn into_c();
}

#[no_mangle]
extern "C" fn into_rust() {
  unsafe { into_c() }
}

```

[Rust](#code:2)

Calling into either of these functions from the Rust or C side
will recurse infinitely across the FFI boundary. The
`extern "C" {}` item on the Rust side declares C symbols, much
like a function prototype in C would; the `extern "C" fn` is a
Rust function with the C calling convention, and the
`#[no_mangle]` annotation ensures that `recurse_into_rust` is the
name that the linker sees for this function. The link works out,
we run our program, and the stack overflows. All is well.

But this is C. We want to rewrite all of the world’s C++ in Rust,
but unfortunately that’s going to take about a decade, so in the
meantime new Rust must be able to call existing C++, and
vise-versa. C++ has a much crazier ABI, and while Rust gives us
the minimum of passing control to C, libraries like \[ `cxx`\]
need to provide a bridge on top of this for Rust and C++ to talk
to each other.

Unfortunately, the C++ and Rust object models are, a priori,
incompatible. In Rust, _every object_ may be “moved” via `memcpy`,
whereas in C++ this only holds for types satisfying
`std::is_trivially_moveable` [3](#fn:3). Some types require
calling a _move constructor_, or may not be moveable at all!

Even more alarming, C++ types are permited to take the address of
the location where they are being constructed: the `this` pointer
is always accessible, allowing easy creation of self-referential
types:

```cpp highlight
class Cyclic {
 public:
  Cyclic() {}

  // Ensure that copy and move construction respect the self-pointer
  // invariant:
  Cyclic(const Cyclic&) {
    new (this) Cyclic;
  }
  // snip: Analogous for other rule-of-five constructors.

 private:
  Cyclic* ptr_ = this;
};

```

[C++](#code:3)

The solution \[ `cxx`\] and other FFI strategies take is to box up
complex C++ objects across the FFI boundary; a
`std::unique_ptr<Cyclic>` (perhaps reinterpreted as a `Box` on the
Rust side) can be passed around without needing to call move
constructors. The heap allocation is a performance regression that
scares off potential Rust users, so it’s not a viable solution.

We can do better.

### [Notation and Terminology](#notation-and-terminology)

“Move” is a very, very overloaded concept across Rust and C++, and
many people have different names for what this means. So that we
don’t get confused, we’ll establish some terminology to use
throughout the rest of the article.

A _destructive move_ is a Rust-style move, which has the following
properties:

- It does not create a new object; from the programmer’s
  perspective, the object has simply changed address.
- The move is implemented by a call to `memcpy`; no user code is
  run.
- The moved-from value becomes inaccessible and its destructor
  does not run.

A destructive move, in effect, is completely invisible to the
user[4](#fn:4), and the Rust compiler can emit as many or as few
of them as it likes. We will refer to this as a “destructive
move”, a “Rust move”, or a “blind, `memcpy` move”.

A _copying move_ is a C++-style move, which has the following
properties:

- It creates a new, distinct object at a new memory location.
- The move is implemented by calling a user-provided function that
  initializes the new object.
- The moved-from value is still accessible but in an “unspecified
  but valid state”. Its destructor is run once the current scope
  ends.

A copying move is just a weird copy operation that mutates the
copied-from object. C++ compilers _may_ elide calls to the move
constructor in certain situations, but calling it usually requires
the programmer to explicitly ask for it. From a Rust perspective,
this is as if `Clone::clone()` took `&mut self` as an argument. We
will refer to this as a “copying move”, a “nondestructive move”, a
“C++ move”, or, metonymically, as a “move constructor”.

## [Pinned Pointers](#pinned-pointers)

As part of introducing support for stackless coroutines[5](#fn:5)
(aka `async`/ `await`), Rust had to provide some kind of supported
for immobile types through _pinned pointers_.

The \[ `Pin`\] type is a wraper around a pointer type, such as
`Pin<&mut i32>` or `Pin<Box<ComplexObject>>`. `Pin` provides the
following guarantee to unsafe code:

> [reference](#ref:1)
>
> Given `p: Pin<P>` for `P: Deref`, and `P::Target: !Unpin`, the
> pointee object `*p` will always be found at that address, and no
> other object will use that address until `*p`’s destructor is
> called.

In a way, `Pin<P>` is a witness to a sort of critical section:
once constructed, that memory is _pinned_ until the destructor
runs. The `Pin` documentation goes into deep detail about when and
why this matters, and how unsafe code can take advantage of this
guarantee to provide a safe interface.

The key benefit is that unsafe code can create self-references
behind the pinned pointer, without worrying about them breaking
when a destructive move occurs. C++ deals with this kind of type
by allowing move/copy constructors to observe the new object’s
address and fix up any self references as necessary.

Our progress so far: C++ types can be immoveable from Rust’s
perspective. They need to be pinned in some memory location:
either on the heap as a `Pin<Box<T>>`, or on the stack (somehow;
keep reading). Our program is now to reconcile C++ move
constructors with this standard library object that explicitly
prevents moves. Easy, right;

## [Constructors](#constructors)

C++ constructors are a peculiar thing. Unlike Rust’s
`Foo::new()`-style factories, or even constructors in dynamic
languages like Java, C++ constructors are unique in that they
construct a value in a _specific location_. This concept is best
illustraced by the
[_placement- `new`_](https://en.cppreference.com/w/cpp/language/new#Placement_new)
operation:

```cpp highlight
void MakeString(std::string* out) {
  new (out) std::string("mwahahaha");
}

```

[C++](#code:4)

Placement- `new` is one of those exotic C++ operations you only
ever run into deep inside fancy library code. Unlike `new`, which
triggers a trip into the allocator, placement- `new` simply calls
the constructor of your type with `this` set to the argument in
parentheses. This is the “most raw” way you can call a
constructor: given a memory location and arguments, construct a
new value.

In Rust, a method call `foo.bar()` is really syntax sugar for
`Foo::bar(foo)`. This is not the case in C++; a member function
has an altogether different type, but some simple template
metaprogramming can flatten it back into a regular old free
function:

```cpp highlight
class Foo {
  int Bar(int x);
};

inline int FreeBar(Foo& foo, int x) {
  return foo.Bar();
}

Foo foo;
FreeBar(foo, 5);

```

[C++](#code:5)

Placement- `new` lets us do the analogous thing for a constructor:

```cpp highlight
class Foo {
  Foo(int x);
}

inline void FreeFoo(Foo& foo, int x) {
  new (&foo) Foo(x);
}

Foo* foo = AllocateSomehow();
FreeFoo(*foo, 5);

```

[C++](#code:6)

We can lift this “flattening” of a specific constructor into Rust,
using the existing vocabulary for pushing fixed-address memory
around:

```rust highlight
unsafe trait Ctor {
  type Output;
  unsafe fn ctor(self, dest: Pin<&mut MaybeUninit<Self::Output>>);
}

```

[Rust](#code:7)

A `Ctor` is a _constructing closure_. A `Ctor` contains the
necessary information for constructing a value of type `Output`
which will live at the location `*dest`. The `Ctor::ctor()`
function performs in-place construction, making `*dest` become
initialized.

A `Ctor` is not the constructor itself; rather, it is more like a
`Future` or an `Iterator` which contain the necessary captured
values to perform the operation. A Rust type that is constructed
using a `Ctor` would have functions like this:

```rust highlight
impl MyType {
  fn new() -> impl Ctor<Output = Self>;
}

```

[Rust](#code:8)

The `unsafe` markers serve distinct purposes:

- It is an `unsafe trait`, because `*dest` must be initialized
  when `ctor()` returns.
- It has an `unsafe fn`, because, in order to respect the
  [`Pin` drop guarantees](https://doc.rust-lang.org/std/pin/index.html#drop-guarantee),
  `*dest` must either be freshly allocated or have had its
  destructor run just prior.

Since we are constructing into `Pin` ned memory, the `Ctor`
implementation can use the address of `*dest` as part of the
construction procedure and assume that that pointer will not
suddenly dangle because of a move. This recovers our C++ behavior
of “ `this`-stability”.

Unfortunately, `Ctor` is covered in `unsafe`, and doesn’t even
allocate storage for us. Luckily, it’s not too hard to build our
own safe `std::make_unique`:

```rust highlight
fn make_box<C: Ctor>(c: C) -> Pin<Box<Ctor::Output>> {
  unsafe {
    type T = Ctor::Output;
    // First, obtain uninitialized memory on the heap.
    let uninit = std::alloc::alloc(Layout::new<T>());

    // Then, pin this memory as a MaybeUninit. This memory
    // isn't going anywhere, and MaybeUninit's entire purpose
    // in life is being magicked into existence like this,
    // so this is safe.
    let pinned = Pin::new_unchecked(
      &mut *uninit.cast::<MaybeUninit<T>>()
    );
    // Now, perform placement-`new`, potentially FFI'ing into
    // C++.
    c.ctor(pinned);

    // Because Ctor guarantees it, `uninit` now points to a
    // valid `T`. We can safely stick this in a `Box`. However,
    // the `Box` must be pinned, since we pinned `uninit`
    // earlier.
    Pin::new_unchecked(Box::from_raw(uninit.cast::<T>()))
  }
}

```

[Rust](#code:9)

Thus, `std::make_unique<MyType>()` in C++ becomes
`make_box(MyType::new())` in Rust. `Ctor::ctor` gives us a
bridging point to call the C++ constructor from Rust, in a context
where its expectations are respected. For example, we might write
the following binding code:

```cpp highlight
class Foo {
  Foo(int x);
}

// Give the constructor an explicit C ABI, using
// placement-`new` to perform the "rawest" construction
// possible.
extern "C" FooCtor(Foo* thiz, int x) {
  new (thiz) Foo(x);
}

```

[foo.cc](#code:foo.cc)

```rust highlight
struct Foo { ... }
impl Foo {
  fn new(x: i32) -> impl Ctor<Output = Self> {
    unsafe {
      // Declare the placement-new bridge.
      extern "C" {
        fn FooCtor(this: *mut Foo, x: i32);
      }

      // Make a new `Ctor` wrapping a "real" closure.
      ctor::from_placement_fn(move |dest| {
        // Call back into C++.
        FooCtor(dest.as_mut_ptr(), x)
      })
    }
  }
}

```

[foo_bindings.rs](#code:foo_bindings.rs)

```rust highlight
use foo_bindings::Foo;

// Lo, behold! A C++ type on the Rust heap!
let foo = make_box(Foo::new(5));

```

[foo_user.rs](#code:foo_user.rs)

But… we’re still on the heap, so we seem to have made no progress.
We could have just called `std::make_unique` on the C++ side and
shunted it over to Rust. In particular, this is what \[ `cxx`\]
resorts to for complex types.

## [Interlude I: Pinning on the Stack](#interlude-i-pinning-on-the-stack)

Creating pinned pointers directly requires a sprinkling of
`unsafe`. `Box::pin()` allows us to safely create a `Pin<Box<T>>`,
since we know it will never move, much like the `make_box()`
example above. However, it’s not possible to create a
`Pin<&mut T>` to not-necessarilly- `Unpin` data as easilly:

```rust highlight
let mut data = 42;
let ptr = &mut data;
let pinned = unsafe {
  // Reborrow `ptr` to create a pointer with a shorter lifetime.
  Pin::new_unchecked(&mut *ptr)
};

// Once `pinned` goes out of scope, we can move out of `*ptr`!
let moved = *ptr;

```

[Rust](#code:10)

The `unsafe` block is necessary because of exactly this situation:
`&mut T` does not own its pointee, and a given mutable reference
might not be the “oldest” mutable reference there is. The
following _is_ a safe usage of this constructor:

```rust highlight
let mut data = 42;
// Intentionally shadow `data` so that no longer-lived reference than
// the pinned one can be created.
let data = unsafe {
  Pin::new_unchecked(&mut *data)
};

```

[Rust](#code:11)

This is such a common pattern in futures code that many futures
libraries provide a macro for performing this kind of pinning on
behalf of the user, such as \[ `tokio::pin!()`\].

With this in hand, we can actually call a constructor on a
stack-pinned value:

```rust highlight
let val = MaybeUninit::uninit();
pin!(val);
unsafe { Foo::new(args).ctor(val); }
let val = unsafe {
  val.map_unchecked_mut(|x| &mut *x.as_mut_ptr())
};

```

[Rust](#code:12)

Unfortunately, we _still_ need to utter a little bit more
`unsafe`, but because of `Ctor`’s guarantees, this is all
perfectly safe; the compiler just can’t guarantee it on its own.
The natural thing to do is to wrap it up in a macro much like
`pin!`, which we’ll call `emplace!`:

```rust highlight
emplace!(let val = Foo::new(args));

```

[Rust](#code:13)

This is _truly_ analogous to C++ stack initialization, such as
`Foo val(args);`, although the type of `val` is `Pin<&mut Foo>`,
whereas in C++ it would merely bee `Foo&`. This isn’t much of an
obstacle, and just means that `Foo`’s API on the Rust side needs
to use `Pin<&mut Self>` for its methods.

## [The Return Value Optimization](#the-return-value-optimization)

Now we go to build our Foo-returning function and are immediately
hit with a roadblock:

```rust highlight
fn make_foo() -> Pin<&'wat mut Foo> {
  emplace!(let val = Foo::new(args));
  foo
}

```

[Rust](#code:14)

What is the lifetime `'wat`? This is just returning a pointer to
the current stack frame, which is no good. In C++ (ignoring fussy
defails about move semantics), NRVO would kick in and `val` would
be constructed “in the return slot”:

```cpp highlight
Foo MakeFoo() {
  Foo val(args);
  return val;
}

```

[C++](#code:15)

[_Return value optimization_](https://en.cppreference.com/w/cpp/language/copy_elision)
(and the related _named return value_ _optimization_) allow C++ to
elide copies when constructing return values. Instead of
constructing `val` on `MakeFoo`’s stack and then copying it into
the ABI’s return location (be that a register like `rax` or
somewhere in the caller’s stack frame), the value is constructed
directly in that location, skipping the copy. Rust itself performs
some limited RVO, though its style of move semantics makes this a
bit less visible.

Rust does not give us a good way of accessing the return slot
directly, for good reason: it need not have an address! Rust
returns all types that look roughly like a single integer in a
register (on modern ABIs), and registers don’t have addresses. C++
ABIs typically solve this by making types which are “sufficiently
complicated” (usually when they are not trivially moveable) get
passed on the stack unconditionally[6](#fn:6).

Since we can’t get at the return slot, we’ll make our own! We just
need to pass the pinned `MaybeUninit<T>` memory that we would pass
into `Ctor::ctor` as a “fake return slot”:

```rust highlight
fn make_foo(return_slot: Pin<&mut MaybeUninit<Foo>>) -> Pin<&mut Foo> {
  unsafe {
    Foo::new(args).ctor(return_slot);
    val.map_unchecked_mut(|x| &mut *x.as_mut_ptr());
  }
}

```

[Rust](#code:16)

This is such a common operation that it makes sense to replace
`Pin<&mut MaybeUninit<T>>` with a specific type, `Slot<'a, T>`:

```rust highlight
struct Slot<'a, T>(Pin<&'a mut MaybeUninit<T>>);
impl<'a, T> Slot<'a, T> {
  fn emplace<C: Ctor>(c: C) -> Pin<&'a mut T> {
    unsafe {
      c.ctor(self.0);
      val.map_unchecked_mut(|x| &mut *x.as_mut_ptr());
    }
  }
}

fn make_foo(return_slot: Slot<Foo>) -> Pin<&mut Foo> {
  return_slot.emplace(Foo::new(args))
}

```

[Rust](#code:17)

We can provide another macro, `slot!()`, which reserves pinned
space on the stack much like `emplace!()` does, but without the
construction step. Calling `make_foo` only requires minimal
ceremony and no user-level `unsafe`.

```rust highlight
slot!(foo);
let foo = make_foo(foo);

```

[Rust](#code:18)

The `slot!()` macro is almost identical to \[ `tokio::pin!()`\],
except that it doesn’t initialize the stack space with an existing
value.

## [Towards Move Constructors: Copy Constructors](#towards-move-constructors-copy-constructors)

Move constructors involve
[_rvalue references_](https://en.cppreference.com/w/cpp/language/reference#Rvalue_references),
which Rust has no meaningful equivalent for, so we’ll attack the
easier version: copy constructors.

A copy constructor is C++’s `Clone` equivalent, but, like all
constructors, is allowed to inspect the address of `*this`. Its
sole argument is a `const T&`, which has a direct Rust analogue: a
`&T`. Let’s write up a trait that captures this operation:

```rust highlight
unsafe trait CopyCtor {
  unsafe fn copy_ctor(src: &Self, dest: Pin<&mut MaybeUninit<Self>>);
}

```

[Rust](#code:19)

Unlike `Ctor`, we would implement `CopyCtor` on the type with the
copy constructor, bridging it to C++ as before. We can then define
a helper that builds a `Ctor` for us:

```rust highlight
fn copy<T: CopyCtor>(val: &T) -> impl Ctor<Output = T> {
  unsafe {
    ctor::from_placement_fn(move |dest| {
      T::copy_ctor(val, dest)
    })
  }
}

emplace!(let y = copy(x));     // Calls the copy constructor.
let boxed = make_box(copy(y)); // Copy onto the heap.

```

[Rust](#code:20)

We can could (modulo orphan rules) even implement `CopyCtor` for
Rust types that implement `Clone` by `clone` ing into the
destination.

It should be straightforward to make a version for move
construction… but, what’s a `T&&` in Rust?

## [Interlude II: Unique Ownership](#interlude-ii-unique-ownership)

`Box<T>` is interesting, because unlike `&T`, it is possible to
_move out_ of a `Box<T>`, since the compiler treats it somewhat
magically. There has long been a desire to introduce a `DerefMove`
trait captures this behavior, but the difficulty is the signature:
if `deref` returns `&T`, and `deref_mut` returns `&mut T`, should
`deref_move` return `T`? Or something… more exotic? You might not
want to dump the value onto the stack; you want `*x = *y` to not
trigger an expensive intermediate copy, when `*y: [u8; BIG]`.

Usually, the “something more exotic” is a `&move T` or `&own T`
reference that “owns” the pointee, similar to how a `T&&` in C++
is taken to mean that the caller wishes to perform ownership
transfer.

Exotic language features aside, we’d like to be able to implement
something like `DerefMove` for move constructors, since this is
the natural analogue of `T&&`. To move out of storage, we need a
smart pointer to provide us with three things:

- It must actually be a smart pointer (duh).
- It must be possible to destroy the storage without running the
  destructor of the pointee (in Rust, unlike in C++, destructors
  do not run on moved-from objects).
- It must be the _unique owner_ of the pointee. Formally, if, when
  `p` goes out of scope, no thread can access `*p`, then `p` is
  the unique owner.

`Box<T>` trivially satisfies all three of these: it’s a smart
pointer, we can destroy the storage using `std::alloc::dealloc`,
and it satisfies the unique ownership property.

`&mut T` fails both tests: we don’t know how to destory the
storage (this is one of the difficulties with a theoretical
`&move T`) and it is not the unique owner: some `&mut T` might
outlive it.

Interestingly, `Arc<T>` only fails the unique ownership test, and
it can pass it dynamically, if we observe the strong and weak
counts to both be 1. This is also true for `Rc<T>`.

Most importantly, however, is that if `Pin<P>`, then it is
sufficient that `P` satisfy these conditions. After all, a
`Pin<Box<P>>` uniquely owns its contents, even if they can’t be
moved.

It’s useful to introduce some traits that record these
requirements:

```rust highlight
unsafe trait OuterDrop {
  unsafe fn outer_drop(this: *mut Self);
}

unsafe trait DerefMove: DerefMut + OuterDrop {}

```

[Rust](#code:21)

`OuterDrop` is simply the “outer storage destruction” operation.
Naturally, it is only safe to perform this operation when the
pointee’s own destructor has been separately dropped (there are
some subtleties around leaking memory here, but in general it’s
not a good idea to destroy storage without destroying the pointee,
too).

`DerefMove` [7](#fn:7) is the third requirement, which the
compiler cannot check (there’s a lot of these, huh?). Any type
which implements `DerefMove` can be moved out of by carefully
dismantling the pointer:

```rust highlight
fn move_out_of<P>(mut p: P) -> P::Target
where
  P: DerefMove,
  P::Target: Sized + Unpin,
{
  unsafe {
    // Copy the pointee out of `p` (all Rust moves are
    // trivial copies). We need `Unpin` for this to be safe.
    let val = (&mut *p as *mut P::Target).read();

    // Destroy `p`'s storage without running the pointee's
    // destructor.
    let ptr = &mut p as *mut P;
    // Make sure to suppress the actual "complete" destructor of
    // `p`.
    std::mem::forget(p);
    // Actually destroy the storage.
    P::outer_drop(ptr);

    // Return the moved pointee, which will be trivially NRVO'ed.
    val
  }
}

```

[Rust](#code:22)

Much like pinning, we need to lift this capability to the stack
somehow. `&mut T` won’t cut it here.

## [Owning the Stack](#owning-the-stack)

We can already speak of uninitialized but uniquely-owned stack
memory with `Slot`, but `Slot::emplace()` returns a (pinned)
`&mut T`, which cannot be `DerefMove`. This operation actually
loses the uniqueness information of `Slot`, so instead we make
`emplace()` return a `StackBox`.

A `StackBox<'a, T>` is like a `Box<T>` that’s bound to a stack
frame, using a `Slot<'a, T>` as underlying storage. Although it’s
just a `&mut T` on the inside, it augments it with the uniqueness
invariant above. In particular, `StackBox::drop()` is entitled to
call the destructor of its pointee in-place.

To the surprise of no one who has read this far,
`StackBox: DerefMove`. The implementation for
`StackBox::outer_drop()` is a no-op, since the calling convention
takes care of destroying stack frames.

It makes sense that, since `Slot::emplace()` returns a
`Pin<StackBox<T>>`, so should `emplace!()`.

(There’s a crate called
[`stackbox`](https://crates.io/crates/stackbox) that provides
similar `StackBox`/ `Slot` types, although it is implemented
slightly differently and does not provide the pinning guarantees
we need.)

## [Move Constructors](#move-constructors)

This is it. The moment we’ve all be waiting for. Behold, the
definition of a move constructor in Rust:

```rust highlight
unsafe trait MoveCtor {
  unsafe fn move_ctor(
    src: &mut Self,
    dest: Pin<&mut MaybeUninit<Self>>
  );
}

```

[Rust](#code:23)

> Wait, that’s it?

There’s no such thing as `&move Self`, so, much like `drop()`, we
have to use a plain ol’ `&mut` instead. Like `Drop`, and like
`CopyCtor`, this function is not called directly by users;
instead, we provide an adaptor that takes in a `MoveCtor` and
spits out a `Ctor`.

```rust highlight
fn mov<P>(mut ptr: P) -> impl Ctor<Output = P::Target>
where
  P: DerefMove,
  P::Target: MoveCtor,
{
  unsafe {
    from_placement_fn(move |dest| {
      MoveCtor::move_ctor(&mut *ptr, dest);

      // Destroy `p`'s storage without running the pointee's
      // destructor.
      let inner = &mut ptr as *mut P;
      mem::forget(ptr);
      P::outer_drop(inner);
    })
  }
}

```

[Rust](#code:24)

Notice that we no longer require that `P::Target: Unpin`, since
the `ptr::read()` call from `move_out_of()` is now gone. Instead,
we need to make a specific requirement of `MoveCtor` that I will
explain shortly. However, we can now freely call the move
constructor just like any other `Ctor`:

```rust highlight
emplace!(let y = mov(x));  // Calls the move constructor.
let boxed = make_box(mov(y));  // Move onto the heap.

```

[Rust](#code:25)

## [The Langauge-Lawyering Part](#the-langauge-lawyering-part)

(If you don’t care for language-lawyering, you can skip this
part.)

Ok. We need to justify the loss of the `P::Target: Unpin` bound on
`mov()`, which seems almost like a contradiction: `Pin<P>`
guarantees its pointee won’t be moved, but isn’t the whole point
of `MoveCtor` to perform moves?

At the begining of this article, I called out the difference
between destructive Rust move and copying C++ moves. The reason
that the above isn’t a contradiction is that the occurences of
“move” in that sentence refer to these different senses of “move”.

The specific thing that `Pin<P>` is protecting unsafe code from is
whatever state is behind the pointer being blindly `memcpy` moved
to another location, leaving any self-references in the new
location dangling. However, by invoking a C++-style move
constructor, the data never “moves” in the Rust sense; it is
merely copied in a way that carefully preserves any
address-dependent state.

We need to ensure two things:

- Implementors of `MoveCtor` for their own type must ensure that
  their type does not rely on any pinning guarantees that the move
  constructor cannot appropriately “fix up”.
- No generic code can hold onto a reference to moved-from state,
  because that way they could witness whatever messed-up
  post-destruction state the move constructor leaves it in.

The first of these is passed onto the implementor as an
`unsafe impl` requirement. Designing an `!Unpin` type by hand is
difficult, and auto-generated C++ bindings using this model would
hopefully inherit move-correctness from the C++ code itself.

The second is more subtle. In the C++ model, the moved-from value
is mutated to mark it as “moved from”, which usually just inhibits
the destructor. C++ believes all destructors are run for all
objects. For example, `std::unique_ptr` sets the moved-from value
to `nullptr`, so that the destructor can be run at the end of
scope and do nothing. Compare with the Rust model, where the
compiler inhibits the destructor automatically through the use of
drop flags.

In order to support move-constructing both Rust and C++ typed
through a uniform interface, `move_ctor` is a fused
destructor/copy operation. In the Rust case, no “destructor” is
run, but in the C++ case we are required to run a destructor.
Although this changes the semantic ordering of destruction
compared to the equivalent C++ program, in practice, no one
depends on moved-from objects actually being destroyed (that I
know of).

After `move_ctor` is called, `src` must be treated as if it had
just been destroyed. This means that the storage for `src` must be
disposed of immediately, without running any destructors for the
pointed-to value. Thus, no one must be able to witness the
messed-up pinned state, which is why `mov()` requires
`P: DerefMove`.

Thus, no code currently observing `Pin<P>` invariants in unsafe
code will notice anything untoward going on. No destructive moves
happen, and no moved-from state is able to hang around.

---

1. This isn’t exactly a _universal_ opinion _glances at Swift_ but
   it is if you write kernel code like me. [↩︎](#fnref:1)

2. You can’t just have rustc consume a `.h` and spit out bindings,
   like e.g. Go can, but it’s better than the disaster that is
   JNI. [↩︎](#fnref:2)

3. Some WG21 folks have tried to introduce a weaker type-trait,
   `std::is_trivially_relocatable`, which is a weakening of
   trivally moveable that permits a Rust-style destructive move.
   The libc++ implementation of most STL types, like
   `std::unique_ptr`, admit this trait. [↩︎](#fnref:3)

4. A lot of unsafe Rust code assumes this is the only kind of
   move. For example,
   [`mem::swap()`](https://doc.rust-lang.org/std/mem/fn.swap.html)
   is implemented using `memcpy`. This is unlike the situation in
   C++, where types will often provide custom `std::swap()`
   implementations that preserve type invariants. [↩︎](#fnref:4)

5. Because `Future` objects collapse their stack state into
   themselves when yielding, they may have pointers into
   themselves (as a stack typically does). Thus, `Future` s need
   to be guaranteed to never move once they begin executing, since
   Rust has no move constructors and no way to fix up the
   self-pointers. [↩︎](#fnref:5)

6. Among other things, this means that `std::unique_ptr` s are
   passed on the stack, not in a register, which is very wasteful!
   Rust’s `Box` does not have this issue. [↩︎](#fnref:6)

7. Rust has attempted to add something like `DerefMove` many
   times. What’s described in this post is nowhere near as
   powerful as a “real” `DerefMove` would be, since such a thing
   would also allow moving _into_ a memory location. [↩︎](#fnref:7)
