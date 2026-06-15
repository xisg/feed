---
title: Unix Quiz answers
url: https://commandcenter.blogspot.com/2020/01/unix-quiz-answers.html
published: "2020-01-22T09:30:00Z"
feed: robpike
guid: tag:blogger.com,1999:blog-6983287.post-4838497052045046556
---

# Unix Quiz answers

(Here's the next resurrected post, this time from Jun 8, April 30, 2012.)

A while back I posted the questions from the 1984 Unix/mpx Exit quiz: [https://commandcenter.blogspot.com/2020/01/unix-quiz.html](https://commandcenter.blogspot.com/2020/01/unix-quiz.html)

Here they are again, with annotated answers. The answers are in the form of the simplified pattern that the mpx exit program used to verify them.

Note: I have never seen a correct set of answers posted on line except when they originated from this list. In other words, I don't believe anyone ever got all the questions correct, even with a web search engine. It may not even be possible.

1\. Q:The source code motel: your source code checks in, but it never checks out. What is it?

A:sccs

The Source Code Control System. The reference is to the Roach Motel.

2\. Q:  Who wrote the first Unix screen editor?

A: irons

Ned Irons wrote a full-screen editor (the first?) at IDA in the 1960s and a Unix version, perhaps a translation, at Yale a few years later.

3\. Q:  Using TSO is like kicking a {what?} down the beach.

A: dead whale

The quote is from Steve Johnson. For those of you who never experienced TSO, it's an accurate characterization.

4\. Q:  What is the filename created by the original dsw(1)?

A: core

Setting the file's i-number using the switches on the front panel was the other half of its user interface.

5\. Q:  Which edition of Unix first had pipes?

A: third \| 3

Look it up.

6\. Q:  What is =O=?

A: empire

Empire was a large-scale strategy game developed at Reed College and made a computer game by Peter Langston (and perhaps others?).

7\. Q:  Which Stephen R. Bourne wrote the shell?

A: software \| 1138 \| regis

There were two people named Stephen R. Bourne working at SGI in the early 1980s. The one who wrote software (as opposed to designing hardware), who had worked in BTL Center 1138, or whose middle name was Regis: he wrote the shell.

8\. Q:  Adam Buchsbaum's original login was sjb. Who is sjb?

A: sol & buchsbaum

Adam was one of many Unix room kids with parents who worked at the Bell Labs. Adam was unusual because his father was executive vice president but apparently didn't have enough clout to get his kid his own login.

9\. Q:  What was the original processor in the Teletype DMD-5620?

A: mac & 32

The 5620 was the productized Blit. Trick question: most people thought the 5620, like the Blit, had a 68000 but it used the much less suitable yet more expensive BELLMAC-32.

10\. Q:  What was the telephone extension of the author of mpx(2)?

A: 7775

The (2) here is important. Greg Chesson wrote mpx(2); I wrote mpx(8).

11\. Q:  Which machine resulted in the naming of the "NUXI problem"?

A: series 1 \| series one

The IBM Series/1 was big-endian. Swap the bytes of "UNIX", the first text printed as Unix booted.

12\. Q:  What customs threat is dangerous only when dropped from an airplane?

A: belle \| chess machine

Ken Thompson took Belle (the chess machine) to the Soviet Union for a tour. Well, he tried: the machine was impounded by U.S. customs and never left American soil. Ken's trip was not much fun.

13\. Q:  Who wrote the Bourne Shell?

A: bourne

The only real gimme in the list. Feels like a trick question though.

14\. Q:  What operator in the Mashey shell was replaced by "here documents"?

A: pump

Sometimes better ideas do win out.

15\. Q:  What names appear on the title page of the 3.0 manual?

A: dolotta & petrucelli & olsson

This refers to System 3, two before System V. It always makes your technology seem more modern when you use Roman numerals.

16\. Q:  Sort the following into chronological order: a) PWB 1.2, b) V7, c) Whirlwind, e) System V, f) 4.2BSD, g) MERT.

A: cagbef

Whirlwind is a ringer.

17\. Q:  The CRAY-2 will be so fast it {what?} in 6 seconds.

A: infinite \| np-complete \| p=np

What can I say? The whole exercise was juvenile.

18\. Q:  How many lights are there on the front panel of the original 11/70?

A: 52

We counted.

19\. Q:  What does FUBAR mean?

A: failed unibus address register

It's really a register name on the VAX-11/780. Someone at DEC liked us.

20\. Q:  What does "joff" stand for?

A: jerq obscure feature finder

Tom Cargill's debugger changed its name soon after we were pressured by management to change Jerq to Blit.

21\. Q:  What is "Blit" an acronym of?

A: nothing

Whatever people thought, even those who insisted on spelling it BLIT, Blit was not an acronym.

22\. Q:  Who was rabbit!bimmler?

A: rob

