---
title: The Stern-Brocot Tree and Farey Sequences - Enumerating the Rationals
description: The infinite binary search tree that contains every positive rational exactly once in lowest terms, built from mediants - how to navigate it, why the L/R path is a continued fraction, and its link to Farey sequences and best rational approximation.
date: 2024-11-25
draft: false
slug: /cp/stern-brocot-tree
tags:
  - Competitive Programming
  - Miscellaneous
  - Number Theory
---

The Stern-Brocot tree lists every positive rational number, each exactly once, already in lowest terms — no duplicates, no gaps, no `gcd` needed. It is an infinite binary search tree whose structure encodes [continued fractions](/citadel/cp/continued-fractions), the [Euclidean algorithm](/citadel/cp/euclidean-gcd), and best rational approximation all at once.

## Construction: mediants

Start with the two "boundary" fractions $\frac{0}{1}$ and $\frac{1}{0}$ (the latter a formal symbol for $+\infty$). Between any two neighbours $\frac{a}{b}$ and $\frac{c}{d}$, insert their **mediant** $\frac{a + c}{b + d}$. Recurse on the two new gaps forever.

The mediant of two fractions always lies strictly between them, and if $bc - ad = 1$ (adjacent in the construction) then the mediant is automatically in lowest terms and stays adjacent to both parents. Every positive rational appears exactly once as some mediant.

```
                         1/1
              1/2                   2/1
        1/3         2/3       3/2         3/1
     1/4   2/5   3/5   3/4  4/3  5/3   5/2   4/1
```

The tree is a binary search tree: an in-order traversal yields the rationals in increasing order.

## Navigating to a fraction

To find the path from the root to a target $\frac{p}{q}$ (in lowest terms), keep the current left/right bounding fractions and repeatedly compare the target to the mediant:

```python
def sb_path(p, q):
    lp, lq, rp, rq = 0, 1, 1, 0          # left = 0/1, right = 1/0
    path = []
    while True:
        mp, mq = lp + rp, lq + rq        # current node = mediant
        if (mp, mq) == (p, q):
            return path
        if p * mq < mp * q:              # target < mediant  ->  go left
            path.append('L')
            rp, rq = mp, mq
        else:                            # target > mediant  ->  go right
            path.append('R')
            lp, lq = mp, mq

def sb_from_path(path):
    lp, lq, rp, rq = 0, 1, 1, 0
    for ch in path:
        mp, mq = lp + rp, lq + rq
        if ch == 'L':
            rp, rq = mp, mq
        else:
            lp, lq = mp, mq
    return (lp + rp, lq + rq)            # the node the path ends on
```

Compare fractions by cross-multiplication ($p \cdot mq$ vs $mp \cdot q$) to stay in integers.

## The path is a continued fraction

Group the path into runs of equal letters. If $\frac{p}{q} = [a_0; a_1, a_2, \dots, a_m]$ as a continued fraction, then the path is

$$R^{a_0}\, L^{a_1}\, R^{a_2}\, L^{a_3} \cdots$$

with the very last run one shorter ($a_m - 1$). So walking the tree *is* running the Euclidean algorithm on $p$ and $q$: each run length is one quotient. This is why navigation takes $O(a_0 + a_1 + \cdots)$ steps naively, but only $O(m)$ if you jump run-by-run using the quotients — and $m = O(\log \max(p, q))$.

## Best rational approximation

To find the fraction with the smallest denominator in an interval, or the closest fraction to a real $x$ with denominator $\le N$: descend the tree from the root, going left or right according to $x$, and stop as soon as the denominator would exceed $N$. The **convergents** and **semiconvergents** you pass are exactly the best approximations (the same ones the continued-fraction expansion of $x$ produces). Binary-searching a whole run at a time keeps this $O(\log N)$ per step.

## Farey sequences

The **Farey sequence** $F_n$ is all fractions in $[0, 1]$ with denominator $\le n$, in increasing order. Consecutive Farey fractions $\frac{a}{b} < \frac{c}{d}$ satisfy $bc - ad = 1$ — the same adjacency relation as the Stern-Brocot tree — so $F_n$ is a horizontal "slice" of the tree cut off at denominator $n$. Given one pair, the next fraction after $\frac{c}{d}$ is

$$\frac{k \cdot c - a}{k \cdot d - b}, \qquad k = \left\lfloor \frac{n + b}{d} \right\rfloor,$$

which lets you stream $F_n$ in $O(|F_n|)$ time and $O(1)$ memory. $|F_n| = 1 + \sum_{k=1}^{n} \varphi(k) \approx \frac{3 n^2}{\pi^2}$.

## Complexity

- **Navigate to $\frac{p}{q}$:** $O(\log(p + q))$ jumping by runs; $O(p + q)$ step-by-step.
- **Best approximation with denominator $\le N$:** $O(\log N)$.
- **Enumerate $F_n$:** $O(n^2 / \pi^2)$ terms, $O(1)$ extra space.

## Common pitfalls

- **The right boundary $\frac{1}{0}$.** It is a sentinel for infinity, not a real fraction. Never reduce it or feed it to `gcd`; just carry the pair $(1, 0)$.
- **Float comparisons.** Compare the target to the mediant with cross-multiplication. Floating point misroutes near equality and the descent never terminates.
- **Naive descent on a skewed fraction.** $\frac{1}{10^9}$ is a path of $10^9$ `L`s. Always advance a whole run using the Euclidean quotient, not one edge at a time.
- **Off-by-one in the CF correspondence.** The final continued-fraction term contributes a run of length $a_m - 1$, not $a_m$; and $a_0 = 0$ for $\frac{p}{q} < 1$ means the path starts with `L`.
- **Farey adjacency direction.** The recurrence generates the *next* fraction to the right; keep the invariant $bc - ad = 1$ or the stream drifts.

## The keystone

Insert mediants between $\frac{0}{1}$ and $\frac{1}{0}$ forever and you enumerate every positive rational once, in lowest terms, as a binary search tree. The L/R path to a fraction is its continued fraction (its Euclidean quotients as run lengths), a denominator cutoff turns the tree into a Farey sequence, and stopping the descent at a denominator bound yields the best rational approximations.
