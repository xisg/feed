---
title: Type resolution redesign, with language changes to taste
url: https://ziglang.org/devlog/2026/#2026-03-10
published: "2026-03-10T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2026/#2026-03-10
---

# [Type resolution redesign, with language changes to taste](\#2026-03-10)

Author: Matthew Lugg

Today, [I merged a 30,000 line PR](https://codeberg.org/ziglang/zig/pulls/31403) after two (arguably three) months of work. The goal of this branch was to rework the Zig compiler’s internal type resolution logic to a more logical and straightforward design. It’s a quite exciting change for me personally, because it allowed me to clean up a bunch of the compiler guts, but it also has some nice user-facing changes which you might be interested in!

For one thing, the Zig compiler is now lazier about analyzing the fields of types: if the type is never initialized, then there’s no need for Zig to care what that type “looks like”. This is important when you have a type which doubles as a namespace, a common pattern in modern Zig. For instance, when using `std.Io.Writer`, you don’t want the compiler to also pull in a bunch of code in `std.Io`! Here’s a straightforward example:

```zig
const Foo = struct {
    bad_field: @compileError("i am an evil field, muahaha"),
    const something = 123;
};
comptime {
    _ = Foo.something; // `Foo` only used as a namespace
}

```

Previously, this code emitted a compile error. Now, it compiles just fine, because Zig never actually looks at the `@compileError` call.

Another improvement we’ve made is in the “dependency loop” experience. Anyone who has encountered a dependency loop compile error in Zig before knows that the error messages for them are entirely unhelpful—but that’s now changed! If you encounter one (which is also a bit less likely now than it used to be), you’ll get a detailed error message telling you exactly where the dependency loop comes from. Check it out:

```zig
const Foo = struct { inner: Bar };
const Bar = struct { x: u32 align(@alignOf(Foo)) };
comptime {
    _ = @as(Foo, undefined);
}

```

```
$ zig build-obj repro.zig
error: dependency loop with length 2
    repro.zig:1:29: note: type 'repro.Foo' depends on type 'repro.Bar' for field declared here
    const Foo = struct { inner: Bar };
                                ^~~
    repro.zig:2:44: note: type 'repro.Bar' depends on type 'repro.Foo' for alignment query here
    const Bar = struct { x: u32 align(@alignOf(Foo)) };
                                               ^~~
    note: eliminate any one of these dependencies to break the loop

```

Of course, dependency loops can get much more complicated than this, but in every case I’ve tested, the error message has had enough information to easily see what’s going on.

Additionally, this PR made big improvements to the Zig compiler’s “incremental compilation” feature. The short version is that it fixed a huge amount of known bugs, but in particular, “over-analysis” problems (where an incremental update did more work than should be necessary, sometimes by a big margin) should finally be all but eliminated—making incremental compilation significantly faster in many cases! If you’ve not already, consider [trying out incremental compilation](https://ziglang.org/download/0.15.1/release-notes.html#Incremental-Compilation): it really is a lovely development experience. This is for sure the improvement which excites me the most, and a large part of what motivated this change to begin with.

There are a bunch more changes that come with this PR—dozens of bugfixes, some small language changes (mostly fairly niche), and compiler performance improvements. It’s far too much to list here, but if you’re interested in reading more about it, you can take a look at [the PR](https://codeberg.org/ziglang/zig/pulls/31403) on Codeberg—and of course, if you encounter any bugs, please do open an issue. Happy hacking!