See Chapter 1 of The Unix Programming Environment for another theory.

23\. Q:  Into how many pieces did Ken Thompson's deer disintegrate?

A: three \| 3

During takeoff at Morristown airport, propeller meets deer; venison results.

24\. Q:  What name is most common at USENIX conferences?

A: joy \| pike

Sun Microsystems made these obnoxious conference badges that said, "The Joy of UNIX". We retaliated.

25\. Q:  What is the US patent number for the setuid bit?

A: 4135240

Dennis Ritchie was granted the only patent that came out of Research Unix.

26\. Q:  What is the patent number that appears in Unix documentation?

A: 2089603

I'm not a fan of this stuff. Look it up yourself.

27\. Q:  Who satisfied the patent office of the viability of the setuid bit patent?

A: faulkner

At a time when software patents were all but unknown, Roger Faulkner demonstrated to a judge that the patent was sufficiently well explained by taking a Unix kernel without it and putting it in, given the patent text. A clean-room recreation, if you will.

28\. Q:  How many Unix systems existed when the Second Edition manual was printed?

A: 10 \| ten

It's right there in the introduction.

29\. Q:  Which Bell Labs location is HL?

A: short hills

Easy for Labbers, not so easy for others. MH was Murray Hill.

30\. Q:  Who mailed out the Sixth Edition tapes?

A: biren \| irma

Packages from Irma Biren were very popular in some circles.

31\. Q:  Which University stole Unix by phone?

A: waterloo

Software piracy by modem. I'm not telling.

32\. Q:  Who received the first rubber chicken award?

A: mumaugh

I don't remember what this was about, and the on-line references are all about this quiz.

33\. Q:  Name a feature of C not in Kernighan and Ritchie.

A: enum \| structure assignment \| void

K&R (pre-ANSI) was pretty old.

34\. Q:  What company did cbosg!ccf work for?

A: weco \| western

Western Electric. Chuck Festoon was created by Ron Hardin as a political statement about automated document processing.

35\. Q:  What does Bnews do?

A: suck \| gulp buckets

Obscure now, but this was a near-gimme at the time.

36\. Q:  Who said "Sex, Drugs, and Unix?"

A: tilson

Mike Tilson handed out these badges at an earlier USENIX. They were very popular, although the phrase doesn't scan in proper Blockhead style.

37\. Q:  What law firm distributed Empire?

A: dpw \| davis&polk&wardwell

Peter Langston worked for this Manhattan law firm, whose computer room had a charming view of FDR Drive.

38\. Q:  What computer was requested by Ken Thompson, but refused by management?

A: pdp-10 \| pdp10

Sic.

39\. Q:  Who is the most obsessed private pilot in USENIX?

A: goble \| ghg

Cruel but fair.

40\. Q:  What operating system runs on the 3B-20D?

A: dmert \| unix/rtr

DMERT gives you dial tone. The D stands for dual, as in dual-processor for redundancy. In a particular Bell System way, it was an awesome machine.

41\. Q:  Who wrote find(1)?

A: haight

Dick Haight also wrote cpio(1), which to my knowledge was the first Unix program that did nothing at all unless you gave it options.

42\. Q:  In what year did Bell Labs organization charts become proprietary?

A: 83

And soon after, PJW appeared.

43\. Q:  What is the Unix epoch in Cleveland?

A: 1969 & dec & 31 & 19:00

Easy.

44\. Q:  What language preceded C?

A: nb

Between B and C was NB. B was interpreted and typeless. NB was compiled and barely typed. C added structs and became powerful enough to rewrite the Unix kernel.

45\. Q:  What language preceded B?

A: bon \| fortran

BCPL is not the right answer.

46\. Q:  What letter is mispunched by bcd(6)?

A: r

This trick question was used to verify that a Unix knock-off was indeed a clean-room reimplementation. Or maybe they just fixed the bug.

47\. Q:  What terminal does the Blit emulate?

A: jerq

Despite what Wikipedia claims at the time I'm writing this, the Blit did not boot up with support for any escape sequences.

48\. Q:  What does "trb" stand for (it's Andy Tannenbaum's login)?

A: tribble

I honestly never knew why this moniker was applied to Andy (no, the other one), although I can guess.

49\. Q:  allegra!honey is no what?

A: lady

Peter Honeyman is many things, but ladylike, no.

50\. Q:  What is the one-line description in vs.c?

A: screw works interface

From the man page for the driver for the Votrax speech synthesizer.

51\. Q:  What is the TU10 tape boot for the PDP-11/70 starting at location 100000 octal?

A: 012700 172526 010040 012740 060003 105710 012376 005007

It's in the book. There was a sad time when I not only had this memorized, it was in muscle memory.

52\. Q:  What company owns the trademark on Writer's Workbench Software?

A: at & t communications

AT&T never could decide what it was to call itself.

53\. Q:  Who designed Belle?

