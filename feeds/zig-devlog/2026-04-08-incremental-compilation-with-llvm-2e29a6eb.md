---
title: Incremental compilation with LLVM
url: https://ziglang.org/devlog/2026/#2026-04-08
published: "2026-04-08T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2026/#2026-04-08
---

# [Incremental compilation with LLVM](\#2026-04-08)

Author: Matthew Lugg

I’ve been spending a bit of time working on personal projects after merging my [type resolution changes](#2026-03-10) last month, but I did find the time recently to make some improvements to the LLVM codegen backend. This involved a few different enhancements with various goals, but one nice user-facing change was that I managed to get incremental compilation working with the LLVM backend.

Sadly this can’t do anything to speed up the dreaded LLVM Emit Object: that time is entirely down to LLVM. However, what incremental compilation _does_ help with is minimizing the time spent in the actual Zig compiler code, which means that if your code has compile errors (so “LLVM Emit Object” will be skipped), you’ll usually get those errors very quickly. (Of course, it does still give you a _slight_ speed-up in successful builds too.)

This support is available in master branch builds right now, and will be in the 0.16.0 release (which we’ll be tagging very soon).

For anyone who still hasn’t tried it, especially if you’re using Zig’s master branch, please do try out incremental compilation by passing `-fincremental --watch` to `zig build`! The Zig core team have benefited from incremental compilation in our workflows for a good year now, and we’re also hearing good things from users. The feature is relatively stable at this point, and people are often surprised how much time they can save just by getting up-to-date compile errors in milliseconds rather than seconds.

I haven’t really personally used incremental compilation with the LLVM backend, but all of the incremental test coverage in CI is now enabled for the LLVM backend, and I’ve had positive feedback from users, so it’s definitely worth giving a shot. As always, if you encounter bugs in incremental compilation, please report them if you can!

Thank you, and I hope you find this useful :)
