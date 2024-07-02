---
title: Stars and Bars - Distributing Identical Items
description: The bijection between putting n identical balls into k boxes and choosing bar positions among n + k - 1 slots, giving binom(n + k - 1, k - 1), plus the fixes for lower bounds, upper bounds, and positivity.
date: 2024-07-02
draft: false
slug: /cp/stars-and-bars
tags:
  - Competitive Programming
  - Combinatorics
  - Counting
---

"How many ways to write $n$ as an ordered sum of $k$ non-negative integers?" — equivalently, distribute $n$ identical balls into $k$ labelled boxes. The answer is $\binom{n + k - 1}{k - 1}$, from a one-line bijection, and small modifications cover bounds on the box contents.

## The problem

Count integer solutions to

$$x_1 + x_2 + \dots + x_k = n, \qquad x_i \ge 0.$$

Variants: $x_i \ge 1$ (positive), $x_i \ge \ell_i$ (lower bounds), $x_i \le u_i$ (upper bounds), "at most $n$" instead of "exactly $n$".

Example: $x_1 + x_2 + x_3 = 5$, $x_i \ge 0$ → $\binom{5 + 2}{2} = \binom{7}{2} = 21$ solutions.

## The idea

Line up $n$ stars. To split them into $k$ ordered groups, insert $k - 1$ **bars** into the gaps — but bars may share a gap or sit at the ends (a group can be empty). Encode a distribution as a sequence of $n$ stars and $k - 1$ bars: $n + k - 1$ symbols, choose which $k - 1$ are bars:

$$\binom{n + k - 1}{k - 1} = \binom{n + k - 1}{n}.$$

## Variants

- **Positive parts** $x_i \ge 1$: give each box one ball first, then distribute the remaining $n - k$ freely: $\binom{n - 1}{k - 1}$ (needs $n \ge k$).
- **Lower bounds** $x_i \ge \ell_i$: substitute $y_i = x_i - \ell_i \ge 0$; solve $\sum y_i = n - \sum \ell_i$: $\binom{n - \sum \ell_i + k - 1}{k - 1}$.
- **"At most $n$"** ($\sum x_i \le n$): add a slack variable $x_{k+1} \ge 0$; count $\sum_{i=1}^{k+1} x_i = n$: $\binom{n + k}{k}$.
- **Upper bounds** $x_i \le u_i$: no clean single binomial — use [inclusion-exclusion](/citadel/cp/inclusion-exclusion) over "which boxes overflow". Subtract $\ge u_i + 1$ from a subset $S$ of boxes:

$$\sum_{S \subseteq \{1..k\}} (-1)^{|S|} \binom{n - \sum_{i \in S}(u_i + 1) + k - 1}{k - 1},$$

dropping terms where the top argument goes negative. Feasible for $k \lesssim 20$.

## Algorithm

```python
MOD = 10 ** 9 + 7

def stars_and_bars(n: int, k: int, C) -> int:
    if n < 0 or k <= 0:
        return 0
    return C(n + k - 1, k - 1)                 # non-negative parts

def with_lower_bounds(n: int, lows: list[int], C) -> int:
    rem = n - sum(lows)
    k = len(lows)
    return C(rem + k - 1, k - 1) if rem >= 0 else 0

def with_upper_bounds(n: int, ups: list[int], C) -> int:
    k = len(ups)
    total = 0
    for S in range(1 << k):
        take = sum(ups[i] + 1 for i in range(k) if S >> i & 1)
        top = n - take + k - 1
        if top >= k - 1:
            sign = -1 if bin(S).count("1") & 1 else 1
            total = (total + sign * C(top, k - 1)) % MOD
    return total % MOD
```

`C(a, b)` is a [binomial coefficient](/citadel/cp/binomial-coefficients) (mod a prime, from factorial tables).

## Where it shows up

- **Compositions of $n$** into $k$ parts, or into *any* number of parts ($2^{n-1}$ total).
- **Monomials of degree $n$ in $k$ variables** — same count.
- **Non-decreasing sequences** $1 \le a_1 \le \dots \le a_k \le m$: substitute $b_i = a_i + (i - 1)$ to make them strictly increasing, then it is $\binom{m + k - 1}{k}$ — "combinations with repetition".
- **Lattice paths** with only right/up steps — a path is a choice of which steps go up.
- **Distributing a budget / resource** with per-item minimums and maximums (the inclusion-exclusion variant).

## Common pitfalls

- **Boxes vs balls in the binomial.** It is $\binom{n + k - 1}{k - 1}$ (equivalently $\binom{n + k - 1}{n}$). Swapping $n$ and $k$ in the "choose" slot is the classic error — double-check with a tiny case.
- **Positive parts feasibility.** $x_i \ge 1$ needs $n \ge k$; otherwise $0$.
- **Upper-bound inclusion-exclusion sign and range.** Terms with a negative top argument contribute $0$ — skip them, do not treat $\binom{\text{negative}}{\cdot}$ as anything but $0$.
- **"At most" vs "exactly".** Add a slack box; do not sum $\binom{i + k - 1}{k - 1}$ over $i \le n$ (correct but $O(n)$ when $O(1)$ is available).
- **Identical vs distinct items.** Stars and bars is for *identical* items. Distinct items into boxes is $k^n$ (or a Stirling-number problem if boxes are unlabelled).

## The keystone

Distributing $n$ identical items into $k$ ordered boxes is choosing $k - 1$ bar positions among $n + k - 1$ symbols: $\binom{n + k - 1}{k - 1}$. Lower bounds shift the target, "at most" adds a slack box, and upper bounds need [inclusion-exclusion](/citadel/cp/inclusion-exclusion) over which boxes overflow.
