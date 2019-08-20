---
layout: post
title:  "QuickSpec and the quest for good lemmas"
date:   2018-07-21 00:00:01
---
Given a fixed set of proofs, it is natural to consider a lemma to be good to the extent it enables compression of the proofs.
But how might we know if a lemma is good before we know how it will be used?
Although I do not expect there to ever be a definitive answer to this question, a [recent paper](https://www.cse.chalmers.se/~nicsma/papers/quickspec2.pdf) out of Chalmers University of Technology suggests a simple and perhaps even useful starting point: a lemma is good if it is minimal and non-redundant. The authors have also released a functioning Haskell library called [QuickSpec](https://github.com/nick8325/quickspec) that finds "good" lemmas about a set of user-provided functions. In this post, we will give a brief overview of their approach and look at a few examples.

As mentioned above, the basic premise of QuickSpec is that a lemma is good if it is minimal and non-redundant. Here *minimal* means minimal in terms of an ordering on lemma statements, and *non-redundant* means it does not follow from smaller lemmas based on first-order equational reasoning alone. QuickSpec looks only for equational lemmas, and (ignoring many crucial optimizations) it proceeds by enumerating all possible terms (which may contain free variables) in increasing order. For every term, it normalizes it with respect to the confluent rewriting system induced by the existing set of equational lemmas. If its normal form has already been visited, it is considered redundant and is discarded. Otherwise, it is evaluated on random inputs and compared to previously visited terms. If it is found to be disequal to all of them, it is simply added to the set of visited terms. On the other hand, if it "seems" to be equal to a visited term, the conjecture that the two terms are indeed equal is added to the set of minimal non-redundant (though unproven) lemmas.

For example, we can query QuickSpec on simple boolean functions as follows:

```haskell
data Boolean = T | F deriving (Eq, Show, Ord)

instance Arbitrary Boolean where
  arbitrary = do
    b <- (arbitrary :: Gen Bool)
    if b then return T else return F

andb T T = T
andb _  _  = F

orb F F  = F
orb _  _   = T

negb F    = T
negb T    = F

nandb b1 b2 = negb (andb b1 b2)

main = quickSpec [

  con "F" (F :: Boolean),
  con "T" (T :: Boolean),
  con "and" (andb :: Boolean -> Boolean -> Boolean),
  con "or" (orb :: Boolean -> Boolean -> Boolean),
  con "nand" (nandb :: Boolean -> Boolean -> Boolean),
  con "neg" (negb :: Boolean -> Boolean),

  monoType (Proxy :: Proxy Boolean)
  ]
```
and in under four seconds we get the following output:
```
== Functions ==
   F :: Boolean
   T :: Boolean
 and :: Boolean -> Boolean -> Boolean
  or :: Boolean -> Boolean -> Boolean
nand :: Boolean -> Boolean -> Boolean
 neg :: Boolean -> Boolean

== Laws ==
  1. neg F = T
  2. neg T = F
  3. and x y = and y x
  4. and x x = x
  5. nand x y = nand y x
  6. nand x x = neg x
  7. or x y = or y x
  8. or x x = x
  9. and x F = F
 10. and x T = x
 11. nand x F = T
 12. nand x T = neg x
 13. neg (neg x) = x
 14. or x F = x
 15. or x T = T
 16. and x (neg x) = F
 17. nand x (neg x) = T
 18. neg (and x y) = nand x y
 19. or x (neg y) = nand y (neg x)
 20. and x (and y z) = and y (and x z)
 21. and x (nand x y) = and x (neg y)
 22. and (nand x y) (nand x z) = nand x (or y z)
```

As a second example, if we run QuickSpec with addition, subtraction and multiplication on integers, we get the following output in twenty six seconds:
```
== Functions ==
zero :: Z
 one :: Z
 add :: Z -> Z -> Z
 neg :: Z -> Z
 sub :: Z -> Z -> Z
 mul :: Z -> Z -> Z

== Laws ==
  1. neg zero = zero
  2. add x y = add y x
  3. mul x y = mul y x
  4. sub x x = zero
  5. add x zero = x
  6. mul x one = x
  7. mul x zero = zero
  8. neg (neg x) = x
  9. sub x zero = x
 10. sub zero x = neg x
 11. add x (neg y) = sub x y
 12. mul x (neg y) = mul y (neg x)
 13. neg (mul x y) = mul x (neg y)
 14. neg (sub x y) = sub y x
 15. add x (add y z) = add y (add x z)
 16. mul x (add y y) = mul y (add x x)
 17. mul x (mul y z) = mul y (mul x z)
 18. mul x (add y one) = add x (mul x y)
 19. add (mul x y) (mul x z) = mul x (add y z)
 20. mul x (add y (add y y)) = mul y (add x (add x x))
 21. sub (mul x x) (mul y y) = mul (add x y) (sub x y)
```

In both of the preceeding examples, I think readers will agree that these are indeed excellent lemmas.

I think QuickSpec is currently the gold standard for lemma generation, from both a practical and a philosophical perspective.
Nonetheless it still has major weaknesses, most notably that it requires brute-force enumeration and so is unlikely to scale.
A future post will consider the possibility of using machine learning to help speed up the search for "good" lemmas.
