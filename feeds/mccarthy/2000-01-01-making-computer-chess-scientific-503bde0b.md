---
title: "Making Computer Chess Scientific"
url: http://jmc.stanford.edu/articles/chess.html
published: "2000-01-01T00:00:00Z"
feed: mccarthy
guid: http://jmc.stanford.edu/articles/chess.html
---

# Making Computer Chess Scientific

Professor John McCarthy

Father of AI

## Articles

### Making Computer Chess Scientific

This essay is an outgrowth of my review of Monty Newborn's Kasparov versus Deep Blue: Computer Chess Comes of Age.
I complained in my Science review of Monty Newborn's Deep Blue vs. Kasparov that the tournament oriented work on computer chess was not contributing as much to the science of AI as it should.

AI has two tools for tackling problems. One is to use methods observed in humans, often observed only by introspection, and the other is to invent methods using ideas of computer science without worrying about whether humans do it this way. Chess programming employs both. Introspection is an unreliable way of determining how humans think, but introspectively suggested methods are valid as AI if they work.

Much of the mental computation done by chess players is invisible to the player and to outside observers. Patterns in the position suggest what lines of play to look at, and the pattern recognition processes in the human mind seem to be invisible to that mind. However, the parts of the move tree that are examined are consciously accessible.

It is an important advantage of chess as a Drosophila for AI that so much of the thought that goes into human chess play is visible to the player and even to spectators. When chess players argue about what is the right move in a position, they follow out lines of play, i.e. argue explicitly about parts of the move tree. Moreover, when a player is found to have made a mistake, it is almost always a failure to follow out a certain line of play rather than a misevaluation of a final position.

A person can examine only a much smaller fraction of the move tree than even the computers of 40 years ago. The ability of a chess master, or even an ordinary player, to look deeply into the move tree depends on his ability to decide that almost all moves are not worth examining. Because it is not visible what moves a player does not examine, how a master decides what not to examine has not been studied. When a superior player defeats an inferior, it would be worthwhile to understand why the master did not examine lines of play on which the inferior player wasted his time.

How to avoid wasting time on fruitless lines of investigation is important for success in every form of computer reasoning, e.g. mathematical theorem proving and problem solving on the basis of logical description of real world situations. Chess is an excellent domain for observing this and for inventing general criteria for ignoring the unimportant. Most likely, a major tool for this will be criteria for judging a move in a position as sufficiently similar to another situation in which the move was examined and found to be unpromising. Note that in an abstract tree there is no correspondence between edges leading out of different edges. In real games, approximate correspondences exist.

The killer heuristic and the anti-killer heuristic which suggest trying or not trying specific moves, i.e. assign properties to specific moves, are probably used by chess players in a more general form. Consider, "He is attacking my queen. If I don't move it or block the attack on it, he will capture it." This is more general than the killer heuristic which suggests trying early the specific move that captured the queen or led to its capture. Studying the abstractions used by chess players, e.g. "capture the undefended piece" will help devise more general ways of representing and inventing appropriate abstractions - and not only in chess.

Another kind of abstraction allows very large reductions in the tree of possibilities. In my imminent note , I discuss how a tree of depth 20 can be reduced to a four node tree by ignoring distinctions between black's moves that are tactically similar and regarding moving the white king along a certain path as a single action as long as black keeps his king in a certain region. This idea, suggested by chess and developed in a chess context also has wider application.

Maybe it will turn out that chess is an inferior tool for understanding these general mechanisms of intelligent behavior. However, people whose interests are attracted to computer chess would do well to consider about how to identify the intellectual mechanisms involved.

These notes lack a bibliography of work using chess as a Drosophila, and the list of authors in my Science Review is inadequate. I hope to repair these deficiencies in future versions of these notes.

"Computer chess" and human chess gives an example of a problem quickly solvable by present chess programs that illustrates the use of ideas required for human level intelligence. The solution to the problem requires the idea of making two threats with one line of play---an idea useful in more domains than chess.

What is AI? is a nontechnical introductory essay.

"I don't see that human intelligence is something that humans can never understand."



Articles

Books and Reviews

Notes on AI

Slides

What is AI?

Top
|
John McCarthy's Original Website
|
We invite you to send comments and feedback
