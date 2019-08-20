---
layout: post
title:  "Issues with AlphaZero"
date:   2018-07-21 00:00:01
---
The AlphaZero algorithm developed by David Silver, Julian
Schrittwieser, Karen Simonyan and others at DeepMind has proved very
successful in the board games Go, Chess, and Shogi. I personally found
the algorithm very elegant upon first reading. However, when I
implemented it on a different problem, it failed catastrophically
until I implemented a few tricks and tuned the hyperparameters
carefully. I still use the algorithm, but I think the failure modes I
stumbled on reflect a (perhaps obvious and practically irrelevant)
conceptual flaw in the algorithm.

Suppose we have a deterministic MDP where a state is a bit string and
the two actions are to append either 0 or 1 to the end of the state.
Suppose the initial state is the empty string \\( \epsilon \\), and that for
some \\(n\\), there is one unknown sequence of bits (e.g. \\(1^n\\)) that
terminates with a reward of 1, and all other n-bit sequences terminate
with no reward.  Suppose \\(n = 2\\), so the agent need only append 1
twice to maximize the reward.

Now let's consider what AlphaZero might do in this environment.
First, it queries the neural network \\( (P(\epsilon, 0), P(\epsilon, 1), V(\epsilon)) = f(\epsilon)
\\) on the root node \\( \epsilon \\) of the search tree. Suppose by bad
luck \\( P(\epsilon, 0) \\) is greater than \\( P(\epsilon, 1) \\).

Now, for \\(K\\) iterations, the algorithm chooses a new leaf node
of the search tree to expand (i.e. to query the network on) by
traversing the existing search tree starting from the root until
reaching a leaf node. On each traversal, it selects actions according
to the following formula:

\\[
\arg\max_a Q(s, a) + c\_{puct} P(s, a) \frac{\sqrt{\sum\_b n\_{v}(s, b)}}{1 + n\_{v}(s, a)}
\\]

where \\( Q(s, a) = 0 \\) when \\( n\_v(s, a) = 0 \\).  Since \\(
P(\epsilon, 0) > P(\epsilon, 1) \\), it expands 0 first. Since the rewards are all between 0 and 1, the network will return a value estimate \\( V(0) > 0 \\), which will become \\( Q(\epsilon, 0) \\). This gives action 0 an even stronger grip on the root node, and depending on \\( c\_{puct} \\) and the ratio \\( P(\epsilon, 0) / P(\epsilon, 1) \\), it may require many rollouts before it even tries taking action 1 at the root node.


Suppose that the number of expansions \\( K \\) is too small relative to
\\( c_{puct} \\) and the ratio \\( P(\epsilon, 0) / P(\epsilon, 1) \\), so that
it never visits action 1 at the root node before commiting to action 0.
This is still reasonable behavior so far, since it has simply
gotten unlucky.

However, what the algorithm does next is _not_ reasonable. It
uses the visit counts \\( n\_v(\epsilon, 0) = K \\) and \\( n\_v(\epsilon,
1) = 0 \\) to determine the distribution \\( \pi(\epsilon) =
(\pi(\epsilon, 0) = 1.0, \pi(\epsilon, 1) = 0.0) \\) the neural network
policy ought to have omitted, and trains the neural network to emit \\(
\pi \\) instead of \\( P \\). The algorithm reinforces the arbitrary
decision it made in the first episode, and _never_ stumbles on
the winning state 11.

There are many ways to modify the algorithm to avoid the issue in this
example (e.g. by boosting the scores of all untried actions, by adding
a large amount of Dirichlet noise to \\( P(s, \cdot) \\), by making \\( K
\\) and \\( c_{puct} \\) larger, etc.), but I think this example
illustrates a fundamental problem with the algorithm: reinforcing the
visit counts only makes sense as \\( K \\) goes to infinity. It is only
justified by asymptotic arguments, and can result in pathological
behavior in the case of finite \\( K \\).

I think the main conceptual flaw of the algorithm is that
it conflates the behavior of the exploration strategy with the
information gathered by the exploration. One can imagine different
exploration strategies with different tradeoffs, and one doesn't want
to blindly train the network to reinforce what the exploration
strategy _did_; one wants to update it based on what the
exploration strategy _learned_.
