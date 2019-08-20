---
layout: post
title:  "Abstractions in mathematics"
date:   2018-07-21 00:00:01
---
The word "abstraction" is vague in most contexts within the AI
community. Has AlphaGo learned abstractions? I don't think the
question is relevant.  AlphaGo clearly represents some knowledge of
the game in its neural weights, but its interface with the world is
purely through selecting Go moves. It doesn't need to represent any
specific type of abstraction in any specific way to play at a
superhuman level.

In contrast, I consider the fundamental challenge of mathematical
theorem proving to be that:
*sub-astronomic-sized proofs do not even exist unless appropriate abstractions are explicitly reified in the logic*.
Given a fixed set of abstractions, there is only a small set of theorems can feasibly be proved; worse, there are only a small number of theorems that can even feasibly be *stated*. The main challenge of mathematics is to develop the abstractions that permit short statements and proofs of the phenomena you want to establish.


Let's make "abstraction" precise. There are three types of abstractions in mathematics:
- Definitional abstractions
- Declarative abstractions
- Procedural abstractions

A *definitional abstraction* is a new function that stands for a concept. For example, *prime number* is a definitional abstraction, since it corresponds to defining a function *prime* as follows:

\\[ \text{definition prime}(n : \mathbb{N}) := n > 1 \wedge \forall m, \text{divides}(m, n) \to m = 1 \vee m = n \\]

Note that the body of a definition can depend on other definitional
abstractions (e.g. *prime* depends on *divides*). Towers of definitions that
depend on each other allow exponential compression of theorem
statements. Even the
*statement* of the trivial theorem \\( (0 : \mathbb{R}) = (0 :
\mathbb{R}) \\) is unimaginably vast without definitional abstractions,
since without the abstraction of \\( \mathbb{R} \\) it would involve
e.g. equivalence classes of Cauchy sequences, which in turn require
more abstractions to state concisely, until eventually reaching the
primitives of set theory.

A *declarative abstraction* is a theorem that has been proved, for example:

\\[ \forall (n : \mathbb{N}), n > 1 \wedge \neg \text{prime}(n) \to \exists m, m > 1 \wedge m < n \wedge \text{divides}(m, n) \\]

The proofs of such theorems may use other theorems, and so towers of
declarative abstractions that depend on each other allow exponential
compression of proofs, on top of the compression acheived by
definitional abstractions.

A *procedural* abstraction is a program (also called
a *tactic*) that lives outside the logic itself and that
automates some aspect of the proving process, in either a complete or
a heuristic way. An example tactic is one that decides if two
arithmetic expressions are equal modulo associativity and
commutativity by sorting both sides and comparing for syntactic
equality. Without such a procedural abstraction, one would need to
tediously rewrite with the commutativity and associativity lemmas a
number of times that scales superlinearly with the size of the terms.

Unlike the first two types of abstractions, procedural abstractions do
not affect which theorems have sub-astronomic sized proofs in the
underlying logic. Specifically, these procedures
simply *construct* the proofs with fewer meta-logical actions.
However, procedural abstractions can still dramatically reduce the
size of the proof *script*, i.e. the sequence of tactics that
must be invoked to construct the proof.

I think all approaches to using machine learning to help prove
mathematical theorems that ignore this fundamental challenge will
always be severely limited.
