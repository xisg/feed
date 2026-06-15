---
title: How fast do browsers correct UTF-16 strings?
url: https://lemire.me/blog/2026/02/21/how-fast-do-browsers-correct-utf-16-strings/
published: "2026-02-21T20:07:17Z"
feed: lemire
guid: https://lemire.me/blog/?p=22443
---

# How fast do browsers correct UTF-16 strings?

![](https://lemire.me/blog/wp-content/uploads/2026/02/Capture-decran-le-2026-02-21-a-15.06.57-150x150.png)

JavaScript represents strings using Unicode, like most programming languages today. Each character in a JavaScript string is stored using one or two 16-bit words. The following JavaScript code might surprise some programmers because a single character becomes two 16-bit words.

```
> t="🧰"
'🧰'
> t.length
2
> t[0]
'\ud83e'
> t[1]
'\uddf0'

```

The convention is that \\uddf0 is the 16-bit value 0xDDF0 also written U+DDF0.

The UTF-16 standard is relatively simple. There are three types of values. high surrogates (the range U+D800 to U+DBFF), low surrogates (U+DC00 to U+DFFF), and all other code units (U+0000–U+D7FF together with U+E000–U+FFFF). A high surrogate must always be followed by a low surrogate, and a low surrogate must always be preceded by a high surrogate.

What happens if you break the rules and have a high surrogate followed by a high surrogate? Then you have an invalid string. We can correct the strings by patching them: we replace the bad values by the replacement character (\\ufffd). The replacement character sometimes appears as a question mark.

To correct a broken string in JavaScript, you can call the toWellFormed method.

```
> t = '\uddf0\uddf0'
'\uddf0\uddf0'
> t.toWellFormed()
'��'

```

How fast is it?

I wrote a s [mall benchmark](https://lemire.github.io/browserwellformed/) that you can test online to measure its speed. I use broken strings of various sizes up to a few kilobytes. I run the benchmarks on my Apple M4 processor using different browsers.

BrowserSpeedSafari 18.61 GiB/sFirefox 1473 GiB/sChrome 14515 GiB/s

Quite a range of performance! The speed of other chromium-based browsers (Brave and Edge) is much the same as Chrome.

I also tested with JavaScript runtimes.

EngineSpeedNode.js v25.5.016 GiB/sBun 1.3.98.4 GiB/s

Usually Bun is faster than Node, but in this instance, Node is twice as far as Bun.

Thus, we can correct strings in JavaScript at over ten gigabytes per second if you use Chromium-based browsers.
