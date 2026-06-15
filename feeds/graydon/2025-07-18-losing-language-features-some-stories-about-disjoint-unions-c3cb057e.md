---
title: 'losing language features: some stories about disjoint unions'
url: https://graydon2.dreamwidth.org/318788.html
published: "2025-07-18T08:09:38Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:318788
---

# losing language features: some stories about disjoint unions

A long time ago I wrote on twitter (now erased): "surprising how much computer stuff makes sense viewed as tragic deprivation of sum types".

[Sum types (a.k.a. disjoint unions a.k.a. tagged unions a.k.a. safe variant types)](https://en.wikipedia.org/wiki/Tagged_union) are of course wonderful and nice and everyone should have them. ML and Haskell have them, and now Rust has them, as does Swift, and Scala with case classes, and C++ kinda with std::variant, and .. lots of modern languages are growing them. Great, hurrah.

But there is just a _little bit of subtlety_ to doing them right. The subtlety is that you have to build the language to _couple together_ two pieces of data -- a tag (or "discriminant") and an unsafe union -- with _mandatory syntactic constructs_ (switch or match expressions) so that you only get to access the unsafe union elements _when you've already checked the tag_, and the type system and name resolution systems know you're in a given case-block, and so only let you access to the union-fields (properly typed) corresponding to the tag case. It's not a hugely complex thing to get right, but you have to get it right.

There are three main degenerate ways you can fail to get it right.

2. You can give users _syntactically unguarded_ access to union members, say by using `container.field` syntax, in which case all you can do if the tag doesn't match that field at runtime is to raise a runtime error, which you can at least _do_ systematically, but the ergonomics are lousy: it's inefficient (you wind up checking twice) and it doesn't help the user avoid the runtime error by statically forcing cases to be handled.

4. You can do #1 but then _also fail to even raise a runtime error_ when the tag is wrong. At which point the tag is basically "advisory", so then...

6. You can even go a step further than #2 and not even require users to declare a tag. Just let the user "know the right case" using some unspecified method, an invariant they have to track themselves. They can use a tag if they like, or some bits hidden somewhere else, who knows.

Ok so .. Casey Muratori gave a great recent talk on the origin of, well, certain OOP-y habits of encapsulation, which is mostly about [Entity-Component-System](https://en.wikipedia.org/wiki/Entity_component_system) organization, but .. also a bit about sum types. He spent a lot of time digging in the PL literature and reconstructing the history of idea transmission. I just watched it, and it's a great talk and you should go watch it, it's here:

One of the things he discusses in here is that safe and correctly-designed disjoint unions aren't just an ML thing, they were around in the early 60s at least. Wikipedia thinks Algol 68. Muratori places their origin in Doug Ross and/or Tony Hoare talking about amendments to Algol 60, linking to [this Tony Hoare paper](https://archive.computerhistory.org/resources/text/Knuth_Don_X4100/PDF_index/k-9-pdf/k-9-u2293-Record-Handling-Hoare.pdf) but of course Hoare references PL/I there and I'm honestly not sure about the _exact_ origin, it's somewhere around there. Point being it predates ML by probably a decade.

But another thing Muratori points out is that is that Dahl and Nygaard _copied the feature in safe working form_ into Simula, and Stroustrup knew about it and _intentionally dropped it_ from C++, thinking it inferior to the encapsulation you get from inheritance. This is funny! Because of course C already had case #3 above -- completely unchecked/unsafe unions, they only showed up in 1976 C, goodness knows why they decided on that -- and the safe(ish) std::variant type has taken forever to regrow in C++.

I was happy to hear this, because it mirrors to some extent another funny story I have reconstructed from my own digging in the literature. Namely about the language [Mesa](http://www.bitsavers.org/pdf/xerox/parc/techReports/CSL-79-3_Mesa_Language_Manual_Version_5.0.pdf), a Butler Lampson project from PARC, very far ahead of its time too. There's far too much to discuss about Mesa to get into here -- separate compilation, interface/implementation splits, etc. -- but it _did_ also have safe variant types (along with a degenerate form called "computed" which is unsafe). Presumably it picked them up from Algol 68, or Simula 67, or one of probably dozens of languges in the late 60s / early 70s it emerged from. No big deal.

Where that relatively unremarkable fact turns into a funny story is during a "technology transfer" event that happened during Mesa's life: Niklaus Wirth took a sabbatical to PARC, and became quite enamoured with Mesa, and went back home to work on what became Modula and eventually Modula 2. But [Modula 2 had only degenerate variant types!](https://www.modula2.org/reference/recordtypes.php) He copied them not from Mesa, but in the same busted form they existed in Pascal, completely missing that Mesa did them correctly. Modula 2 variants are degenerate case #1 above (if you turn on a special checking compilation mode) otherwise case #2: tags declared but no checking at all! He even writes it up in [the report on Modula 2 and Oberon](https://dl.acm.org/doi/pdf/10.1145/1238844.1238847), criticizing its lack of safety while simultaneously citing all the ways Mesa influenced his design. Evidently not enough.

Anyway, all this is to say: language features are easily broken, mis-copied, forgotten or intentionally omitted due to the designer's pet beliefs. Progress is very circuitous, if it exists at all!

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=318788) comments
