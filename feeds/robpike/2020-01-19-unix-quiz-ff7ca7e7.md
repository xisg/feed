---
title: Unix Quiz
url: https://commandcenter.blogspot.com/2020/01/unix-quiz.html
published: "2020-01-19T19:52:00Z"
feed: robpike
guid: tag:blogger.com,1999:blog-6983287.post-3363467847655099343
---

# Unix Quiz

(Here's another resurrected post from April 30, 2012. Answers in a followup.)

People objected that there was no Exit item on the main menu for the mpx program that put windows on the Blit; see [http://en.wikipedia.org/wiki/Blit\_(computer\_terminal)](http://en.wikipedia.org/wiki/Blit_%28computer_terminal%29) (which mistakenly says it implemented cursor addressing when turned on - as if!) and [http://www.cs.bell-labs.com/cm/cs/doc/83/mpx.ps.gz](http://www.cs.bell-labs.com/cm/cs/doc/83/mpx.ps.gz). It seemed unnecessary, since you could just power cycle. Why clutter the menu? (Those were simpler times.)

After hearing too much complaining, I decided to implement Exit, but did it a special way. Late one night, with help from Brian Redman (ber) and Pat Parseghian (pep), I cranked out a set of trivia questions to drive the Exit control. Answer the question right, you can exit; get it wrong, you're stuck in mpx for a little longer. To make this worthwhile, the questions had to be numerous and hard, and had to be verified by the machine, so the quiz code included a little pattern matcher. It also had to be tiny, since the machine only had 256KB and the display took 100KB of that. (Those were simpler times.)

The response was gratifying. I'll never forget seeing someone, who shall remain nameless, a vociferous complainer about the lack of Exit, burble with excitement when he saw the menu item appear, only to crumble in despair when the question arrived. I forget which question it was, but it doesn't matter: they're all hard.

The questions were extended by lots of suggestions from others in the Unix lab, and then in 1984 they were handed out as a bloc in a trivia contest at the USENIX conference in Salt Lake City. To quote an observer, "The submission with the most correct answers (60) was from a team comprising David Tilbrook, Sam Leffler, and presumably others. Jim McKie had the best score for an individual (57) and was awarded an authenticated 1972 DECtape containing Unix Version 2. Finally, Ron Gomes had 56 correct answers and received an original engraved "Bill Joy" badge, which once belonged to Bill himself, from Sun Microsystems." That score of 57 was so impressive we hired Jim a little later, but that's another story.

How much Unix trivia do you know? Test your mettle; the questions appear below. This may be one of the hardest quizzes ever to originate outside of King William's College.

I've disabled comments because people will just send in spoilers. If you want to discuss or collaborate, do so elsewhere. I'll publish the computer-readable, pattern-matching answers here in a few days.

Good luck, and may your TU-10 never break your 9-track boot tape.

-rob

1\. The source code motel: your source code checks in, but it never checks out. What is it?

2\. Who wrote the first Unix screen editor?

3\. Using TSO is like kicking a {what?} down the beach.

4\. What is the filename created by the original dsw(1)?

5\. Which edition of Unix first had pipes?

6\. What is =O=?

7\. Which Stephen R. Bourne wrote the shell?

8\. Adam Buchsbaum's original login was sjb. Who is sjb?

9\. What was the original processor in the Teletype DMD-5620?

10\. What was the telephone extension of the author of mpx(2)?

11\. Which machine resulted in the naming of the "NUXI problem"?

12\. What customs threat is dangerous only when dropped from an airplane?

13\. Who wrote the Bourne Shell?

14\. What operator in the Mashey shell was replaced by "here documents"?

15\. What names appear on the title page of the 3.0 manual?

16\. Sort the following into chronological order: a) PWB 1.2, b) V7, c) Whirlwind, d) System V, e) 4.2BSD, f) MERT.

17\. The CRAY-2 will be so fast it {what?} in 6 seconds.

18\. How many lights are there on the front panel of the original 11/70?

19\. What does FUBAR mean?

20\. What does "joff" stand for?

21\. What is "Blit" an acronym of?

22\. Who was rabbit!bimmler?

23\. Into how many pieces did Ken Thompson's deer disintegrate?

24\. What name is most common at USENIX conferences?

25\. What is the US patent number for the setuid bit?

26\. What is the patent number that appears in Unix documentation?

27\. Who satisfied the patent office of the viability of the setuid bit patent?

28\. How many Unix systems existed when the Second Edition manual was printed?

29\. Which Bell Labs location is HL?

30\. Who mailed out the Sixth Edition tapes?

31\. Which University stole Unix by phone?

32\. Who received the first rubber chicken award?

33\. Name a feature of C not in Kernighan and Ritchie.

34\. What company did cbosg!ccf work for?

35\. What does Bnews do?

36\. Who said "Sex, Drugs, and Unix?"

37\. What law firm distributed Empire?

38\. What computer was requested by Ken Thompson, but refused by management?

39\. Who is the most obsessed private pilot in USENIX?

40\. What operating system runs on the 3B-20D?

41\. Who wrote find(1)?

42\. In what year did Bell Labs organization charts become proprietary?

43\. What is the Unix epoch in Cleveland?

44\. What language preceded C?

45\. What language preceded B?

46\. What letter is mispunched by bcd(6)?

47\. What terminal does the Blit emulate?

48\. What does "trb" stand for (it's Andy Tannenbaum's login)?

49\. allegra!honey is no what?

50\. What is the one-line description in vs.c?

51\. What is the TU10 tape boot for the PDP-11/70 starting at location 100000 octal?

52\. What company owns the trademark on Writer's Workbench Software?

53\. Who designed Belle?

54\. Who coined the name "Unix"?

55\. What manual page mentioned Urdu?

56\. What politician is mentioned in the Unix documentation?

57\. What program was compat(1) written to support?

58\. Who is "mctesq"?

59\. What was "ubl"?

60\. Who bought the first commercial Unix license?

61\. Who bought the first Unix license?

62\. Who signed the Sixth Edition licenses?

63\. What color is the front console on the PDP-11/45 (exactly)?

64\. How many different meanings does Unix assign to '.'?

65\. Who said "Smooth rotation butters no parsnips?"

66\. What was the original name for cd(1)?

67\. Which was the first edition of the manual to be typeset?

68\. Which was the first edition of Unix to have standard error/diagnostic output?

69\. Who ran the first Unix Support Group?

70\. Whose Ph.D. thesis concerned Unix paging?

71\. Who (other than the obvious) designed the original Unix file system?

72\. Who wrote the PWB shell?

73\. Who invented uucp?

74\. Who thought of PWB?

75\. What does grep stand for?

76\. What hardware device does "dsw" refer to?

77\. What was the old name of the "/sys" directory?

78\. What was the old name of the "/dev" directory?

79\. Who has written many random number generators, but never one that worked?

80\. Where was the first Unix system outside 127?

81\. What was the first Unix network?

82\. What was the original syntax for ls -l \| pr -h?

83\. Why is there a comment in the shell source /\* Must not be a register variable \*/?

84\. What is it you're not expected to understand?