A: condon \| jhc

Joe Condon, hardware genius, died just a few months ago. Belle was the first computer to achieve international master status, using (roughly speaking) Joe's hardware and Ken's software.

54\. Q:  Who coined the name "Unix"?

A: kernighan \| bwk

This one is well known.

55\. Q:  What manual page mentioned Urdu?

A: typo

I miss the typo command and occasionally think of recreating it.

56\. Q:  What politician is mentioned in the Unix documentation?

A: nixon

The Nixon era was a dark period for people maintaining time zone tables.

57\. Q:  What program was compat(1) written to support?

A: zork \| adventure

My memory is rusty on this one and I don't have the 6th Edition manual to hand.

58\. Q:  Who is "mctesq"?

A: michael & toy & esquire

Michael Toy wrote rogue...

59\. Q:  What was "ubl"?

A: rogue \| under bell labs

... which Peter Weinberger renamed ubl when he imported it. The renaming made sense to anyone who worked at Murray Hill.

60\. Q:  Who bought the first commercial Unix license?

A: rand

The RAND Corporation led the way.

61\. Q:  Who bought the first Unix license?

A: columbia

Columbia University.

62\. Q:  Who signed the Sixth Edition licenses?

A: shahpazian

He was the lawyer who literally signed off on the licenses.

63\. Q:  What color is the front console on the PDP-11/45 (exactly)?

A: puce

That's what DEC called it. It's not puce at all, which is why it's a good trivia question.

64\. Q:  How many different meanings does Unix assign to '.'?

A: lots \| many \| countless \| myriad \| thousands

Ken's favorite character on the keyboard.

65\. Q:  Who said "Smooth rotation butters no parsnips?"

A: john & tukey

John Tukey discovered/invented the Fast Fourier Transform algorithm, coined the term "bit" for Claude Shannon, and of course uttered this unforgettable gem.

66\. Q:  What was the original name for cd(1)?

A: ch

You answered chdir, didn't you? You were wrong.

67\. Q:  Which was the first edition of the manual to be typeset?

A: 4 \| four

The old phototypesetting process was much smellier than a modern printer.

68\. Q:  Which was the first edition of Unix to have standard error/diagnostic output?

A: 5 \| five

The idea came remarkably late. Also, back then shell scripts bound standard input to the script, not the terminal.

69\. Q:  Who ran the first Unix Support Group?

A: maranzano

USG was a force for internal and commercial Unix development in AT&T.

70\. Q:  Whose Ph.D. thesis concerned Unix paging?

A: ozalp & babaoglu

The name was well known; the trick was spelling it correctly when you were in a hurry to get home.

71\. Q:  Who (other than the obvious) designed the original Unix file system?

A: canaday

Rudd Canaday was there at the beginning.

72\. Q:  Who wrote the PWB shell?

A: mashey

John Mashey, inventor of the pump operator.

73\. Q:  Who invented uucp?

A: lesk

In so doing, Mike Lesk created the job category of (amateur) network operations engineer.

74\. Q:  Who thought of PWB?

A: evan ivie

Evan Ivie pushed on the Software Tools metaphor to instigate the Programmer's Workbench. It feels quaint now.

75\. Q:  What does grep stand for?

A: global regular expressions print

I've read countless incorrect etymologies for 'grep'. It's just the ed command g/re/p. The reference here is the spelling on the original man page. That final 's' is key to getting this one right.

76\. Q:  What hardware device does "dsw" refer to?

A: console & 7

The profound "delete from switches" program was in its purest form on the PDP-7 in the First Edition.

77\. Q:  What was the old name of the "/sys" directory?

A: ken

Ken and...

78\. Q:  What was the old name of the "/dev" directory?

A: dmr

... Dennis divided their work this way until the Seventh Edition.

79\. Q:  Who has written many random number generators, but never one that worked?

A: ken \| thompson

Sorry, Ken, but it's true and you know it.

80\. Q:  Where was the first Unix system outside 127?

A: patent

The Bell Labs patent office truly benefited from automated document processing made possible in the early versions of Research Unix. I believe they started using it when the kernel was still in assembler.

81\. Q:  What was the first Unix network?

A: spider

You thought it was Datakit, didn't you? But Sandy Fraser had an earlier project.

82\. Q:  What was the original syntax for ls -l \| pr -h?

A: ls -l>"pr -h"> \| <"ls -l"Notation is important. Ken's introduction of the pipe symbol (not to be confused with the \| of the pattern here) was a masterstroke.

83\. Q:  Why is there a comment in the shell source /\* Must not be a register variable \*/?

A: registers & longjmp

You should be able to understand this.

84\. Q:  What is it you're not expected to understand?

A: 6 \| 5 & process

What's amazing to me now is hard this was to understand, let alone to invent, yet within a few years we all realized it could be done almost trivially with setjmp and longjmp.
