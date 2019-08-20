---
layout: post
title:  "Neural networks, Occam's razor, and heuristic search"
date:   2018-07-21 00:00:01
---
Neural networks sacrifice Occam's razor in exchange for the ability to
efficiently find parameters that fit the training data.  This may be a
fantastic trade in many contexts, but unless the training set is very
representative of the test set, the network will likely learn the
wrong thing.

In general, the best way to generalize from a finite set of training
data involves finding the shortest computer program that (efficiently)
explains the training data. Unfortunately, this approach almost never
works in practice because (traditional) programming languages induce
extremely unfavorable search spaces in which small syntactic changes
cause major semantic changes and heuristic search rarely beats
exponential time.

Neural networks reparameterize the space of programs (or for
feed-forward architectures, the space of circuits) in such a way that
small syntactic changes cause commensurately small semantic changes so
that heuristic search (e.g. SGD) has a shot at working reasonably
well.  As a consequence, it is often possible to efficiently find
parameters that explain a given finite set of data, even if the data
is so complex that no (traditional) computer program could be found to
fit it by any known means.

This is great, but there is a catch: the parameters discovered this
way will not generalize to unseen data nearly as well as the shortest
computer program that fit the training data would have.

Let's look at the simplest possible example. Suppose we try to learn the identity
function on scalars \\( f(x : \mathbb{R}) = x \\) with a
2-hidden-layer MLP with 64 hidden units on each layer (full code [here](https://github.com/dselsam/identity)):

```python
parser = argparse.ArgumentParser()
parser.add_argument('--batch_size', action='store', dest='batch_size', type=int, default=128)
parser.add_argument('--n_inputs', action='store', dest='n_inputs', type=int, default=1)
parser.add_argument('--n_nodes_per_layer', action='store', dest='n_nodes_per_layer', type=int, default=64)
parser.add_argument('--n_layers', action='store', dest='n_layers', type=int, default=2)
parser.add_argument('--mlp_transfer_fn', action='store', dest='mlp_transfer_fn', type=str, default="relu")
parser.add_argument('--train_range', action='store', dest='train_range', type=float, default=1.0)
parser.add_argument('--test_scale', action='store', dest='test_scale', type=float, default=10.0)
...
opts = parser.parse_args()

sess = tf.Session()

net = MLP(opts, opts.n_inputs, repeat_end(opts.n_nodes_per_layer, opts.n_layers, opts.n_inputs), name="net")

inputs  = tf.placeholder(tf.float32, shape=[None, opts.n_inputs], name='inputs')
targets = tf.placeholder(tf.float32, shape=[None, opts.n_inputs], name='outputs')

outputs = net.forward(inputs)

predict_loss = tf.reduce_mean(tf.square(outputs - targets))
l2_loss = tf.zeros([])
for var in tf.trainable_variables(): l2_loss += tf.nn.l2_loss(var)
loss = predict_loss + opts.l2_weight * l2_loss
update  = tf.train.AdamOptimizer(learning_rate=opts.learning_rate).minimize(loss)

tf.global_variables_initializer().run(session=sess)

for epoch in range(opts.n_epochs):
    x_train = np.random.uniform(low=-opts.train_range,
                                high=opts.train_range, size=(opts.batch_size, opts.n_inputs))
    x_test = np.random.uniform(low=-opts.test_scale * opts.train_range,
                               high=opts.test_scale * opts.train_range, size=(opts.batch_size, opts.n_inputs))
    train_loss, _    = sess.run((loss, update), feed_dict={ inputs : x_train, targets : x_train })
    test_loss        = sess.run(loss, feed_dict={ inputs : x_test, targets : x_test })
    if epoch % opts.print_freq == 0:
        print("[%5d] %.12f %.12f" % (epoch, train_loss, test_loss))
```
Note that we train on scalars in the range (-`opts.train_range`, `opts.train_range`) but test on
scalars in the range (-`opts.test_scale` * `opts.train_range`, `opts.test_scale` * `opts.train_range`).

Although the MLP fits the training data easily, it does not do so by computing the identity function and so it never fits the test set:
```
> python3 identity.py --train_range 1.0
[    0] 0.301834642887 35.366600036621
[ 5000] 0.000000458354 1.947080612183
[10000] 0.000000025643 1.913878440857
[15000] 0.000000181023 2.132648468018
[20000] 0.000000004028 1.961588859558
[25000] 0.000000001182 2.179308891296
[30000] 0.000000002907 2.000000000000
[35000] 0.000000002512 2.091145992279
[40000] 0.000000000794 1.993439435959
```

Here is a plot of the learned function on the test set:

<center>
<img src="/assets/images/identity/test_train_range=1.png" style="width:600px;">
</center>

The problem is that the MLP lacks the inductive bias that it should
prefer parameters that behave like short computer programs (the
identity function being one of the shortest possible computer
programs). Even if we set the parameter `opts.train_range` ten times
or a hundred times higher, the network will still make errors on
numbers larger than what it saw during training:

```
> python3 identity.py --train_range 10.0
[    0] 34.92973327636 3487.688964843
[ 5000] 0.000028685839 3.255097150803
[10000] 0.000000603474 3.124519586563
[15000] 0.000000129645 2.434590339661
[20000] 0.000000021062 3.108342885971
[25000] 0.000000012790 2.926368236542
[30000] 0.000001183950 3.146960258484
[35000] 0.000000183261 2.830707550049
[40000] 0.000000003755 3.080693483353

> python3 identity.py --train_range 100.0
[    0] 3358.638427734 326403.0625000
[ 5000] 0.000380217127 5.951098442078
[10000] 0.000002550267 4.455306529999
[15000] 0.000040882987 2.499538421631
[20000] 0.000006450267 3.010317325592
[25000] 0.000001683057 2.145411491394
[30000] 0.000001559869 2.693157434464
[35000] 0.000203355157 2.469587802887
[40000] 0.000000052537 2.609251976013
```

It simply never learns the identity function.

MLPs are the simplest neural networks and are the most flagrant
violators of Occam's razor, but some neural network architectures do
abide by it to some extent. For example, some architectures involve
controllers that interact with diffentiable analogues of conventional
data structures such as arrays, stacks, and queues. However, these
fancier architectures have not yet been successful in practice, and I
suspect it is partly because they are no longer smooth in the way that
heuristic local search (e.g. SGD) relies on. Training these
architectures successfully might require hybrid search strategies that
complement SGD by interleaving global search over some kind of
discrete variables.

There is also a middle ground in neural network architecture design
that is arguably a sweet-spot: hard-coding a (specific) short program
template into the architecture itself. For example, convolutional
neural networks build in a for-loop that shares weights aggressively
over regions of a grid. Similarly, graph neural networks essentially
build in a nested for-loop that iterates over nodes and their edges,
also sharing weights aggressively. These program templates can make
the models much more sample efficient, but still the network does not
employ Occam's razor for regularities that were not built into the
architecture itself.

Lastly, there are also ways of adding additional parsimony bias to a
fixed architecture, such as by regularizing the magnitude of the
weights. However, the effects of such regularization are subtle and do
not correspond to what we really want, which is to prefer parameters
that behave like short computer programs. Indeed, l2 regularization
does not help learn the identity function in the example above.
