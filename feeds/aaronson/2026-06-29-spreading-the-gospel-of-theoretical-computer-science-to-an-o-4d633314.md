---
title: 'Spreading the Gospel of Theoretical Computer Science to an Omega(1) Fraction of Humanity: My Trevisan Award Acceptance Speech at STOC'
url: https://scottaaronson.blog/?p=9881
published: "2026-06-29T01:32:13Z"
feed: aaronson
guid: https://scottaaronson.blog/?p=9881
---

# Spreading the Gospel of Theoretical Computer Science to an Omega(1) Fraction of Humanity: My Trevisan Award Acceptance Speech at STOC

[![](https://scottaaronson.blog/wp-content/uploads/2026/06/trevisanaward-1-1024x768.jpg)](https://scottaaronson.blog/wp-content/uploads/2026/06/trevisanaward-1-scaled.jpg) Take that, *Shtetl-Optimized* haters of the world!

[![](https://scottaaronson.blog/wp-content/uploads/2026/06/award2-1024x768.jpg)](https://scottaaronson.blog/wp-content/uploads/2026/06/award2.jpg) With longtime friend and colleague [Salil Vadhan](https://salil.seas.harvard.edu/), as well as Luca Trevisan’s widower Junce Zhang, at the STOC banquet on Tuesday, before I was given half an hour to try to make people laugh

**Spreading the Gospel of Theoretical Computer Science to an Ω(1) Fraction of Humanity (Or, How We Can Do Like the Physicists)**

**Scott Aaronson’s Trevisan Award Acceptance Speech**

**Salt Lake City, Utah, June 23, 2026**

Thank you so much! It’s one of the highlights of my life, frankly, to accept the first-ever [Luca Trevisan Award for Expository Work in Theoretical Computer Science](https://scottaaronson.blog/?p=9732)—because of, firstly, what this entire STOC community means to me, but also what [Luca Trevisan](https://en.wikipedia.org/wiki/Luca_Trevisan) in particular meant to me. Luca was one of the main people who taught me complexity theory—first at an IAS summer school in 2000, then at UC Berkeley, where I took two of his courses and TA’ed for him. As a member of my dissertation committee, Luca once stood on a street corner in San Francisco to meet my friend to sign the signature page of my [thesis](https://www.scottaaronson.com/thesis.html), as I struggled to get the thing in by the deadline. Later, Luca’s theoretical computer science blog, [In Theory](https://lucatrevisan.wordpress.com/), bounced off of my blog.

I wish Luca were here now. But knowing him as well as I did for a quarter century, I feel like I know what he’d say if he learned that I had received the inaugural prize that bears his name. I imagine he’d slap his forehead and say “Seriously, there was no other option??” But I’d like to think that he’d eventually reconcile himself to the choice!

By the way, I noticed that in the committee’s prize announcement, which I found so moving, they added a special paragraph at the end that basically said, “ *please* don’t imagine that to win this prize in the future, you need to behave the way Aaronson behaves. You can just write beautiful textbooks or survey articles or whatnot, and be normal and sane.”

---

The foundation of my career is that I realized 25 years ago that there were better theoretical computer scientists than me—like many in this room, or like [Ryan Williams](https://en.wikipedia.org/wiki/Ryan_Williams_(computer_scientist)) or [Andris Ambainis](https://en.wikipedia.org/wiki/Andris_Ambainis), both of whom I knew at the time. *Certainly* there were better quantum physicists than me. There were better writers, better expositors, better performers. On the other hand, if you looked specifically at the intersection of computational complexity *and* quantum physics *and* standup comedy, that was just this totally uncontested territory!

I’ll let you in on a secret: pretty much everything I’ve done for decades has just been drawing out one joke. That joke is, basically, “computer scientists they be like *this*, but physicists they be like *that*.” The physicists they be like *\[exaggerated doofus voice\]* “duhhhh, [NP](https://en.wikipedia.org/wiki/NP_(complexity)), what’s that stand for? Not Polynomial?” See, but then there’s also a Rodney Dangerfield aspect to it, because it’s like, how come we never get as much respect as the physicists get? (Though when we *do* get that respect, I confess that I complain all the more, because then I lose my shtick…)

It’s true that the physicists have certain built-in advantages. They had Einstein, Stephen Hawking, the atom bomb—and just the fact that they’re ultimately talking about, or trying to talk about, the world that we can see and touch. A black hole is an actual place that you could visit, even though I wouldn’t recommend it. But physicists also have much better names for things than we do. I mean, black hole? Big Bang? Quark? Gluon? Supersymmetry? Dark matter?

Meanwhile, what names have we got? [TFNP](https://en.wikipedia.org/wiki/TFNP). [NC1](https://en.wikipedia.org/wiki/NC_(complexity)). And worst of all, [PP](https://en.wikipedia.org/wiki/PP_(complexity)). These are names that you want to flush down the toilet. But also, the *concept* of a [zero-knowledge protocol](https://en.wikipedia.org/wiki/Zero-knowledge_proof), or a [two-source extractor](https://annals.math.princeton.edu/2019/189-3/p01), just inherently take longer to explain to people than the concept of a particle, or even a field—even though the latter *also* turn out to be extremely abstract and mathematical when you push on them. Ask a physicist what a particle is, they’ll tell you that it’s an [irreducible representation of the Poincaré group](https://physics.stackexchange.com/questions/277986/why-are-particles-thought-of-as-irreducible-representations-in-plain-english). See, but people *think* they know what a particle is, it’s just a tiny little hard sphere that moves around, and that’s good enough for them.

---

So then, how can we win the grand popularity contest against the physicists? How can we, as I put it in my title, spread the gospel of CS theory to a constant fraction of the human race? In my view, the first step is to reframe who we are and what we’re about. We’re not this obscure little community off to the side, proving its little theorems about derandomization and catalytic space. No! What we are is the conceptual and mathematical core of computer science, the field that’s changing the face of civilization in obvious and undeniable ways.

This was even true a long time ago. The physicists had Galileo and Einstein? Well, we had Turing, a figure so heroic and so tragic that no one would’ve believed him as a fictional character. And while we’re at it, we’ll claim Gödel and Shannon and von Neumann, Leibniz and Babbage and Ada Lovelace—they’re all ours too.

That’s our proud history. But then when we turn to today, it’s like, holy crap! Even the densest ignoramus can now see how deep intellectual ideas originating in CS are changing the world.

---

Blockchains—some people might wish they’d never been invented, but they *were* invented, so we all need to think about how they change the world’s economy for better and worse. And of course, they’re fundamentally based on hardness assumptions; they couldn’t exist in a world where NP was easy.

Part of my outreach job these days is to explain to finance people, over and over, why a quantum computer could break the elliptic curve signature schemes used by Bitcoin and many other coins, but would have only a more modest effect on the proof-of-work part, the hash function. And it’s like, if you actually want to know, then we need to talk about BQP versus NP, Grover’s algorithm and its optimality, black-box problems with and without abelian group structure—and now we’re deep into TCS!

Speaking of quantum computing—even if we set aside the question of whether quantum computing is going to revolutionize materials science or chemistry or pharmaceuticals design—or whether it will revolutionize AI and machine learning and optimization *\[I shake my head, make a thumb-down, and blow a raspberry\]*—even if we set aside those practical questions, quantum computing plausibly represents the most dramatic test of quantum mechanics itself that we’re ever going to see. And it now looks clear that we *will* see that test within the next decade or sooner. One way or the other, we’re going to learn the truth.

People sometimes ask me, why did it take until the 1980s for anyone to propose the idea of quantum computing? You know, Heisenberg and Schrödinger were in the 1920s, Turing was in the 1930s, so it seems like all the ingredients were in place a half-century earlier! In my *[Quantum Computing Since Democritus](https://www.amazon.com/Quantum-Computing-since-Democritus-Aaronson/dp/0521199565)* book, I reflected on this, and I think the deepest answer is that *not* quite all of the intellectual ingredients were in place. Quantum computing is something that it doesn’t make a great deal of sense even to ask about until you’ve established polynomial versus exponential, and even P and NP and NP-hard, as central concepts. And that’s what didn’t happen until the 1970s.

---

But of course, the *biggest* thing that our CS concepts have unleashed on humanity—the thing that the entire world now realizes holds even greater promise and greater peril than nuclear energy did in the last century—is *\[pause for effect\]* the [Razborov-Smolensky lower bound method](https://ocw.mit.edu/courses/18-405j-advanced-complexity-theory-spring-2016/7da23045f5aa17dc72c329c23b3b6c94_MIT18_405JS16_Razborov.pdf).

No, I’m kidding of course. It’s generative AI.

Twenty years ago, I remember people in our community—was it Fortnow? Impagliazzo? I’m not sure—saying, “you know the *real* reason why P vs. NP is such an important problem? Suppose P=NP, via an algorithm that was fast in practice. Then it’s not just that you could break all the encryption systems, or have your computer find a proof of the Riemann Hypothesis, or whatever. No, it’s that you could program your computer to find the shortest efficient compression of, for example, the full text of Wikipedia. For in order to create that compression, it seems plausible that your computer would need to create an AGI as a byproduct.”

I remember thinking to myself: “that’s an amusing thought experiment, I’ll need to steal it sometime, but still, what an utterly simplistic vision of the nature of intelligence! There *has* to be more to intelligence than sheer data compression!”

Fast forward to spring 2022, when I accepted an invitation to go on leave for a couple of years, to join what was then a relatively obscure little nonprofit foundation by the name of … err … OpenAI. When I flew to San Francisco to start my assignment, I had lunch with [Ilya Sutskever](https://en.wikipedia.org/wiki/Ilya_Sutskever), the cofounder of OpenAI and then its chief scientist. And Ilya said to me, “Scott, let me explain to you how we think about things here at OpenAI. For us, intelligence is fundamentally about prediction, and prediction is fundamentally about compressing your training data. As you know, [Kolmogorov complexity](https://en.wikipedia.org/wiki/Kolmogorov_complexity) is uncomputable, but one can get better and better computable upper bounds on it. We conjectured that, in order to get sufficiently good at predicting and compressing all the text on the Internet, you’d need to build a model of the entire world that had led to that text being written. And we made a gamble that large neural nets would do that well enough, despite the problem’s worst-case intractability.”

That conversation was when it hit me that, if only we in CS theory had taken our own concepts and thought experiments more seriously, one of *us* could’ve started OpenAI 15 or 20 years ago. So OK, we didn’t, and that’s why I flew coach to get here. But this is the kind of story that it seems to me we could be singing from the rooftops.

(Incidentally, the reason why OpenAI wanted me back in 2022, was to use theoretical computer science to figure out how to make AI safe for humanity. Alas, that problem is still open! But I’m thrilled that there are so many sessions about exactly this question at STOC this year, and I hope many of you will choose to get involved.)

---

In the rest of this talk, I’d like to offer some advice—such as I have—for any of you who’d like to try *your* hand at speaking or writing or blogging or podcasting about theoretical computer science for a broad audience. You see how my hair is starting to gray? Yeah, that’s what authorizes me to go into advice mode.

Let’s start with the obvious: meeting the audience where they are. This is something that I learned years ago from [Steven Rudich](https://scottaaronson.blog/?p=8449), who along with Luca, was another irreplaceable figure who our community recently lost, and lost too soon. I remember 26 years ago, at that same IAS summer school where I learned from Luca, Rudich gave the students a talk about how to give talks. In it, he showed a cartoon of someone lecturing. And there were little thought bubbles that said:

**What the speaker thinks the audience is thinking:** MORE! HARDER! FASTER! Ah yes, QED, truth is beauty and beauty is truth!

**What the audience is *actually* thinking:** What the hell are they talking about? When is this over? Can I get a date with the person sitting next to me?

You know, this misconception that because something has become obvious to you, after thinking about it for years, *therefore* it should be equally obvious to your readers or listeners encountering it for the first time? This is what [Steven Pinker dubbed “the Curse of Knowledge,”](https://www.amazon.com/Sense-Style-Thinking-Persons-Writing/dp/0143127799) and calls the most fundamental problem of all exposition. (I could mention the related misconception that because something has become *interesting* to you, therefore it’s *interesting* to your audience. But you can make just about anything that’s interesting to you interesting to your audience, by telling a suitable story about it.)

What can you do about the Curse of Knowledge? Practice giving a buttload of talks to undergrads, high school clubs, even physicists, and *listen* to the feedback you get. If the same weird confusion shows up at least twice, it’s a safe bet that it’s going to keep showing up—which means, now you can anticipate and preempt it the next time you explain the same concept.

But it’s not just misconceptions that you should listen for. Listen for which of your metaphors and anecdotes actually land. *Certainly* listen for which of your jokes get a laugh. Use those more the next time. And if saying, for example, “hur hur, I’m in a *quantum superposition* of two different topics that I could talk about next”—if that *fails* to get a laugh, then DROP IT.

Eventually, you’ll build up what Carl Sagan [once called](https://www.amazon.com/Demon-Haunted-World-Science-Candle-Dark/dp/0345409469) “consumer-tested stepping stones”: that is, a library of jokes, anecdotes, and metaphors that can get you from wherever you see the audience is to wherever you need them to be. Here’s an intentionally tiny example of one of my stepping stones: “Why is P contained in NP? Because the verifier just says to the prover, dude, take a hike, you’re not needed here.” Or another stepping stone, for when we reach the question of the likelihood of P=NP: “look, if we were physicists, we would’ve declared P≠NP to be a law of nature. We would’ve given ourselves Nobel Prizes for the discovery of that law. And if it later turned out that P=NP? We’d just give ourselves more Nobel Prizes for the law’s overthrow!”

---

This brings me to a broader point. CS theory is unusually rich with facts that are true for silly or absurd or ironic reasons. Lean into that! Don’t hide it!

I mean, “if NP has small circuits then this theorem is true, but if NP doesn’t have small circuits, the theorem is *again* true, but now for a totally different reason”? That’s sidesplittingly hilarious! OK, maybe only to some of us.

Or why does IP=PSPACE? An alien lands and is like, “I COME TO EARTH TO TELL YOU THAT WHITE HAS THE WIN IN YOUR GAME CALLED CHESS.” And we’re like, “why should we believe that?” So the alien is like, “LET US PLAY A GAME. I’LL PLAY WHITE AND WILL WIN.” And we’re like, “oh, we assume you’re smarter than us! You came all the way here in a spaceship and all! But that still doesn’t prove it.” So the alien is like, “THEN LET US PLAY A DIFFERENT GAME, MATHEMATICALLY EQUIVALENT TO CHESS, INVOLVING SUMS OF POLYNOMIALS OVER A FINITE FIELD. IN THIS TRANSFORMED GAME, THE BEST YOU CAN DO IS TO MOVE RANDOMLY. SO IF I STILL WIN, YOU’RE STATISTICALLY CERTAIN I WOULD’VE WON REGARDLESS OF HOW YOU PLAYED.” It’s like, dude. Dude!

Of course, our founding irony, our founding absurdity, was self-reference and diagonalization. Like, “you can’t predict what any human brain will do 5 seconds from now, because if you could, you could predict what *you yourself* were going to do 5 seconds from now, and then do the opposite of that!” BOOM! Therefore the halting problem is undecidable and the [Time Hierarchy Theorem](https://en.wikipedia.org/wiki/Time_hierarchy_theorem) is true, QED. But beyond that: “ [black-box program obfuscation is impossible](https://eprint.iacr.org/2001/069), because one thing you can always do, if given the actual code of a program, is to *run the program on its own code* and see what happens.” Dude! Or: the reason why it’s so hard to prove P≠NP, is that it’s presumably *true* that P≠NP. That’s a wisecrack that, in the context of the [Natural Proofs barrier](https://mit6875.github.io/PAPERS/natural_proofs.pdf), becomes so much more than a wisecrack.

One special case of leaning into absurdity concerns the central role in our field played by asymptotics. I’m always slightly at a loss when someone asks me, “so, how many times faster would a quantum computer be than a classical computer? A million times faster? A billion?”

Part of me wants to reply: “I *must* educate you about polynomial versus exponential scaling until you see the profound error of your question, and retract it.” But another part of me simply wants to say: “depending on the problem, a quantum computer could be anywhere from not faster at all to, let’s say, *1010000 times* faster.”

The truth is, I think we need to do both. Anytime you’re talking about asymptotics to laypeople, if you can plug in some representative numbers, it will help them understand what you’re talking about. And then, if the asymptotics are what really control the real-world numbers, so much the better! If, on the other hand, the asymptotics are comically disconnected from the real-world numbers—if, for example, you’re trying to improve something from [log\*(n)](https://en.wikipedia.org/wiki/Iterated_logarithm) to [Ackermann-1(n)](https://www.gabrielnivasch.org/fun/inverse-ackermann) or whatever—well then, you can lean into that as an additional source of humor.

---

Alright, one last piece of advice. Tell true stories about how *you* came to understand or discover whatever it is that you’re talking about. Don’t be like the mathematicians who love to cover their tracks.

When people ask me how I proved the [lower bound](https://www.scottaaronson.com/papers/collision.pdf) on the number of steps needed for a quantum computer to find collisions in a list—a centerpiece of my PhD thesis, and one of the two or three hardest technical things I’ve done in my career—I say, look, I was 20 years old and I had no social life. So I just pulled many all-nighters trying every possible approach. Eventually, I came across some complicated expression that had no right to be a polynomial. But somehow, every term in the denominator cancelled against a corresponding term in the numerator, and it *was* a polynomial! And that let me use the [polynomial method](https://arxiv.org/abs/quant-ph/9802049) to prove a lower bound. Why was it a polynomial? I still don’t really understand, a quarter-century later! My point is, people want the truth.

The secret of blogging is that, even if people despise what you’re saying, even if they think it’s wrong, offensive, problematic, cringe, you name it, they need to trust that you’re telling them the truth of what *you* know or believe or remember about the subject at hand, the same as you’d tell your closest friend.

---

In summary: we, the CS theory community, are sitting on top of one of the greatest conceptual and intellectual goldmines of our whole civilization. I exhort everyone here: please help tell the world about it! As you do so, think about how to honor Luca’s memory and make him proud. But also think about how to make me, and my silly little blog, superfluous and obsolete.

Thank you for this honor, thank you for the incredible privilege of being part of the CS theory community, and thank you for listening.
