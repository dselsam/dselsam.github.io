---
layout: post
title:  "Mathematics: our overlooked ability"
date:   2018-07-21 00:00:01
---
Suppose you have a list of 25 characters, 13 of which are 'a', and no character ever appears twice in a row. What can you tell me about the list?

Most readers will instantly know that the list starts and ends with 'a', and that it alternates between 'a' and characters besides 'a'. This inference may seem mundane to some of you, yet despite half a century of developing systems for automated and interactive theorem proving, and of designing libraries of definitions, lemmas, and tactics just for reasoning conveniently about lists, it is still surprisingly difficult to construct a formal proof of this basic fact. We humans are *so* good at this type of reasoning that it is easy to overlook how sophisticated our abilities really are.

I have spent a good deal of time trying to formalize elementary mathematics and computer science textbooks in the [Lean Theorem Prover](https://github.com/leanprover/lean), and in trying to extend and improve Lean to make the process easier. I have been able to translate entire chapters of several textbooks into Lean in a natural way, with every line of Lean seemingly isomorphic to the informal presentation. However, once in a while I will hit a statement or proof step that may seem simple to me but that requires a major refactor, or adding new features to Lean itself, or just seems like a brick wall. My brain is able to perform massive refactorings of mathematical knowledge and abstractions, synthesize the equivalent of tens of thousands of lines of tricky and performance-critical software, and maybe even expand the logic I am effectively operating in, all in the blink of an eye.

Our field has little appreciation for these abilities, no explanation of them, and no plausible path to engineering anything like it.

Deep learning doesn't move the needle at all on these problems.
