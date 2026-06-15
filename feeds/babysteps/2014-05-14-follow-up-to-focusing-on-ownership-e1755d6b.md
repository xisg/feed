---
title: Follow up to Focusing on Ownership
url: https://smallcultfollowing.com/babysteps/blog/2014/05/14/follow-up-to-focusing-on-ownership/?utm_source=atom_feed
published: "2014-05-14T00:00:00Z"
feed: babysteps
guid: https://smallcultfollowing.com/babysteps/blog/2014/05/14/follow-up-to-focusing-on-ownership/
---

# Follow up to Focusing on Ownership

This post withdrawn: it was posted by accident and was incomplete.

 \| &mut \| ----> \| int \|

```
+---+       +------+       +-----+
               ^
+---+          |
| & | ---------+
+---+

```

As you can see from the diagram, the `&mut` reference _is_ a unique
reference to the integer. That is, it can’t be copied, and it’s the
only _direct_ pointer to that integer. However, there are multiple
paths to the `&mut` itself. That’s not the fault of the `&mut`, it’s
just that uniqueness is a global property. In other words, if I have a
variable `p` of type `&&mut int`, then `**p` is not a _unique path_,
even though it traverses through a unique reference at some point.

_Note:_ the existence of types like `&&mut int` may seem like a wart
on the type system. It is not. It is in fact a very useful pattern, as
I’ll explain below.

### On mutability

It’s also important to emphasize that under no proposal are mutability
and uniqueness _precisely_ conflated. There will always be a
connection (“aliasing, mutability, and safety, pick two”) and hence
the two cannot be completely orthogonal.

In my proposal, the story would not be “a unique reference is required
to mutate”. Rather, it is better to say: “a unique reference is one
way to mutate, the other is `Cell`”. This is (naturally) the same as
today, the only real difference is that we don’t write `&mut` (mutable
reference) but rather just talk about uniqueness.

### Static variables

I neglected to mention how we should treat static variables. To be
honest, I hadn’t thought about them much at all, but I don’t think
there is any significant complication there. My preferred approach
would be to remove `static mut` variables and instead rely on the
existing types for mutating alias state ( `Atomic`, `Cell` etc). There
is a slight twist, though, in that `static` variables are always
accessible to multiple threads.

My preferred solution would be to say that static variables can be
declared as `unsafe`, in which case it is illegal to access them
outside of an unsafe block. Moreover, any static variable that (a)
contains an `Unsafe<T>` instance and (b) is not `Share` _must_ be
declared unsafe. The idea here is that `Unsafe<T>` is the marker we
use to signal “mutability even when aliased” and `Share` is the way we
distinguish types whose APIs guarantee thread-safety. So it’s unsafe
to stick non-thread-safe, inherently mutable data into a static, since
we can’t prevent you from accessing it from multiple threads.

As an aside, the name `Share` for the threadsafe trait doesn’t work so
well with my intention to call `&T` a “shared reference”. Perhaps
`Share` would be better called `Threadsafe`. Not sure.

### On composition

Here is a brief example to explain why a type like `&&mut int` is not
an anti-pattern, but rather something that arises naturally. The
high-level bit is that, when it comes to mutability, `&mut T` behaves
in basically the same way as `T` itself. (The differences between
arise when it comes to moving, or freeing: you can move (or free) a
`T`, but you can’t move the referent of an `&mut T` reference, because
you don’t own it.)

Here is an example that relies on the aliasing of `&&mut T`. Imagine I
want to have a type `FilterMap` that imposes a filter onto another
map, screening out certain keys from being inserted. For fun, let’s
say we don’t want even integers in the map for some reason. I can now
write a type like this:

```
struct FilterMap<'a, V> {
    map: &'a mut HashMap<int, V>
}

```

Now I can implement `insert` like so:

```
impl<'a, V> FilterMap<'a, V> {
    fn insert(&mut self, key: int, value: V) {
        if (key & 1) != 0 { // key is odd
            // Introducing a temporary for explanatory purposes:
            let map = &mut self.map;
            map.insert(key, value);
        }
    }
}

```

Here to do the insertion I create a temporary called `map`. This will
have the type `&mut &mut HashMap<int, V>`. Because every step along
this way is a mutable reference, I wind up with a unique, mutable path
to the HashMap I am delegating to. Great.

Now suppose I wanted to implement `find`. The trick with `find` is
that it will return a pointer into the map itself, so it must ensure
that the map is not mutated until that pointer goes out of scope. We
normally do this by having a signature like this:

```
fn find<'a>(&'a self, key: int) -> Option<&'a V>

```

Here you can see that the input is a shared reference with lifetime
`'a`. Basically this means that the map is aliased

```
impl<'a, V> FilterMap<'a, V> {
    fn find<'a>(&'a self, key: int) -> Option<&'a V> {
        // Introducing a temporary for explanatory purposes:
        let map = &self.map;
        map.find(key)
    }
}

```

Here the temporary `map` has type `&'a &mut HashMap<int, V>`. In other
words, I have a shared reference to the mutable reference. This
implies that the `HashMap` itself is aliased for the lifetime `'a`
(i.e., so long as this shared reference exists). This in turn implies
that the `HashMap` is immutable, and hence we can call `find` as
normal.

–>
