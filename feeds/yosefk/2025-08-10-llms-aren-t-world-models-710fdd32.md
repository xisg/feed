---
title: LLMs aren’t world models
url: https://yosefk.com/blog/llms-arent-world-models.html
published: "2025-08-10T00:00:00Z"
feed: yosefk
---

# LLMs aren’t world models

I believe that language models aren’t world models. It’s a weak claim — I’m not saying they’re useless, or that we’re done
milking them. It’s also a fuzzy-sounding claim — with its trillion weights, who can prove that there’s something an LLM isn't a
model of? But I hope to make my claim clear and persuasive enough with some examples.

A friend who plays better chess than me — and knows more math & CS than me - said that he played some moves against a
newly released LLM, and it must be at least as good as him. I said, no way, I’m going to cRRRush it, in my best Russian accent.
I make a few moves – but unlike him, I don't make good moves [1](#fn1), which would be opening book moves it has seen a million times; I make weak moves, which it
hasn't [2](#fn2). The thing makes decent moves in
response, with cheerful commentary about how we're attacking this and developing that — until about move 10, when it tries to
move a knight which isn't there, and loses in a few more moves. This was a year or two ago; I’ve just tried this again, and it
lost track of the board state by move 9.

When I’m saying that LLMs have no world model, I don’t mean that they haven't seen enough photos of chess knights, or held a
knight in their greasy fingers; I don’t mean the physical world, necessarily. And I obviously don’t mean that a machine can’t
learn a model of chess, when all leading chess engines use machine learning. I only mean that, **having read a trillion**
**chess games, _LLMs_, specifically, have not learned that to make legal moves, you need to know where the pieces are on**
**the board**. Why would they? For predicting the moves or commentary in chess games, which is what they’re optimized for,
this would help very marginally, if at all.

Of course, nobody uses LLMs as chess engines — so whatever they did learn about chess, they learned entirely “by accident”,
without any effort by developers to improve the process for this kind of data. And we could say that the whole argument that
LLMs learn about the world is that they _have_ to understand the world _as a side effect of modeling the distribution_
_of text_ — which is soundly refuted by them literally failing to learn the first thing about chess. But maybe we could
charitably assume that LLMs fail this badly with chess for silly reasons you could easily fix, but nobody bothered. So let’s
look at something virtual enough to learn a model of without having greasy fingers to touch it with, but also relevant enough
for developers to try to make it work.

So, for my second example, we will consider the so-called “normal blending mode” in image editors like [Krita](https://krita.org/) — what happens when you put a layer with some partially transparent pixels on top of another
layer? What’s the mathematical formula for blending 2 layers? An LLM replied roughly like so:

> In Krita Normal blending mode, colors are **not blended using a mathematical formula**. The "Normal" mode
> **simply** displays the upper layer's color, **potentially affected** by its transparency,
> **without any interaction or calculation** with the base layer's color. _(It then said how other blending modes_
> _were different and involved mathematical formulas.)_

This answer tells us the LLM doesn't know things such as:

- Computers work with numbers. A color is represented by a number in a computer.
- Therefore, a color cannot be blended by something other than a mathematical formula — nor can it be “affected” without a
  “calculation” by transparency, another number.
- “Transparency” is when you can see through something.
- “Seeing” works by sampling the color at various points, and processing that signal.
- Therefore, if you can see something through something, like, say, a base layer through an upper layer, then by definition,
  the color you will see is affected not only by the color of the upper layer and its degree of transparency, but also by the
  color of the base layer — or you wouldn’t be _seeing_ the base layer, which means that the upper layer is _not at all_ transparent, because you’re not seeing through it.

I mean, it sounds stupid to break it down like that, but I’m not wrong, am I? It really doesn’t know any of these things,
does it.

