---
title:
  Initial experiences and notes on Talon, Cursorless and Rango
url: https://graydon2.dreamwidth.org/312887.html
published: "2024-05-28T01:44:30Z"
feed: graydon
guid: tag:dreamwidth.org,2011-01-04:684470:312887
---

# Initial experiences and notes on Talon, Cursorless and Rango

My wrist has been hurting a little bit recently, and a friend of
mine has been telling me about how he's using
[Talon](https://talonvoice.com/) (and its extensions
[Cursorless](https://www.cursorless.org/) and
[Rango](https://rango.click/)) as an interface to programming
entirely with his voice, so I thought I'd give it a try. I'm now
about three days into using it, and I have it working well enough
that I just composed my first real patch that makes a not
completely trivial change to a program, and while it took me
almost an entire workday to complete a 165-line patch, I can
definitely feel myself accelerating and gaining fluency in the way
the system works.

(This blogpost is also being mostly composed using Talon)

Some notes about limitations and experiences so far:

\- Talon is extremely oriented towards programmers.

\- Cursorless even more so, I don't think it works on anything
except VSCode.

\- Talon, Cursorless and Rango are all seemingly being developed
by different people, and to some extent they fight over their
definitions of words and commands. This is most evident in things
like Cursorless anchoring its cut and paste commands in weird
synonyms like "carve", "chuck" and "bring".

\- By far the most annoying instance of this is the fact that you
have to refer to "line"s as "row"s in some contexts but not
others, and if you get it wrong the command recognition system
doesn't work properly and you just wind up in putting a garbage
command and everything gets mangled by a mis-parsed bulk command.

\- It's also evident in several phrase level inconsistencies, such
as subject verb object ordering varying between the packages and
even sometimes within packages, or the use of counting words with
or without ordinal suffixes (sometimes you have to say three,
other times you have to say third, in extremely unnatural contexts
like "tab next third").

\- There are also just a bunch of cutesy words that seem to be
chosen for no reason other than they are cute: surely it is not
any easier for the voice recognizer to hear the word "hunt" than
it would be to hear the word "find" but we must "hunt" and "scout"
for things.

\- You also have to learn a custom phonetic alphabet, which I kind
of understand the motivation behind, but it adds substantially to
the cognitive load because you simply can't use the letters you
already know most of the time. Also one of the words (sit) starts
with a letter different from its letter -- i -- and I find this
vexing. Surely there's a short i-word. Ick? Ice? Inch?

\- Generally speaking the voice recognizer is very good,
especially good for streams of continuous text. The fact that this
sort of thing works on consumer hardware is quite remarkable to
me, having grown up in the eighties and nineties where we had demo
after demo of this sort of thing theoretically working but then in
reality it didn't. it really seems like computers can at least
understand speech now.

\- It does struggle, however, with changes to ambient noise, and
with gain levels. I have had to adjust these levels a lot to get
correct recognition, and am mostly programming with my face a
couple inches from a high quality hundred dollar microphone. This
seems a little fussier than I'd expect! Also I live near a
seaplane aerodrome and so every 15 minutes or so there's an engine
running up and the voice recognizer is defeated for a minute or
two. This gets stale.

\- Technically the interface is modal, but in practice you spend
almost all your time in command mode, and the inconsistencies
between the two modes are almost more annoying than if there was
only one mode.

\- I ran into a bunch of direct snags right off the bat: obviously
audio configuration on Linux remains a complete disaster in the
year of our lord twenty twenty four, also it relies on desktop
panel indicators that only exist in certain desktop environments
on Linux, and all that sort of nonsense. More seriously it does
not engage readline mode on gnome terminal, or the VSCode
terminal, and the terminal inhibits the use of the keystroke that
is used to trigger commands in VSCode unless you hack around it
(there's an open bug on this at least). It also incorrectly
assumes that you've customized no keys at all inside a VSCode,
sending keystrokes rather than raw commands for a wide variety of
editing commands, so those of us who are emacs refugees with emacs
keymaps need to undo that before we can do anything, and then
can't type anything if we ever need to take over briefly using our
hands because we're too frustrated with the edits happening from
our voice. Also the grid mode seems like it will work well but
then stops working at random and just shows incorrect magnified
regions of the screen, probably this is some resolution related
thing.

\- The Cursorless unambiguous-colored-dots plus
structured-target-grammar navigation system is extremely clever
but it does take a lot of concentration. Between that and the risk
of issuing a wrong command and having everything go to hell, I
found myself with very little cognitive space available to
actually think about what I was trying to do. Programming this way
is much more mentally taxing, and uses up time in the day much
more quickly, requiring much more focus on what I'm doing. I also
have to stop and think in great detail before I make an edit
because I can't just let my fingers walk me through the code and
figure out what I'm doing by looking and jumping around and
scrolling. I don't know how else to describe it but it's very
different from normal programming, it feels like really using a
different body part, which I guess it is.

\- One funny aspect is that you have to be able to basically guess
the number of things fairly often. Or at least eyeball it and get
the number nearly right. Like you often have to say "go word right
sixth" and if you're wrong and it was actually seven or eight or
five words, you have to correct it. Since you don't actually want
to count the words out you just kind of have to go by your gut and
hope that your counting happens subconsciously correctly.

\- There's a very clever system of automatic formatting prefixes
and modifiers that allow you to automatically apply capitalization
patterns or dash patterns or underscore patterns or that sort of
thing to words while you're entering them. This part seems to work
quite well.

\- Theoretically there's some sort of mechanism for abbreviations
and homophones as well which occur all the time in programming,
but I haven't figured out how to use that part yet. Maybe that's
for tomorrow.

\- Customization seems to involve learning the entire mental model
of the system as a python program and then figuring out where
you're supposed to insert overrides in an extremely dynamic,
override rich environment. On the one hand there appear to be many
very convenient places to insert such overrides, on the other hand
debugging them appears to be about as easy as debugging emacs
extensions if they were written in a mixture of python,
javascript, and two bespoke programming languages with nearly no
documentation (Talon and VSCode commands).

\- Generally speaking there is the kernel of greatness here.
Particularly if the three different developers working on the
three interacting systems were to collaborate, eliminate
inconsistencies and ensure interoperability, and integrate with
platform accessibility features such that the best of all three
systems would be available in all contexts: the Cursorless edit
functions should be available inside the browser, the Rango
widget-navigation anchors should be available inside VSCode, and
both should be available inside a wide variety of other
applications on the desktop because there are more programs than
just those two.

\- I'm going to keep trying to use the thing for another few days
to see if it continues to get better, or if annoyances become
overwhelming and I just can't stand it. Obviously this depends on
me having the ability to continue using my hands when I feel like
it, which is definitely still the case, I'm really just having the
slightest twinges of soreness and it probably just has to do with
a little bit too much garden work. But it would be nice to be
proficient in two totally disjoint input methods, to have a
backup, and it's also just kind of cool to do editing in the
larger semantic units and direct navigation that this system
offers or at least suggests the possibility of.

![comment count unavailable](https://www.dreamwidth.org/tools/commentcount?user=graydon2&ditemid=312887)
comments
