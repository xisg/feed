---
title: snuffle / salsa / chacha
url: https://graydon2.dreamwidth.org/319755.html
published: "2025-08-29T20:12:33Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:319755
---

# snuffle / salsa / chacha

This is a small note about a delightful function. Not cryptography advice or serious commentary. Just amusement.

A couple years back I had occasion to read in slightly more detail than I had before about the state of the art in cryptographically secure PRNGs (CSPRNGs). These are PRNGs we trust to have additional properties beyond the speed and randomness requirements of normal ones -- inability for an attacker to reveal internal state, mainly, so you can use them to generate secrets.

If you look, you'll find a lot of people recommending something based on one of Dan Bernstein's algorithms: [Salsa20 or ChaCha (or even more obscurely "Snuffle")](https://en.wikipedia.org/wiki/Salsa20). All the algorithms we're discussing here are very similar in design, and vary only in minor details of interest only to cryptographers.

If you follow that link though, you'll notice it's a description of a (symmetric) stream cipher. Not a CSPRNG at all!

But that's ok! Because it turns out that people have long known an interesting trick -- actually more of a construction device? -- which is that a CSPRNG "is" a stream cipher. Or rather, if you hold it the other way, you might even say a stream cipher "is" just a CSPRNG. Many stream ciphers are built by deriving an unpredictable "key stream" off the key material and then just XOR'ing it with the plaintext. So long as the "key stream" is unpredictable / has unrecoverable state, this is sufficient; but it's the same condition we want out of the stream of numbers coming out of a CSPRNG, just with "seed" standing in for "key". They're fundamentally the same object.

I knew all this before, so people naming a CSPRNG and a stream cipher the same did not come as any surprise to me. But I went and looked a little further into ChaCha in particular (and its ancestor Salsa and, earlier still, Snuffle) because they have one additional cool and weird property.

They are _seekable_.

This means that you can, with O(1) effort, "reposition" the Snuffle/Salsa/ChaCha "key stream" / CSPRNG number stream to anywhere in its future. You want the pseudorandom bytes for block 20,000,000? No problem, just "set the position" to 20,000,000 and it will output those bytes. This is _not_ how all CSPRNGs or stream ciphers work. But some do. ChaCha does! Which is very nice. It makes it useful for all sorts of stuff, especially things like partially decrypting randomly-read single blocks in the middle of large files.

I got to wondering about this, so I went back and read through design docs on it, and I discovered something surprising (to me): it's not just a floor wax and dessert topping CSPRNG and stream cipher. ChaCha is also a cryptographic hash function (CHF)! Because a CHF is _also_ something you can build a CSPRNG out of, and therefore also build a stream cipher out of. They're _all the same object_.

How does the construction work? Embarassingly easily. You put the key material and a counter (and enough fixed nonzero bits to make the CHF happy) in an array and hash it. That's it. The hash output is your block of data. For the next block, you increment the counter and hash again. Want block 20,000,000? Set the counter to 20,000,000. The CHF's one-way-function-ness implies the non-recoverability of the key material and its mixing properties ensure that bumping the counter is enough to flip lots of bits. The end.

Amazing!

But then I got curious and dug a bit into the origins of ChaCha and .. stumbled on something hilarious. In the earliest design doc I could find ( [Salsa20 Design](https://cr.yp.to/snuffle/design.pdf) which still refers to it as "Snuffle 2005") the introduction starts with this:

> Fifteen years ago, the United States government was trying to stop publication
>
> of new cryptographic ideas—but it had made an exception for cryptographic
>
> hash functions, such as Ralph Merkle’s new Snefru.
>
> This struck me as silly. I introduced Snuffle to point out that one can easily
>
> use a strong cryptographic hash function to efficiently encrypt data.
>
> Snuffle 2005, formally designated the “Salsa20 encryption function,” is the
>
> latest expression of my thoughts along these lines. It uses a strong cryptographic
>
> hash function, namely the “Salsa20 hash function,” to efficiently encrypt data.
>
> This approach raises two obvious questions. First, why did I choose this
>
> particular hash function? Second, now that the United States government seems
>
> to have abandoned its asinine policies, why am I continuing to use a hash function
>
> to encrypt data?

In other words: the cool seekability wasn't a design goal. Shuffle/Salsa/ChaCha was intended as a tangible demonstration of a _political argument_ that it's stupid to regulate one of the 3 objects (CHF, CSPRNG and stream cipher) since you can build them all out of the CHF. (And, I guess, "obviously you should be allowed to export CHFs" though I wouldn't bet on anything being obvious to the people who make such decisions).

And then I googled more and realized that when I was a teenager I had completely missed all the drama / failed to connect the dots. Snuffle was the subject of [Bernstein v. United States](https://en.wikipedia.org/wiki/Bernstein_v._United_States), the case that overturned US export restrictions on cryptography altogether! And as [this page](https://cryptography.fandom.com/wiki/Snuffle) points out "the subject of the case, Snuffle, was itself an attempt to bypass the regulations".

Anyway, I thought this was both wonderful and funny: both the CHF-to-CSPRNG construction (which I'd never understood/seen before), but also the fact that Snuffle/Salsa/ChaCha is like the ultimate case of winning big in cryptography. Not only does ChaCha now transport like 99%\[EDIT "double-digit percentages"\] of the world's internet traffic (it's become the standard we all use because it's fast and secure) but that it was pivotal in the evolution of the legal landscape and all arises from a sort of neener-neener assessment that the law at the time was internally inconsistent / contained a loophole for CHFs that made the whole thing "asinine".

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=319755) comments