Can you prompt the LLM to explain [alpha blending](https://en.wikipedia.org/wiki/Alpha_compositing) properly?
Sure. But that just shows the LLM knows to put the words explaining it after the words asking the question. This capability does
not make the answer above into lesser evidence of the LLM not knowing _the things_ as opposed to _the words._

And of course people can be like that, too - eg [much better at the big O\
notation and complexity analysis in interviews than on the job](https://danluu.com/algorithms-interviews/). But I guarantee you that [if you put a gun to their head or\
offer them a million dollar bonus for getting it right](https://yosefk.com/blog/the-cardinal-programming-jokes.html#expanding-your-skill-set), they will do well enough on the job, too. And with 200 billion
thrown at LLM hardware last year, the thing can't complain that it wasn't incentivized to perform [3](#fn3).

Of course, these are simple examples. An LLM triumphalist will observe that they often stop reproducing; an LLM denialist
will assume they stopped reproducing through some conspiracy, like a chess engine tool having been given to the LLM, or it
having been drenched with synthetic data similar to your question. (I used to ask LLMs to prove 2+2=4; they'd very pompously
enumerate various notable properties of 2 and 4, and proudly declare that 2+2 must equal 4 based on these properties, and I had
a good laugh. Then LLMs were flogged to become “good at math,” and now they might say something about “Peano axioms,” and some
total garbage about set theory — but they emit enough S(S(2)) and such that it probably counts as a proof, though I am yet to
see the simple “2+2 = 2+(1+1) = (2+1)+1 = 3+1 = 4” which I’d expect from an entity understanding the question.)

For a more complex example, we can take associativity (which, as we’ve seen in 2+2=4, LLMs understand vaguely at best),
combine it with alpha blending and transparency (which apparently they don’t understand at all), and see how well LLMs do. I’ve
had an exchange with an LLM asking whether alpha blending, as implemented in commonly used libraries, is associative, or whether
it isn’t due to precision loss or whatever — and if it’s not associative, how does caching work in drawing programs (where the
program must be precomputing the blending of the layers above and below the currently edited one, to avoid recomputing the
blending of 10 or 100 layers upon every brush stroke.)

Sure enough, it said that alpha blending wasn’t associative — probably because I suggested that it might not be — and that
this is “solved with caching instead of mathematical elegance” — probably because I suggested that caching was involved. And
then I ask, but how can caching work if blending is not associative? If layer 6 is selected, and you blend the cached blending
of {1…5}, the selected layer 6, and the cached blending of {7…10}, you would get different results from blending {1…4}, 5, and
{6…10}, if blending is not in fact associative? And then if you selected layer 5 in the program, you would see a different
picture compared to selecting layer 6 - but in practice you see the same picture?

“You got me,” says the LLM, more or less. So their not knowing what any of the words actually mean very much does extend to
complex examples.

You could say that the LLM was a victim of its agreeableness [4](#fn4), since it might have been influenced by my contradictory implications that blending might
not be associative, yet caching must be implemented that counts on it being associative. I could say that, well, my whole
question was about which parts of my suspicions are incorrect, and saying they’re all correct is an abject failure — but let’s
assume it could be a character flaw more than an intellectual weakness. So in our last example, we’ll see the LLM having its own
opinion and sticking to it, despite being told repeatedly that it can’t be true.

I ask it about the thread safety of appending to a Python list from multiple threads, and whether I can tell the number of
times append was called with len(myList), and whether it will work once the [GIL](https://en.wikipedia.org/wiki/Global_interpreter_lock) is [removed](https://peps.python.org/pep-0703/).
It says that without the GIL, the program could corrupt memory. I say, no way, this is not C, it must be more like Java? And it
goes, no, _CPython is a C program_, and without the GIL your racy code can crash like C does. Java is different, _it_
_has a memory model_, and look at these crash reports from GIL-less Python. And I’m like, but these are bug reports, it’s not
_by design_, is there evidence that this is by design? — and it goes, it’s too early for the kind of evidence you’re
looking for to exist, no-GIL is too new, but here’s how a C program could crash in such scenarios… and on and on and on.

It does not know that [(pure) Python is a memory-safe\
language](https://yosefk.com/blog/a-100x-speedup-with-unsafe-python.html), and that no suggestion making it memory-unsafe would ever be accepted, and I found no way of persuading it to take
this notion into account — or to acknowledge that the evidence it’s citing in support of its claims is more like evidence to the
contrary (if all the crashes upon races you find are bug reports, it points to the requirement being that races don’t lead to
crashes.)

So it can be either kinda agreeable or very stubborn — and it might obviously not know what it just said in both modes.

**Can this be quantified?**

I don't see how.

I mean, I wish it could be. It's clear that LLMs do learn _some_ things about the world. For instance, even just the
token embeddings contain the representation of the concept of gender learned without any specific effort to teach the model what
gender is, as evidenced by “king - man + woman ~= queen” in the embedding space.

Ideally, you would want to quantify "how much of the world LLMs model." But even if you resolve the difficulty of defining
what this means, you'll run into the ease with which LLMs memorize answers to specific questions, so the vendor can celebrate
the new bar having been cleared.

All I can confidently claim is that they don't learn a world model except by accident, and there's neither a theoretical
reason nor empirical evidence for your being able to count on this accident in any defined and broad set of circumstances.

**So-called conclusions [5](#fn5)**

A guy who made $100 million from being an early employee of some startup came to give a lecture for that startup, and said “a
fundamentally incorrect approach to a problem can be taken very far in practice with sufficient engineering effort.” (He then
cheered up his listeners, most of whom had $100 million less than him, with the addendum “That's what I think is happening in
this company!”)

It is therefore not one of my conclusions that you can’t take LLMs very, very far just because they demonstrably do not learn
a model of the many worlds described by the words they’re trained on (which, BTW, is exactly as it says on the tin; nobody ever
called them LWMs.) I will, however, predict a few things — something you shouldn’t do if you don’t want to look stupid in the
future, but here goes.

**There will be at least one big breakthrough in machine learning around “world models”**. I have no idea what
this breakthrough will look like; I predict that it will happen because some important kinds of thinking cannot be done without
it, and I trust the Church-Turing thesis when it comes to these kinds of thinking, and I think someone will figure this out,
same as people have come up with deep learning, convnets and transformers. And of course you already have “world models”, such
as systems recovering object classes and positions from images — by a breakthrough, I mean a “generic” ability to build models
of “novel worlds” (even if the model isn’t as good as a specially tailored one), much like you throw any text into an LLM and
have it learn “something” without much tuning for this kind of text.

(In fact, I would guess there will be at least 2 more breakthroughs, the other one being around needing far less training
data — again, not because I know how machines could use less training data, but because I know you and I get by with less.
Feeding these algorithms gobs of data is another example of how an approach that must be fundamentally incorrect at least in
some sense, as evidenced by how data-hungry it is, can be taken very far by engineering efforts — as long as something is useful
enough to fund such efforts and isn’t outcompeted by a new idea, it can persist.)

**LLMs are not by themselves sufficient as a path to general machine intelligence; in some sense they are a distraction**
**because of how far you can take them despite the approach being fundamentally incorrect**. This should make “AI risk”
people happy; but “AI risk” is its own hilarity best left to another time.

**LLMs will never [6](#fn6) manage to deal**
**with large code bases “autonomously”**, because they would need to have a model of the program, and they don’t even learn
to track chess pieces having read everything there is to read about chess.

**LLMs will never reliably know what they don’t know, or stop making things up.** You need some sort of a world
model to have notions of knowledge, truth and falsehood. Any mechanism that is supposed to make LLMs “safe”, trustworthy or
other such is a mix of snake oil and honest efforts to somehow steer it away from text that spooks users — which can be done,
since users are spooked by form more than substance. For example, when it says some politically related nonsense, people drag it
for having the wrong politics, and you can “fix” it by making its output less politically charged — without making it less
nonsensical, which there’s no way to reliably achieve.

**LLMs will always be able to teach a student complex (standard) curriculum, answer an expert’s question with a useful**
**(known) insight, and yet fail at basic (novel) questions on the same subject, all at the same time**. This is not
surprising — this is exactly what you would expect from a language model that isn’t a world model. This fuzzy insight is more
cute than useful, however, since it’s hard to know what is and is not novel — in part because you come to the LLM in the first
place with things you don’t already know everything about.

(Some sources, such as [this\
wonderful writeup about LLMs not knowing that rotating a tic-tac-toe board doesn’t change the game](https://mindmatters.ai/2025/01/some-lessons-from-deepseek-compared-with-other-chatbots/), take this point to its
logical conclusion: “ _If you know the answer, you don’t need to ask an LLM; if you don’t know the answer, you can’t trust an_
_LLM._” But this conveys a true insight into LLMs together with unwarranted pessimism about their utility. In fact, sometimes
you know the answer, but it’s quicker to proofread an LLM’s output than to type it out; sometimes you don’t know the answer, but
know to check if the LLM’s answer is correct, etc. etc.)

**LLM-style language processing is definitely a part of how human intelligence works — and how human stupidity works.** I agree with Dijkstra that “can machines think?” and “can submarines swim?” are poor questions to ask, and I hate it
when people say that neural networks “work like the brain” and such. But I can’t help feeling that LLMs are a mirror into a part
of how people such as myself think — and I don’t like what I’m seeing in that mirror. “Thinking” by guessing what words to say
next based on words we’ve previously heard might actually help find a good idea — and it’s also how know-nothings get through
work meetings, and how people come to think they know stuff they really don’t, and how they internalize the stupidest notions. I
am starting to think that in today’s environment, high cognitive skills are an actual risk factor for stupidity, and that
learning words without learning a model of what they refer to is one big part of the problem.

**P.S.** I wish I could say something about how to best use LLMs for programming — something I would like to be
qualified to speak about, and that I am kinda supposed to learn enough to be qualified to speak about. I don’t think I am; I can
only say that I tried Cursor and it failed every time, including at replacing f(obj) and g(obj) with obj.f() and obj.g() (it
would occasionally mix up f and g, and I got tired of reviewing its output), and I went back to simply copying code into and out
of chat windows. I would say that I use LLMs like I use SIMD — sometimes it’s a good fit for leaf functions whose behavior is
relatively easy to specify and test, and it has no business being anywhere else.

I have conflicting theories about why some people do great things with “agentic AI” while I think it’s hopelessly useless for
me; I am waiting for someone to write something crisp and well-researched about this to teach me the truth, or a useful
approximation. I console myself with the idea that I can’t be missing out on too much, given how terrible the output I’m getting
from LLMs often is.

**P.P.S.** Here’s a somewhat rough Soviet-era joke about a school for kids with special needs that illustrates
my point. Rachel says that a Russian joke isn’t a joke, but a story of pain. To this I reply that some people like them.

> An inspector comes to a school for kids with developmental issues. He asks a kid riding a wooden horsie his name, and the kid
> says “MMMM.” He says, what do you want to be when you grow up? — and the kid says “MMMM.” The inspector turns to the principal
> and says, “you’re doing nothing for these kids. I’ll be back in a month — if there’s no improvement, we’ll close the
> school.”
>
> He comes back a month later and finds the kid swinging on the wooden horsie, same as last time; if you want to tell this
> masterpiece of Soviet humor at parties — the perfect conversation starter — you should be swinging wildly when saying the kid’s
> lines:
>
> — What’s your name?
>
> — MMMMikey!!
>
> — Mikey? Nice to meet you, Mikey. What do you want to be when you grow up,
> Mikey?
>
> — MMMMastrounaut!!
>
> — An astronaut? Good stuff, good stuff! And how old are you?
>
> — MMMMikey!!

The moral of the story being that you can learn to predict the next word without learning much about the world — at least up
to a point.

_Thanks to Dan Luu for reviewing a draft of this post._

* * *

1. It helps that I don’t know the good opening moves; I can’t be bothered to learn any opening theory. The fact
   that my poorer chess knowledge makes it easier for me to see how bad the LLM is at chess is an interesting case study. It turns
   out that you can get good answers out of LLMs by asking very well-phrased questions sounding like someone else’s well-phrased
   questions answered in its training data; whereas if you ask simpler questions which are perfectly valid but not commonly asked,
   they will fall apart. [↩︎](#fnref1)

2. Funnily or tragically, my system of tripping up the opponent with weak moves it hasn’t memorized a response to
   is conceptually similar to grandmaster play of today, where grandmasters memorize chess engine lines, and a “novelty” is a
   relatively weak move your opponent has never analyzed with the engine, whereas you did and you remember all the strong moves
   after this weak one. Of course my strategy would not work against a grandmaster, because I don’t come prepared with memorized
   engine lines, and the grandmaster would find much better moves than I would over the board. Still, this 21st century concept of
   “chess novelty” is tangentially related, and funny, or tragic, as the case might be. [↩︎](#fnref2)

3. People can also give the wrong answer because they’re drunk. I don’t think the LLM was drunk. My point is that a
   person who gave this answer would get zero points for this question on a test, and that the LLM is constantly under test because
   it’s a machine serving no purpose other than answering these questions, and I don’t see why it should not get zero points here,
   even though people might eg fail to answer some logic puzzle phrased in one way but succeed when it is phrased in another way,
   etc. etc. — I don’t see how the cognitive weaknesses of people provide an excuse for the machine in this specific case. [↩︎](#fnref3)

4. Actually, in this case, it was agreeable in substance but snarky in tone — it gave me an answer that confirmed
   all my different suspicions, contradictory as they were, and at the same time it was saying something like “don’t expect the
   world to be pretty or simple, man, the world is messy, man.” Generally I don’t think that LLMs “personality,” “style,”
   “politics” and other anthropomorphic characteristics are the main thing about them; I think the main thing is what they model
   (text) and what they don’t model except by accident (the thing the text is about.) [↩︎](#fnref4)

5. It’s hard to call them “conclusions” when they’re fuzzy statements supposedly following from my fuzzy claim. In
   fact this is what bugs me about LLMs in general: the thing is fuzzy — you can’t say it does something, because sometimes it
   fails to do it; you can’t say it doesn’t do something, because sometimes it succeeds; and you can’t discuss the rates of
   real-life success and failure, because who’s keeping score? This is why it’s hard for me to write about LLMs — I don’t like it
   when things get this fuzzy, certainly when it comes to long-form writing; I’m reduced by the very nature of the subject to
   shitposting about this on Twitter, along the lines of “ [Computers\
   used to provide cheap, reliable automation; then AI came along.](https://x.com/YossiKreinin/status/1946153421817397567)” [↩︎](#fnref5)

6. When I’m saying that an LLM will never be able to do something, I mean it in the sense of “y = ax + b will never
   represent a parabola” rather than in the sense of “the points residing on a curve rather than a straight line can never be
   represented by an equation.” _Machine learning_ might do what _LLMs_ can’t do. Of course this could be used for a
   No True Scotsman defense — “if it clearly learns a model of the world, it’s not a true LLM.” I’m assuming that when a big
   breakthrough is achieved, we’ll know enough about it to be able to settle the question whether it’s still an LLM, as long as
   we’re arguing in good faith — same as we don’t know all the details of how commercial LLMs work, but we know about transformers,
   tokenization, encoders, decoders, next token prediction, whole-text synthesis, etc., and this is enough for “LLM” to have a
   somewhat technical meaning — not as precise as “y=ax+b,” but not nearly as vague as, say, “AI.” [↩︎](#fnref6)
