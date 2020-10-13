---
layout: post
title:  "The Inevitable Trust Bottleneck"
date:   2020-10-13 00:00:01
permalink: /the-inevitable-trust-bottleneck/
---

An obvious and important limitation of ML is that since its outputs cannot be trusted,
its usefulness in a domain is upper-bounded by the gap in difficulty between verifying a correct guess and finding a solution through other means.
This is a weak upper-bound in practice, since today's ML is extremely unlikely to make correct guesses in the first place in challenging domains.

Suppose you want to build a compiler from one language to another. You explain the problem in words, and Transformer-&infin; emits
a million lines of Rust for you. What next? Do you try to read the code? Do you ask Transformer-&infin; to explain it to you, to convince you it is correct?
Do you try to write exhaustive unit tests yourself? Do you ask Transformer-&infin; to write exhaustive unit tests for you?

As untrustable methods become increasingly capable in challenging domains, establishing trust inevitably becomes the bottleneck.
Moreover, any approach in which humans must understand solutions in order to trust them is inherently unscalable.
The key question is: *how can we minimize the amount that humans must understand while permitting arbitrarily high levels of trust*?

The field of formal methods offers an answer to this timeless question: in applicable domains, the minimum can often be achieved by writing a formal
theorem whose validity would imply that the candidate solution is adequate. With such a theorem, untrusted code can be used both to
find the candidate solution and to produce a formal (i.e. machine-checkable) proof that it satisfies the correctness theorem.
Crucially, *the formal theorem statement is all that humans need to understand to have near-absolute trust that the candidate solution is adequate*.

As untrustable methods improve over time, the cost of producing formal proofs will decrease and the benefit of writing formal specifications
will increase. In the limit, we may hope that humans need only concern themselves with specifications and can safely relegate the rest to machines.
