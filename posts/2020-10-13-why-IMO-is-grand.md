---
layout: post
title:  "Why IMO is Grand"
date:   2020-10-13 00:00:01
permalink: /why-IMO-is-grand/
---

Last year a handful of colleagues and I proposed a [grand challenge](https://imo-grand-challenge.github.io/): build an AI that can
win a gold medal in the International Mathematical Olympiad (IMO).
Like Chess before it, the IMO is a proxy task that is not of direct practical import
and thus its importance is not self-evident.

I think the IMO Grand Challenge is extremely important due to a confluence of factors:

1. It is impossible with today's technology---so impossible that we have been all but ridiculed by people in multiple subfields for even suggesting it as a goal.
2. It is probably impossible to "hack" it without developing powerful general-purpose methods. A winning system could likely be repurposed to revolutionize mathematics and software engineering.
3. There is major uncertainty about what *paradigm* could possibly win.

The last point is the most important. Ultimately the IMO Grand Challenge is a battle of AI paradigms, a battle of ideas.

Of course, the zeitgeist question is: what is the proper role of machine learning in building intelligent systems?
Related questions: what is the proper role of symbols, and of humans?

I can imagine several resolutions to these questions:

- *ML-topia*. A massive neural network is trained on all the world's information, then is given the LaTeX of the IMO problems
and spits out a brilliant informal proof. My entire research career has been based on the assumption that this approach will not work.
Despite the amazing successes of modern ML, I still see no evidence that this approach is plausible.
If it can win a gold medal within ten years, I will resign.

- *ML-kernel*. Here the network operates in formal logic rather than informal logic so that search
can be performed over the network's guesses without humans in the loop. This approach requires formalizing all the lemmas
that might be needed within the formal proofs. We use the name *kernel* to stress that the only role of the formal system is to verify and that it is not being used to automate.
I don't think this approach makes sense because formal proofs can be enormous and there are well-established procedures that can efficiently construct them from much smaller certificates.

- *ML-solver*. Instead of producing formal proofs directly, here the neural network's role is to split a problem into a sequence of easier problems that a black-box solver
(e.g. a proof-producing Z3 with 100ms timeout) can solve. I believe that even with existing solvers, this proof calculus is sufficient for short proofs to exist for all IMO problems.
I find it plausible but unlikely that a neural network could supply the necessary decomposition. It would be amazing if it could.

- *Solver-ML*. Here a general-purpose solver such as Z3 proves the theorems, but the solver may be extended internally to make use of a neural network's guidance.
I find this implausible since traditional solver technology seems to be plateauing far below IMO level,
and even sophisticated neural networks will struggle when crammed into SMT/superposition action spaces.

- *Strategy-ML*. Here humans write (potentially domain-specific) strategies to define search spaces that neural networks help search. This is the approach
I outlined in my [recent talk](https://www.youtube.com/watch?v=GtAo8wqWHHg). I think this approach is plausible, and is more likely to work than *ML-solver*.
On the other hand, it is not quite as "pure" as the *ML-solver* approach, and its applicability to other domains will depend on the cost of writing the
strategies and how specialized they are.

- *ML-Tactic*. Here humans write (potentially domain-specific) tactics while ML may select top-level tactics in an outer loop.
I think it is impossible to hand-engineer the heuristics necessary to develop sufficiently good tactic primitives for ML to select from.

- *Not-yet-conceived*. Here the winning paradigm has not yet been discovered.

Out of these 7, I think that only 3 are plausible: *ML-solver*, *Strategy-ML*, and *Not-yet-conceived*.

The IMO Grand Challenge will not be the last word for AI either way, but it could provide a uniquely authorative answer for the proper role of learning, symbols, and humans
in building intelligent systems.
