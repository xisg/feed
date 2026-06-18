---
title: "Formalization of Strips in Situation Calculus"
url: http://jmc.stanford.edu/articles/strips.html
published: "1985-01-01T00:00:00Z"
feed: mccarthy
guid: http://jmc.stanford.edu/articles/strips.html
---

# Formalization of Strips in Situation Calculus

Professor John McCarthy

Father of AI

## Articles

### Formalization of Strips in Situation Calculus

This is a 1985 note aimed at regarding STRIPS as a proof strategy for an interactive theorem prover using a situation calculus formalism. It doesn't quite get there.
STRIPS is a planning system that uses logical formulas to represent information about a state. Each action has a precondition, an add list and a delete list. When an action is considered, it is first determined whether its precondition is satisfied. This can be done by a theorem prover, but my understanding is that the preconditions actually used are simple enough that whether one is true doesn't require substantial theorem proving. If the precondition isn't met, then another action must be tried. If the precondition is met, then then sentences on the delete list are deleted from the database and sentences on the add list are added to it.

STRIPS was considered to be an improvement on earlier systems using the situation calculus, because these earlier systems ran too slowly.

Download the article in PDF.

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
