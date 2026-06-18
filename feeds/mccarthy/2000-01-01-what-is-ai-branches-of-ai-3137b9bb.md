---
title: "What is AI? / Branches of AI"
url: http://jmc.stanford.edu/artificial-intelligence/what-is-ai/branches-of-ai.html
published: "2000-01-01T00:00:00Z"
feed: mccarthy
guid: http://jmc.stanford.edu/artificial-intelligence/what-is-ai/branches-of-ai.html
---

# What is AI? / Branches of AI

Professor John McCarthy

Father of AI

What is AI? / Branches of AI

Q. What are the branches of AI?

A. Here's a list, but some branches are surely missing, because
no-one has identified them yet. Some of these may be regarded as
concepts or topics rather than full branches.

Logical AI
What a program knows about the world in general
the facts of the specific situation in which it must act, and its
goals are all represented by sentences of some mathematical
logical language. The program decides what to do by inferring
that certain actions are appropriate for achieving its goals.
The first article proposing this was [McC59]. [McC89]
is a more recent summary. [McC96b] lists some of the
concepts involved in logical aI. [Sha97] is an
important text.

Search
AI programs often examine large numbers of
possibilities, e.g. moves in a chess game or inferences by a
theorem proving program. Discoveries are continually made about
how to do this more efficiently in various domains.

Pattern recognition
When a program makes observations of
some kind, it is often programmed to compare what it sees with a
pattern. For example, a vision program may try to match a
pattern of eyes and a nose in a scene in order to find a face.
More complex patterns, e.g. in a natural language text, in a
chess position, or in the history of some event are also
studied. These more complex patterns require quite different
methods than do the simple patterns that have been studied the
most.

Representation
Facts about the world have to be represented
in some way. Usually languages of mathematical logic are used.

Inference
From some facts, others can be inferred.
Mathematical logical deduction is adequate for some purposes, but
new methods of non-monotonic inference have been added to
logic since the 1970s. The simplest kind of non-monotonic
reasoning is default reasoning in which a conclusion is to be
inferred by default, but the conclusion can be withdrawn if there
is evidence to the contrary. For example, when we hear of a
bird, we man infer that it can fly, but this conclusion can be
reversed when we hear that it is a penguin. It is the
possibility that a conclusion may have to be withdrawn that
constitutes the non-monotonic character of the reasoning.
Ordinary logical reasoning is monotonic in that the set of
conclusions that can the drawn from a set of premises is a
monotonic increasing function of the premises. Circumscription
is another form of non-monotonic reasoning.

Common sense knowledge and reasoning
This is the area in
which AI is farthest from human-level, in spite of the fact that
it has been an active research area since the 1950s. While there
has been considerable progress, e.g. in developing systems of
non-monotonic reasoning and theories of action, yet more
new ideas are needed. The Cyc system contains a large but spotty
collection of common sense facts.

Learning from experience
Programs do that. The approaches
to AI based on connectionism and neural nets specialize in
that. There is also learning of laws expressed in logic.
[Mit97] is a comprehensive undergraduate text on
machine learning. Programs can only learn what facts or
behaviors their formalisms can represent, and unfortunately
learning systems are almost all based on very limited abilities
to represent information.

Planning
Planning programs start with general facts about
the world (especially facts about the effects of actions), facts
about the particular situation and a statement of a goal. From
these, they generate a strategy for achieving the goal. In the
most common cases, the strategy is just a sequence of actions.

Epistemology
This is a study of the kinds of knowledge that
are required for solving problems in the world.

Ontology
Ontology is the study of the kinds of things that
exist. In AI, the programs and sentences deal with various kinds
of objects, and we study what these kinds are and what their
basic properties are. Emphasis on ontology begins in the 1990s.

Heuristics
A heuristic is a way of trying to discover
something or an idea imbedded in a program. The term is used
variously in AI. Heuristic functions are used in some
approaches to search to measure how far a node in a search tree
seems to be from a goal. Heuristic predicates that
compare two nodes in a search tree to see if one is better than
the other, i.e. constitutes an advance toward the goal, may be
more useful. [My opinion].

Genetic programming
Genetic programming is a technique for
getting programs to solve a task by mating random Lisp programs
and selecting fittest in millions of generations. It is being
developed by John Koza's group and here's a
tutorial.

Go to next page on Applications of AI.

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
