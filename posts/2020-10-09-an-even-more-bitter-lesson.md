---
layout: post
title:  "An Even More Bitter Lesson"
date:   2020-10-09 00:00:01
permalink: /an-even-more-bitter-lesson/
---

Many readers will have seen Richard Sutton's short 2019 post on [The Bitter Lesson](http://www.incompleteideas.net/IncIdeas/BitterLesson.html)
with its two main conclusions:

1. general purpose methods that scale with increased computation are powerful
2. the contents of minds are extremely complex.

I agree with these claims, yet I think there are issues with the broader argument in the post, even beyond those
discussed by Rodney Brooks in his short response [A Better Lesson](https://rodneybrooks.com/a-better-lesson/).

The biggest issue is: *board game solvers should not be the reference point for intelligent systems!*
Contra Big BoardGame's endless propaganda, they are not prototypical AI domains.
The sub-communities that happen to wave the "AI flag" at any given time are arbitrary, and sometimes even worse than arbitary
since tools that solve real-world problems have other ways of justifying themselves.

Here is a thought experiment to counteract this bias: what are some technologies that we don't usually associate with AI that arguably fall under its umbrella?

1. Hardware (analogous to the physical substrate of brains)
2. Operating systems (hindbrains)
3. Networks (nervous system, communication within brains)
4. Databases (memory, retrieval, some basic reasoning)
5. Programming languages (purpose is to understand/communicate high-level abstractions)
6. SMT solvers (reasoning)
7. ...

The list could include almost any subfield of computer science. Computer graphics? Humans seem to have renderers in their head and to use them for mental simulations.
As alluded to above, one reason none of these technologies are associated with AI is that *they work*, i.e. they solve important real-world problems.

Here is a lesson that is even more bitter than the one Sutton describes: *if we didn't manually engineer extremely complex software systems all the time,
we would still be in the Dark Ages of computing*.
