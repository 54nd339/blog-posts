---
title: Enumerating Submasks of a Bitmask - The O(3^n) Loop
description: The two-line idiom that visits every subset of a bitmask in decreasing order, why iterating all submasks of all masks totals 3 to the n rather than 4 to the n, and the traps around the empty submask.
date: 2024-02-09
draft: false
slug: /cp/enumerating-submasks
tags:
  - Competitive Programming
  - Algebra
  - Bit Manipulation
---

Bitmask DP constantly needs "for each subset $s$ of a set $m$, do something". The naive way — loop $s$ from $0$ to $m$ and skip the ones that are not subsets — is $O(2^n)$ per mask. There is a two-line trick that visits *only* the submasks, and summed over all masks it costs $3^n$, not $4^n$.

## The problem

Given a bitmask $m$, iterate over every $s$ with $s \mathbin{\&} m = s$ (every subset of the set bits of $m$), each exactly once.

Example: $m = 1011_2$. Its submasks are $1011, 1010, 1001, 1000, 0011, 0010, 0001, 0000$ — eight of them, $2^{\text{popcount}(m)}$.

## The idea

To go from one submask $s$ to the next smaller one, you want the largest submask of $m$ that is strictly less than $s$. The idiom:

$$s \leftarrow (s - 1) \mathbin{\&} m.$$

Subtracting $1$ borrows through the low zero-run of $s$, turning the lowest set bit off and all bits below it on; AND-ing with $m$ restricts those turned-on bits to positions that are actually in $m$. The result is the next submask down. Start at $s = m$ and stop after processing $s = 0$.

## How it works

$m = 1011_2$, starting at $s = 1011$:

| $s$ | $s - 1$ | $(s-1)\ \&\ m$ |
| --- | --- | --- |
| 1011 | 1010 | 1010 |
| 1010 | 1001 | 1001 |
| 1001 | 1000 | 1000 |
| 1000 | 0111 | 0011 |
| 0011 | 0010 | 0010 |
| 0010 | 0001 | 0001 |
| 0001 | 0000 | 0000 |
| 0000 | — | stop |

Eight submasks, strictly decreasing.

## Algorithm

```python
def submasks(m: int):
    s = m
    while True:
        yield s
        if s == 0:
            break
        s = (s - 1) & m
```

The `if s == 0: break` *after* yielding is the point: `0` is a valid submask and must be visited, but `(0 - 1) & m == m` would restart the loop forever.

Typical use inside a DP over $2^n$ masks:

```python
for m in range(1 << n):
    s = m
    while True:
        # combine dp[s] with dp[m ^ s], etc.
        if s == 0:
            break
        s = (s - 1) & m
```

## Why the total is 3^n

Iterating every submask of every mask touches each pair $(m, s)$ with $s \subseteq m$. For each of the $n$ bit positions there are three independent possibilities: the bit is in neither, in $m$ only, or in both. That is $3^n$ pairs, so the nested loop is $\Theta(3^n)$ — a big saving over the $4^n$ of "all $s$ for all $m$" and the $2^n \cdot 2^n$ of the skip-if-not-subset approach.

For reference: $3^{20} \approx 3.5 \times 10^9$ (borderline), $3^{18} \approx 3.9 \times 10^8$ (fine). Sum-over-subsets ([SOS DP](/citadel/cp/dynamic-programming)) does the same aggregation in $O(2^n \cdot n)$ when you only need subset *sums*, and is preferred when it applies.

## Common pitfalls

- **Infinite loop on $s = 0$.** `(0 - 1) & m == m`, so without the post-yield break the loop cycles. Structure it as "process, test zero, step".
- **Excluding the empty or full submask.** Both $s = m$ and $s = 0$ are submasks. If the problem wants *proper* non-empty subsets, start from `(m - 1) & m` and stop before `0`.
- **Thinking it is $O(2^n)$ overall.** Per mask it is $2^{\text{popcount}(m)}$; the *sum* over all masks is $3^n$. Budget accordingly — $n = 20$ is the practical ceiling.
- **Using it when SOS DP fits.** If you are computing, for every mask, an aggregate over all its submasks (a sum, a max), sum-over-subsets DP is $O(2^n n)$ and beats $3^n$ for $n \gtrsim 15$.

## The keystone

`s = (s - 1) & m` walks the submasks of `m` in strictly decreasing order; start at `m`, process, break on `0`. Nested inside a loop over all $2^n$ masks it runs in $3^n$ total, because every bit is independently "out, in $m$, or in both" — the standard cost of subset-sum-style bitmask DP.
