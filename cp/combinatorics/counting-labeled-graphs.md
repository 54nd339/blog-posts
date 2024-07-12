---
title: Counting Labeled Graphs - Connected and Otherwise
description: The 2 to the (n choose 2) total labeled graphs, the recurrence that peels off the component containing vertex 1 to count connected ones, and the same trick for labeled trees, forests, and DAGs.
date: 2024-07-12
draft: false
slug: /cp/counting-labeled-graphs
tags:
  - Competitive Programming
  - Combinatorics
  - Graph Theory
---

How many graphs are there on $n$ **labeled** vertices? All of them: $2^{\binom{n}{2}}$, since each possible edge is independently present or not. The interesting count is **connected** labeled graphs — there is no closed form, but a clean $O(n^2)$ recurrence, obtained by fixing the component that contains vertex $1$.

## The problem

Count, on vertex set $\{1, \dots, n\}$:

- all graphs: $G_n = 2^{\binom{n}{2}}$,
- **connected** graphs: $C_n$,
- graphs with exactly $k$ components, labeled trees, labeled forests, DAGs — all via the same "peel vertex 1's component" idea.

Values: $C_1 = 1$, $C_2 = 1$, $C_3 = 4$, $C_4 = 38$, $C_5 = 728$.

## The idea

Every graph on $n$ vertices has a unique component containing vertex $1$. Sum over its size $k$: choose the other $k - 1$ vertices of that component ($\binom{n-1}{k-1}$ ways), make that component connected ($C_k$ ways), and put an arbitrary graph on the remaining $n - k$ vertices ($2^{\binom{n-k}{2}}$ ways):

$$2^{\binom{n}{2}} = \sum_{k=1}^{n} \binom{n-1}{k-1}\, C_k \, 2^{\binom{n-k}{2}}.$$

Solve for $C_n$ (the $k = n$ term):

$$C_n = 2^{\binom{n}{2}} - \sum_{k=1}^{n-1} \binom{n-1}{k-1}\, C_k\, 2^{\binom{n-k}{2}}.$$

## How it works

$C_1 = 1$.
$C_2 = 2^{1} - \binom{1}{0} C_1 \cdot 2^{0} = 2 - 1 = 1$.
$C_3 = 2^{3} - \big[\binom{2}{0}C_1 2^{1} + \binom{2}{1}C_2 2^{0}\big] = 8 - (1\cdot1\cdot2 + 2\cdot1\cdot1) = 8 - 4 = 4$.
$C_4 = 2^{6} - \big[\binom{3}{0}C_1 2^{3} + \binom{3}{1}C_2 2^{1} + \binom{3}{2}C_3 2^{0}\big] = 64 - (8 + 6 + 12) = 38$.

## Algorithm

```python
from math import comb

def connected_labeled_graphs(n: int, mod: int | None = None) -> list[int]:
    def binom2(x):
        return x * (x - 1) // 2

    pow2 = [1] * (binom2(n) + 1)
    for i in range(1, len(pow2)):
        pow2[i] = pow2[i - 1] * 2
        if mod:
            pow2[i] %= mod

    C = [0] * (n + 1)
    for m in range(1, n + 1):
        total = pow2[binom2(m)]
        for k in range(1, m):
            term = comb(m - 1, k - 1) * C[k] % (mod or 1 << 62) * pow2[binom2(m - k)]
            total -= term
        C[m] = total % mod if mod else total
    return C                                    # C[m] = # connected labeled graphs on m vertices
```

(For a modulus, precompute [binomial coefficients](/citadel/cp/binomial-coefficients) with factorial tables rather than `math.comb`.)

## The same trick elsewhere

- **Graphs with exactly $k$ components:** an [exponential generating function](/citadel/cp/operations-on-polynomials) statement — $\text{(all)} = \exp(\text{connected})$ in EGF terms; the $k$-component count is $[x^n/n!]\, \frac{(\text{connected EGF})^k}{k!}$, or a convolution DP.
- **Labeled trees:** Cayley's formula $n^{n-2}$ (see also the [Prüfer code](/citadel/cp/prufer-code)). Labeled *forests* with $k$ trees on $n$ vertices: $\binom{n-1}{k-1} n^{n-k}$ (generalised Cayley).
- **Labeled connected graphs with $n$ vertices and $n$ edges** (exactly one cycle): $\binom{n}{3} + 3\binom{n}{4}\cdot\dots$ — or via the same peeling.
- **Labeled DAGs:** $a_n = \sum_{k=1}^{n} (-1)^{k-1} \binom{n}{k} 2^{k(n-k)} a_{n-k}$, peeling off a non-empty set of sources.
- **Eulerian / bipartite / triangle-free labeled graphs** — each has its own inclusion-exclusion or recurrence in the same spirit.

## Complexity

- **Connected-graph recurrence:** $O(n^2)$ time (an $O(n)$ convolution for each of $n$ values), $O(n^2)$ for the powers of two, or precompute $2^{\binom{m}{2}}$ incrementally.
- **DAG recurrence:** $O(n^2)$ similarly.

## Common pitfalls

- **"Labeled" vs "unlabeled".** These formulas count graphs on *distinguishable* vertices. Unlabeled (isomorphism classes) counts need [Burnside](/citadel/cp/burnside-polya) over the symmetric group and are much harder — no simple recurrence.
- **The $k = n$ term.** It is exactly $C_n$ (with $\binom{n-1}{n-1} = 1$, $2^{\binom{0}{2}} = 1$); that is why you can isolate it. Don't include it in the subtracted sum.
- **$\binom{0}{2} = \binom{1}{2} = 0$.** Powers of two of these are $2^0 = 1$; make sure `binom2` returns $0$, not a negative.
- **Overflow without a modulus.** $2^{\binom{n}{2}}$ is astronomically large for $n \ge 10$; Python is fine, but a modulus is almost always intended.
- **Sign in the DAG recurrence.** It is an inclusion-exclusion (peeling a non-empty source set), so the alternating sign is essential.

## The keystone

Total labeled graphs on $n$ vertices is $2^{\binom{n}{2}}$; connected ones follow from fixing vertex $1$'s component: $\sum_k \binom{n-1}{k-1} C_k\, 2^{\binom{n-k}{2}} = 2^{\binom{n}{2}}$, solved for $C_n$ in $O(n^2)$. The same "peel the piece containing vertex 1 (or a source set)" idea counts forests, one-cycle graphs, and DAGs.
