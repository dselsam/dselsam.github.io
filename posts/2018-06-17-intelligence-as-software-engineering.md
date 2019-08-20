---
layout: post
title:  "Intelligence as software engineering"
date:   2018-07-21 00:00:01
---
Humans can learn to become competent at playing board games such as
Chess and Go, but humans are *phenomenally excellent* at learning *how
to play* such games quickly. Recent approaches to using reinforcement
learning to play games hardcode simulators for the games, and mostly
ignore what I think is the more important challenge on the path to
AI.

You can teach a child how to play Connect Four in a few minutes, at
which point the child can already perform some kind of model-based
lookahead search. Yet even a game as seemingly trivial as Connect Four
is not trivial to implement a simulator for. Even the code that checks
whether either player has won the game is a little tricky, and there are ample
opportunities for off-by-one errors and out-of-bounds errors.

We can describe variations of the game (e.g. use a diamond-shaped grid
where pieces fall on either diagonal) and instantly build bug-free
internal simulators in our minds that permit lookahead search. Yet
even such a simple change might require a tricky refactor of the
implemented simulator, and the refactoring would be even trickier if
the implementation needed to be general enough to support both
versions, in which case it might even require new classes and
abstractions. And this is a trivial game. We humans can build
simulators in our heads for much more complicated domains, such as
mathematics, that might be legitimately difficult for expert software
engineers to implement at all, let alone in a flexible and
maintainable way.

By what magic are humans so good at this?

It is tempting to think that a human mind has the equivalent of a
large codebase with reusable libraries, and that it builds its Connect
Four simulator by gluing together a generic *Game* class with
a *Grid* class and a *enumerate_connected_sequences*
function that can operate on *Grid*s and that can deal with
diagonals and edge cases correctly, and so forth.  However, every
software engineer knows how incredibly hard it is to design such
flexible and reusable abstractions.

Every programming language makes some kinds of compositions and
abstractions easier than others. Traditional languages such as C++ and
Python make this kind of software engineering possible in principle
but even expert software engineers must design abstractions carefully
for specific problems and find it hard to extend systems for new,
unanticipated use cases. Other kinds of programming languages have
different trade-offs.  The [Game Description
Language (GDL)](https://en.wikipedia.org/wiki/Game_Description_Language)
allows simulators for many finite games like Connect Four to be synthesized from declarative descriptions
and has built-in support for several common game concepts such as players, legal moves, and goals.
However, it is otherwise a very low-level language and has very limited support for building new abstractions.
For example, there is no easy way to declare an \\( m \\times n \\) grid; the [the GDL description of Connect Four](https://github.com/ggp-org/ggp-repository/blob/master/war/root/games/connect4/connect4.kif) requires a distinct logical atom for the existence of every slot in the grid.
Even more exotic kinds of languages such as probabilistic
programming languages (e.g. Church) and differentiable programming
languages (e.g. Tensorflow) make other kinds of software engineering
dramatically easier, but don't seem to help with this kind of programming.


I do not know of any existing kind of programming language
that could make it so easy to build simulators for new games that building such simulators
would be a plausible search problem for an AI system.
I expect the following idea to be controversial, but I sometimes suspect that building AI
may require inventing a new kind of programming language, vastly
superior to those we have developed so far, that makes it much easier
to engineer, refactor and extend complicated software systems. I think
we may need such a language both as a resource to provide our AIs
with, and as an encoding of the inductive biases we want to embue them
with.
