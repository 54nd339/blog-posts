---
title: Generating All K-Combinations - Lexicographic and Gray-Code Order
description: Enumerating every k-subset of an n-set — the next-combination increment in lexicographic order, the revolving-door Gray code that changes one element per step, and the bitmask trick for small n.
date: 2024-07-02
draft: false
slug: /cp/generating-k-combinations
tags:
  - Competitive Programming
  - Combinatorics
  - Enumeration
---

Sometimes you actually have to *list* every $k$-element subset of $\{0, 1, \dots, n-1\}$, not just count them. Three ways: lexicographic order with an $O(k)$-amortised "next combination", a Gray-code order that swaps one element per step, and — for $n \le 63$ — a bitmask enumeration.

## The problem

Produce all $\binom{n}{k}$ combinations, each exactly once, in a predictable order, cheaply per step.

Example: $n = 4$, $k = 2$, lexicographic → `01, 02, 03, 12, 13, 23`.

## Lexicographic: next combination

Keep the combination as a sorted list `c[0] < c[1] < ... < c[k-1]`. To advance:

1. Find the rightmost index $i$ with $c[i] < n - k + i$ (i.e. $c[i]$ can still increase without pushing the elements after it past $n - 1$).
2. Increment $c[i]$.
3. Reset $c[i+1], c[i+2], \dots$ to $c[i] + 1, c[i] + 2, \dots$ (the smallest legal tail).
4. If no such $i$ exists, you were at the last combination.

```python
def combinations_lex(n: int, k: int):
    c = list(range(k))
    while True:
        yield c[:]
        i = k - 1
        while i >= 0 and c[i] == n - k + i:
            i -= 1
        if i < 0:
            return
        c[i] += 1
        for j in range(i + 1, k):
            c[j] = c[j - 1] + 1
```

Amortised $O(1)$ per combination (the resets are rare and short on average); worst step $O(k)$.

## Gray code: one element changes per step

A **revolving-door** ordering lists all $k$-subsets so that consecutive subsets differ by removing one element and adding another (symmetric difference of size $2$). Useful when maintaining an incremental quantity over the subset (a determinant, a running sum, a DP value) so each step is $O(1)$ instead of $O(k)$.

A clean recursive construction (Knuth): the $k$-subsets of $\{0, \dots, m-1\}$ in Gray order are the $k$-subsets of $\{0, \dots, m-2\}$ in Gray order, followed by the $(k-1)$-subsets of $\{0, \dots, m-2\}$ in *reverse* Gray order, each with $m-1$ appended. The join across the boundary changes one element.

```python
def combinations_gray(n: int, k: int) -> list[tuple[int, ...]]:
    if k == 0:
        return [()]
    if k == n:
        return [tuple(range(n))]
    if k < 0 or k > n:
        return []
    without = combinations_gray(n - 1, k)
    with_last = [t + (n - 1,) for t in combinations_gray(n - 1, k - 1)][::-1]
    return without + with_last
```

Consecutive entries satisfy `len(set(a) ^ set(b)) == 2`. Diff them to recover which element left and which entered.

## Bitmask enumeration (small n)

For $n \le 63$, iterate integers with exactly $k$ set bits using the **Gosper's hack** successor:

```python
def combinations_bitmask(n: int, k: int):
    if k == 0:
        yield 0
        return
    mask = (1 << k) - 1
    limit = 1 << n
    while mask < limit:
        yield mask
        c = mask & -mask
        r = mask + c
        mask = (((mask ^ r) >> 2) // c) | r
```

`mask` walks through all $n$-bit numbers with popcount $k$ in increasing numeric order — which is **colexicographic** order of the subsets (sort by largest element, then next-largest, …), not the lexicographic order of `combinations_lex`. Each successor is $O(1)$.

## Ranking and unranking (colexicographic)

For a strictly increasing $c_0 < c_1 < \dots < c_{k-1}$, its **colex rank** — its 0-based position among all $k$-subsets in colexicographic order — is

$$\text{rank}(c) = \sum_{i=0}^{k-1} \binom{c_i}{i + 1}.$$

**Unrank** the $r$-th subset by peeling from the top index down: for $i = k-1, \dots, 0$, take $c_i$ = the largest $v$ with $\binom{v}{i+1} \le r$, then $r \mathrel{-}= \binom{c_i}{i+1}$.

```python
from math import comb

def colex_rank(c: list[int]) -> int:
    return sum(comb(c[i], i + 1) for i in range(len(c)))

def colex_unrank(r: int, k: int) -> tuple[int, ...]:
    c = [0] * k
    for i in range(k - 1, -1, -1):
        v = i
        while comb(v + 1, i + 1) <= r:
            v += 1
        c[i] = v
        r -= comb(v, i + 1)
    return tuple(c)
```

This jumps to the $r$-th combination in $O(k \log n)$ without generating the earlier ones — the tool for "the $10^{12}$-th combination" problems. (Lexicographic rank/unrank is analogous with $\binom{n - 1 - c_i}{k - i}$-style terms; colex is the one that matches Gosper's hack order.)

## Common pitfalls

- **Lexicographic termination test.** $c[i] = n - k + i$ is the maximum value position $i$ can hold. Comparing against $n - 1$ or $n$ instead is off by the tail length.
- **Tail reset.** After incrementing $c[i]$, positions $i+1 \dots k-1$ must be the *minimal* legal continuation $c[i]+1, c[i]+2, \dots$ — not left as they were.
- **Gosper's hack with $k = 0$.** `mask & -mask` on `mask = 0` is `0`, causing a division by zero. Special-case $k = 0$.
- **$k > n$.** Zero combinations; guard before starting.
- **Gray-code vs lexicographic when order matters.** If the problem wants combinations in lexicographic order, do not hand it the revolving-door sequence.

## The keystone

Lexicographic "next combination" finds the rightmost element that can grow, bumps it, and resets the tail to the minimum — amortised $O(1)$. When you carry an incremental quantity across subsets, the revolving-door Gray code changes one element per step; and for $n \le 63$, Gosper's hack steps through fixed-popcount bitmasks in $O(1)$.
