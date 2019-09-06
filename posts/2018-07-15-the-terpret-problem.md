---
layout: post
title:  "The \"Terpret Problem\" and the limits of SGD"
date:   2018-07-21 00:00:01
permalink: /the-terpret-problem/
---
Many AI researchers (myself included) are optimistic that stochastic gradient descent (SGD) can somehow be leveraged to help learn sophisticated computer programs. My colleagues and I recently showed that SGD over a graph neural network [can learn a local search procedure for the boolean satisfiability problem](https://arxiv.org/abs/1802.03685), even though the function it is trying to learn is extremely sensitive to individual bits in the input.

However, there are many synthesis problems for which discrete search works well but for which SGD fails. [Gaunt et al.](https://arxiv.org/abs/1608.04428) have done us the great service of constructing a minimal example of a synthesis problem that exhibits an exponential number of inadequate local optima, for which SGD consistently gets stuck, even when applying every trick in the book. This minimal example is relatively unknown to the ML community, since it is buried in a dense paper with a forbidding amount of PL jargon. In this post I will explain the example for an ML audience. In honor of the probabilistic programming language, TerpreT, introduced in the paper, I refer to this example as "the TerpreT problem". The code for this post can be found [here](https://github.com/dselsam/terpret_problem).

Suppose we have the following program sketch:

| ------- |:-------------|
| Constants     | `k : int`            |
| Inputs        | `x_0 : bool`         |
| Parameters    | `xs : array bool (k-1)` |
| Outputs       | `ys : array bool k` |

```python
def foo(x_0):
    xs = [x_0] ++ xs
    ys = [xor(xs[i], xs[i+1 % k]) for i in range(k)]
    return ys
```

In words, the program `foo` is parameterized by \\( k-1 \\) booleans, and it connects an input boolean `x_0` to the \\(k-1\\) parameters to form a circular chain of length \\(k\\). Then, it returns the \\(k\\) values resulting from `xor`-ing adjacent elements of the chain.

If we observe the single input-output example where 0 maps to the array with every element 0, we can conclude that all the parameters of the program must be 0. Indeed, constraint-propagation can easily deduce this in linear time. However, we will see shortly that SGD cannot discover this fact.

First, we need to explain how to apply SGD to this problem at all. The first step is to *relax* the program, so that every boolean variable and parameter is represented by its *probability distribution*, and the program itself propagates probability distributions instead of specific boolean values. Then instead of observing that every element of `ys` is 0, we want to minimize the cross-entropy loss between the computed *distribution* of each element of `ys` with the (deterministic) distribution that puts all its mass on 0.

Next, we will use SGD to minimize this loss as a function of the *parameters* of the distribution for each element of `xs`. We follow the convention in the paper of representing the parameters for the Bernoulli distribution of `x_i` by two scalars \\(s_i(0)\\) and \\(s_i(1)\\), such that

\\[
P(x_i = z) = \frac{e^{s_i(z)}}{e^{s_i(0)} + e^{s_i(1)}}
\\]

Clearly the loss function can be made arbitrarily small by making \\(s_i(0)\\) big relative to \\(s_i(1)\\) for all \\(i\\), and clearly this is the only way to make it arbitrarily small. However, when we run SGD on this problem, the loss function almost never goes to 0.

Here is example Tensorflow code (complete code is [here](https://github.com/dselsam/terpret_problem)):

```python
# mu0 : T [1, 2]
# This is the marginal for the variable x0 which is observed to be 0.
mu0 = tf.constant([[1.0, 0.0]], dtype=tf.float32)

# ss : T [k-1, 2]
# These are the parameters we are trying to optimize for, that control the marginal distribution
# of the xs in the chain.
# Note: `SharedLogDirichletInitializer` is the initialization method used in the paper, and its details
# are not important here.
ss = tf.get_variable("ss", shape=[opts.k - 1, 2],
                     initializer=SharedLogDirichletInitializer(opts.alpha, opts.k - 1, 2))

# mus : T [k, 2]
# These are the parameters for the distributions of the full xs chain.
mus = tf.concat([mu0, tf.nn.softmax(ss)], axis=0)

# Returns the probability that `xor(xs[i], xs[i+1%k])` equals `0`
def soft_xor_p0(i):
  j = tf.mod(i+1, opts.k)
  return (mus[i, 0] * mus[j, 0]) + (mus[i, 1] * mus[j, 1])

# ys_eq_0 : T [k]
ys_eq_0 = tf.map_fn(soft_xor_p0, tf.range(opts.k), dtype=tf.float32)
# cross-entropy loss
loss = - tf.reduce_sum(tf.log(ys_eq_0))
update = tf.train.AdamOptimizer(learning_rate=opts.learning_rate).minimize(loss)

for epoch in range(opts.n_epochs):
  _, loss = sess.run([tp.update, tp.loss])
  sys.stdout.write("[%d] %.8f\n" % (epoch, loss))

```

When we run this code, we see that (depending on the size of \\(k\\)) it almost always converges to a suboptimal solution with non-zero loss.
To see what is happening, we can print out \\(P(x_i = 0)\\) for every `x_i` in `xs` at every step in the loop:

```python
for epoch in range(opts.n_epochs):
  _, mus = sess.run([tp.update, tp.mus])
  for i in range(opts.k):
    sys.stdout.write("%d " % (round(100 * mus[i, 0])))
  sys.stdout.write("\n")
```

Each row of the following table shows the local optima found for various random seeds, with \\(k\\) = 16. A 100 indicates 100% probability that the corresponding parameter is set to 0 (see code snippet above for details), so for each row we want every element to be 100.

| x1|  x2 |  x3 |  x4 |  x5 |  x6 |  x7 |  x8 |  x9 |  x10 |  x11 |  x12 |  x13 |  x14 |  x15 |
|--:|----:|----:|----:|----:|----:|----:|----:|----:|----:|----:|----:|----:|----:|----:|
100 | 100 | 100 | 100 | 100 | 100 | 100 |  50 |   0 |   0 |  50 | 100 | 100 | 100 | 100
50  |   0 |   0 |   0 |  50 | 100 | 100 | 100 | 100 | 100 | 100 | 100 | 100 | 100 | 100
100 | 100 | 50  |   0 |   0 |   0 |  0 | 50 | 100 | 100 | 100 | 100 | 100 | 100 | 100
100 | 100 | 50  |   0 |   0 |   0 |  0 | 50 | 100 | 100 | 50 | 0 | 50 | 100 | 100

The pattern is clear: any two `x_i` at 50% create local optima with all elements between them set to 0 instead of 100.
The intuition for this phenemenon is simple: if you only look at your neighbors and they agree, it is best for you to agree with them; if they disagree, it is best for you to hedge your bets between them.

It may be clarifying to see how this emerges from the math.
Let \\(p_i \\) stand for \\( P(x_i = 0) \\).
For a given \\(i\\), the parameters \\(s_i(0)\\) and \\(s_i(1)\\) only contribute to two terms of the loss function:

\\[
\\log \\left( p_{i-1} p_i + (1 - p_{i-1})(1 - p_i) \\right)
+
\\log \\left( p_{i+1} p_i + (1 - p_{i+1})(1 - p_i) \\right)
\\]

After a few steps of simplification, we can express the derivative of this quantity with respect to \\( s_i(0) \\) as follows:

\\[
\\frac{\\partial \\mathcal{L}}{\\partial s_i(0)}
=
p_i (1 - p_i) \\left( \\frac{2 p_{i-1} - 1}{p_{i-1} p_i + (1 - p_{i-1})(1 - p_i)} +
\\frac{2 p_{i+1} - 1}{p_{i+1} p_i + (1 - p_{i+1})(1 - p_i)} \\right)
\\]

When \\( p_i = 0 \\) and \\( p_{i-1}, p_{i+1} < 1 \\), then the partial derivative with respect to \\( s_i(0) \\) is 0.
Similarly, when \\( p_i = 0.5 \\) and \\( (p_{i-1}, p_{i+1}) \\) equal (0.0, 1.0) or (1.0, 0.0), the partial derivative with respect to \\( s_i(0) \\) is also 0. To see that there are an exponential number of such local optima, notice that if we fix every other \\( p_i \\) to either 0 or 1 in any of the \\( O(2^{k/2}) \\) ways with atleast one \\( 0.0 \\), we can set the remaining \\( p_j \\) to construct a local optima.

The authors report trying many optimization heuristics, such as adding random Gaussian noise to the gradients, and adding an exponentially-decayed entropy bonus. However, as \\( k \\) starts to get large (\\( \approx \\)128), no amount of tricks or hyperparameter tuning helped: SGD never found the answer.

In contrast, a SAT solver can immediately deduce that all parameters must be 0 using unit propagation without any search.
