---
layout: post
title:  "Search vs meta-search"
date:   2018-07-21 00:00:01
---
Consider the following puzzle, from Martin Gardner’s __My Best Mathematical and Logic Puzzles__ (1994):

*Cut Down the Cuts*.

*You have taken up the hobby of jewelry making and you want to join the four pieces of silver chain shown at the left to form the circular bracelet shown at the right. Since it takes a bit of doing to cut open a link and weld it together again, you naturally want to cut as few links as possible. What is the minimum number of links you must cut to do the job?*


<center><img src="/assets/images/gardner_chains.jpg" width="300px"></center>


(Note: the solution will be revealed during this post, so if you would like to solve it first on your own, stop reading.)

Chances are, the first solution you will see requires four cuts. A single cut-and-weld operation allows you to connect two chains together. It requires three such operations to connect all four together in a chain, and a fourth one to connect the two ends of the chain to form a circle.

As you may have guessed (by pragmatic inference, given that it is a puzzle), there is a way to form the circular bracelet that involves cutting only three links. Once you see how to do it, the three-cut solution seems just as easy as the approach discussed above that requires four cuts.

What I find interesting about this problem is how hard it can be to find the three-cut solution. I think the key to understanding this phenomenon is to make a careful distinction between searching for a solution within a problem formulation, and meta-searching for a different problem formulation that yields a simpler solution.

  Let us define a problem formulation to be a formal representation of a search problem. For the purpose of this exercise, we will represent search problems using a standard planning calculus, in terms of a set of objects, a set of relations, a set of actions, an initial state, and a goal state.

  Here is an approximation to the problem formulation I seemed to make (without thinking) when first looking at the puzzle, which leads naturally to the four-cut solution discussed above. I saw each silver chain as a distinct object \\(c_1, c_2, c_3, c_4 \\), and ignored the fact that each silver chain is itself composed of three links. The only property that seemed relevant to me about the chains were which were connected to which others \\(( \text{Connected}(c_i, c_j)) \\), and the only action that seemed applicable was connecting two chains (that were each connected to at most one other chain) by an atomic \\( \text{CutAndWeld}(c_i, c_j) \\) operation. No chains were connected to each other in the initial state, and the goal was to connect them all.

  One can easily make this formalization more precise, enter it into a generic problem solver, and get the following solution:
1. \\( \text{CutAndWeld}(c_1, c_2) \\)
2. \\(\text{CutAndWeld}(c_2, c_3)\\)
3. \\(\text{CutAndWeld}(c_3, c_4)\\)
4. \\(\text{CutAndWeld}(c_4, c_1)\\),

which is exactly the solution discussed above.

  The important point is that *this is the optimal solution given this problem formulation; that is, there is no way to form the circular bracelet using only three cut-and-weld operations*. One of the implications of this is that as long as you cling to this problem formulation, you will not be able to find a solution that involves only three cut operations, no matter how hard you try. In order to solve the puzzle, you need to first relinquish your problem formulation and meta-search for a new one.

  If you compare the original problem (as specified by both the words and the images) to the problem formulation discussed above, you will see that the formulation abstracts away many details of the original problem. In particular, in the original problem, once we cut a link, there is no a priori restriction that we weld it together with one of its old neighbors. However, the above formulation implicitly enforces this restriction by nature of the cut-and-weld operation.

  We can remove this restriction by considering more fine-grained objects (links \\(l_i\\) instead of chains \\( c_j \\)), additional predicates \\( \text{IsCut}(l)\\) and more fine-grained actions \\(\text{Cut}(l) \\) and \\( \text{Weld}(l_i, l, l_j) \\).

One can easily make this formalization more precise as well, enter it into a generic problem solver, and get the following solution:
1. \\( \text{Cut}(l_1) \\)
2. \\(\text{Cut}(l_2)\\)
3. \\(\text{Cut}(l_3)\\)
4. \\(\text{Weld}(l_6, l_1, l_7)\\)
5. \\(\text{Weld}(l_9, l_2, l_{10})\\)
6. \\(\text{Weld}(l_{12}, l_3, l_4)\\),

  which is the desired three-step solution. The English translation is “First split off all the links in one of the chains, and then use the free links to join the remaining chains together in sequence.”

A few takeaways.
1. Humans are incredibly good at building problem formulations on the fly, even when the inputs are informal and multi-modal.
2. The problem formulations we build are <i>abstract</i> and throw away details that may (or may not) be irrelevant to the task at hand.
3. This process is immediate and unconscious, and can be difficult to introspect on.
4. Given a problem formulation, we seem to be able to perform search competently.
5. By asking questions such as 'how else might I go about this?', we can trigger meta-search for different problem formulations.
