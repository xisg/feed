---
title: By how much does your memory allocator overallocate?
url: https://lemire.me/blog/2025/12/30/by-how-much-does-your-memory-allocator-overallocates/
published: "2025-12-30T19:15:55Z"
feed: lemire
guid: https://lemire.me/blog/?p=22414
---

# By how much does your memory allocator overallocate?

![](https://lemire.me/blog/wp-content/uploads/2025/12/plot-150x150.png)

How much virtual memory does the following C++ expression allocate on the heap?

```
new char[4096]

```

The answer is _at least 4 kibibytes_ but surely more.

Firstly, each heap memory allocation requires some memory to keep track of what has been allocated. You are likely using 8 bytes or so of overhead that your program cannot access.

Secondly, the memory allocator may allocate a bit more than the 4096 bytes you requested. On a Linux machine, I found that it would allocate 4104 bytes, so 8 extra bytes that are usable by your program. You can check this value by calling `malloc_usable_size` under Linux.

Thus, overall, you may end up with an extra 16 bytes allocated when you requested 4096 bytes. It is an overhead of about 0.4%. You are basically _wasting_ a byte for every 256 bytes that you allocate.

But that is not the worst possible case. On macOS, let us consider the following line of code.

```
new char[3585]

```

The system reports an allocation of 4096 bytes: a 14% overhead. What is happening is that macOS rounds up the memory allocation to the nearest 512 byte boundary for moderately small allocations. If you try allocating even larger memory blocks, it starts rounding up even more.

[![](http://lemire.me/blog/wp-content/uploads/2025/12/plot.png)](http://lemire.me/blog/wp-content/uploads/2025/12/plot.png)
