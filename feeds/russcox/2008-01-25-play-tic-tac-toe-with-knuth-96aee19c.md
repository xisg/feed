---
title: Play Tic-Tac-Toe with Knuth
url: https://research.swtch.com/tictactoe
published: "2008-01-25T05:00:00Z"
feed: russcox
guid: tag:research.swtch.com,2012:research.swtch.com/tictactoe
---

# Play Tic-Tac-Toe with Knuth

Section 7.1.2 of the **[Volume 4 pre-fascicle 0A](http://www-cs-faculty.stanford.edu/~knuth/taocp.html#vol4)** of Donald Knuth's _The Art of Computer Programming_ is titled “Boolean Evaluation.” In it, Knuth considers the construction of a set of nine boolean functions telling the correct next move in an optimal game of tic-tac-toe. In a footnote, Knuth tells this story:

> This setup is based on an exhibit from the early 1950s at the Museum of Science and Industry in Chicago, where the author was first introduced to the magic of switching circuits. The machine in Chicago, designed by researchers at Bell Telephone Laboratories, allowed me to go first; yet I soon discovered there was no way to defeat it. Therefore I decided to move as stupidly as possible, hoping that the designers had not anticipated such bizarre behavior. In fact I allowed the machine to reach a position where it had two winning moves; and it seized _both_ of them! Moving twice is of course a flagrant violation of the rules, so I had won a moral victory even though the machine had announced that I had lost.

That story alone is fairly amusing. But turning the page, the reader finds a quotation from Charles Babbage's _[Passages from the Life of a Philosopher](http://onlinebooks.library.upenn.edu/webbin/book/lookupid?key=olbp36384)_, published in 1864:

> I commenced an examination of a game called “tit-tat-to” ... to ascertain what number of combinations were required for all the possible variety of moves and situations. I found this to be comparatively insignificant. ... A difficulty, however, arose of a novel kind. When the automaton had to move, it might occur that there were two different moves, each equally conducive to his winning the game. ... Unless, also, some provision were made, the machine would attempt two contradictory motions.

The only real winning move is not to play.
