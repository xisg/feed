---
title: The byte order fallacy
url: https://commandcenter.blogspot.com/2012/04/byte-order-fallacy.html
published: "2012-04-04T05:22:00Z"
feed: robpike
guid: tag:blogger.com,1999:blog-6983287.post-204147499802585288
---

# The byte order fallacy

Whenever I see code that asks what the native byte order is, it's almost certain the code is either wrong or misguided. And if the native byte order really does matter to the execution of the program, it's almost certain to be dealing with some external software that is either wrong or misguided. If your code contains #ifdef BIG\_ENDIAN or the equivalent, you need to unlearn about byte order.

The byte order of the computer doesn't matter much at all except to compiler writers and the like, who fuss over allocation of bytes of memory mapped to register pieces. Chances are you're not a compiler writer, so the computer's byte order shouldn't matter to you one bit.

Notice the phrase "computer's byte order". What _does_ matter is the byte order of a peripheral or encoded data stream, but--and this is the key point--the byte order of the computer doing the processing is irrelevant to the processing of the data itself. If the data stream encodes values with byte order B, then the algorithm to decode the value on computer with byte order C should be about B, _not about the relationship between B and C_.

Let's say your data stream has a little-endian-encoded 32-bit integer. Here's how to extract it (assuming unsigned bytes):

i = (data\[0\]<<0) \| (data\[1\]<<8) \| (data\[2\]<<16) \| (data\[3\]<<24);

If it's big-endian, here's how to extract it:

i = (data\[3\]<<0) \| (data\[2\]<<8) \| (data\[1\]<<16) \| (data\[0\]<<24);

Both these snippets work on any machine, independent of the machine's byte order, independent of alignment issues, independent of just about anything. They are totally portable, given unsigned bytes and 32-bit integers.

What you might have expected to see for the little-endian case was something like

i = \*((int\*)data);

#ifdef BIG\_ENDIAN

/\\* swap the bytes \*/

i = ((i&0xFF)<<24) \| (((i>>8)&0xFF)<<16) \|   (((i>>16)&0xFF)<<8) \| (((i>>24)&0xFF)<<0);

#endif

or something similar. I've seen code like that many times. Why not do it that way? Well, for starters:

1. It's more code.
2. It assumes integers are addressable at any byte offset; on some machines that's not true.
3. It depends on integers being 32 bits long, or requires more #ifdefs to pick a 32-bit integer type.
4. It may be a little faster on little-endian machines, but not much, and it's slower on big-endian machines.
5. If you're using a little-endian machine when you write this, there's no way to test the big-endian code.
6. It swaps the bytes, a sure sign of trouble (see below).

By contrast, my version of the code:

1. Is shorter.
2. Does not depend on alignment issues.
3. Computes a 32-bit integer value regardless of the local size of integers.
4. Is equally fast regardless of local endianness, and fast enough (especially on modern processsors) anyway.
5. Runs the same code on all computers: I can state with confidence that if it works on a little-endian machine it will work on a big-endian machine.
6. Never "byte swaps".

In other words, it's simpler, cleaner, and utterly portable. There is no reason to ask about local byte order when about to interpret an externally provided byte stream.

I've seen programs that end up swapping bytes two, three, even four times as layers of software grapple over byte order. In fact, byte-swapping is the surest indicator the programmer doesn't understand how byte order works.

Why do people make the byte order mistake so often? I think it's because they've seen a lot of bad code that has convinced them byte order matters. "Here comes an encoded byte stream; time for an #ifdef." In fact, C may be part of the problem: in C it's easy to make byte order look like an issue. If instead you try to write byte-order-dependent code in a type-safe language, you'll find it's very hard. In a sense, byte order only bites you when you cheat.

There's plenty of software that demonstrates the byte order fallacy is really a fallacy. The entire Plan 9 system ran, without architecture-dependent #ifdefs of any kind, on dozens of computers of different makes, models, and byte orders. I promise you, your computer's byte order doesn't matter even at the level of the operating system.

And there's plenty of software that demonstrates how easily you can get it wrong. Here's one example. I don't know if it's still true, but some time back Adobe Photoshop screwed up byte order. Back then, Macs were big-endian and PCs, of course, were little-endian. If you wrote a Photoshop file on the Mac and read it back in, it worked. If you wrote it on a PC and tried to read it on a Mac, though, it wouldn't work unless back on the PC you checked a button that said you wanted the file to be readable on a Mac. (Why wouldn't you? Seriously, why wouldn't you?) Ironically, when you read a Mac-written file on a PC, it always worked, which demonstrates that someone at Adobe figured out something about byte order. But there would have been no problems transferring files between machines, and no need for a check box, if the people at Adobe wrote proper code to encode and decode their files, code that could have been identical between the platforms. I guarantee that to get this wrong took far more code than it would have taken to get it right. \[Note added in 2013: I'm told by folks at Adobe that the option was for TIFF files and only needed for third-party plugins. That doesn't explain why it was PC-only or necessary at all. Adobe might not be the right culprit but the issue was real.\]

Just last week I was reviewing some test code that was checking byte order, and after some discussion it turned out that there was a byte-order-dependency bug in the code being tested. As is often the case, the existence of byte-order-checking was evidence of the presence of a bug. Once the bug was fixed, the test no longer cared about byte order.

And neither should you, because byte order doesn't matter.
