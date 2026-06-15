---
title: Two Memory Bugs From Ringbahn
url: https://without.boats/blog/two-memory-bugs-from-ringbahn/
published: "2020-06-10T00:00:00Z"
feed: boats
guid: https://without.boats/blog/two-memory-bugs-from-ringbahn/
---

# Two Memory Bugs From Ringbahn

While implementing [ringbahn](https://github.com/withoutboats/ringbahn), I introduced at least two bugs that caused memory safety
errors, resulting in segfaults, allocator aborts, and bizarre undefined behavior. I’ve fixed both
bugs that I could find, and now I have no evidence that there are more memory safety issues in the
current codebase (though that doesn’t mean there aren’t, of course). I wanted to write about both of
these bugs, because they had an interesting thing in common: they were both caused by destructors.
