---
title: Knuth's Optimization - Interval DP in O(n^2)
description: For interval DPs of the form dp[l][r] = min over k of dp[l][k] + dp[k+1][r] + w(l, r), the optimal split is sandwiched between two neighbouring optima, collapsing the cubic DP to quadratic.
date: 2024-04-24
draft: false
slug: /cp/knuths-optimization
tags:
  - Competitive Programming
  - Dynamic Programming
  - Algorithm Optimization
---

The interval DP for optimal binary search trees, [matrix chain multiplication](/citadel/algorithms/MatrixChainMultiplication), and "merge $n$ ordered piles at minimum cost" all have the same shape: $dp[l][r] = \min_k dp[l][k] + dp[k{+}1][r] + w(l, r)$. That is $O(n^3)$. Knuth's optimisation drops it to $O(n^2)$ by proving the best split point moves monotonically, so you only ever scan a shrinking window of candidates.

## The problem

Compute

$$dp[l][r] = \min_{l \le k < r} \Big( dp[l][k] + dp[k+1][r] \Big) + w(l, r),$$

with $dp[l][l] = 0$ (or a given base), where $w(l, r)$ is a cost depending only on the endpoints (typically a range sum via prefix sums).

Example: "merge stones" — $n$ piles in a row, merging two **adjacent** piles costs the sum of their sizes, repeat until one pile; minimise total cost. Here $w(l, r) = \text{sum of sizes } l..r$.

## The idea

Let $opt[l][r]$ be an argmin $k$. Knuth's optimisation applies when

$$opt[l][r-1] \;\le\; opt[l][r] \;\le\; opt[l+1][r].$$

That is: adding an element on the right cannot move the optimal split left; adding one on the left cannot move it right. So when you compute $dp[l][r]$, the candidate $k$ only needs to range over

$$\big[\, opt[l][r-1],\; opt[l+1][r] \,\big].$$

Sum the width of that window over all $(l, r)$ with the same length $r - l$: it telescopes to $O(n)$ per length, hence $O(n^2)$ total.

**Sufficient conditions** (Yao's theorem): $w$ satisfies the quadrangle inequality $w(a,c) + w(b,d) \le w(a,d) + w(b,c)$ for $a \le b \le c \le d$, **and** $w$ is monotone on intervals ($w(b, c) \le w(a, d)$ for $[b,c] \subseteq [a,d]$). A range-sum $w$ of non-negative values satisfies both.

## How it works

Process intervals by increasing length. For length $1$, $dp[l][l] = 0$ and $opt[l][l] = l$. For a longer interval $[l, r]$, scan $k$ from $opt[l][r-1]$ to $opt[l+1][r]$ only:

$$dp[l][r] = \min_{k} \big( dp[l][k] + dp[k+1][r] \big) + w(l, r),$$

recording the achieving $k$ as $opt[l][r]$. Both $opt[l][r-1]$ and $opt[l+1][r]$ are already computed (shorter intervals), so the bracket is always available.

## Algorithm

```python
def knuth(n: int, w) -> int:
    INF = float("inf")
    dp = [[0] * n for _ in range(n)]
    opt = [[0] * n for _ in range(n)]
    for i in range(n):
        opt[i][i] = i
    for length in range(1, n):                     # r - l
        for l in range(0, n - length):
            r = l + length
            dp[l][r] = INF
            lo = opt[l][r - 1]
            hi = min(opt[l + 1][r], r - 1)         # k must be < r
            for k in range(lo, hi + 1):
                cand = dp[l][k] + dp[k + 1][r] + w(l, r)
                if cand < dp[l][r]:
                    dp[l][r] = cand
                    opt[l][r] = k
    return dp[0][n - 1]
```

With `w(l, r) = prefix[r+1] - prefix[l]` this is the merge-stones / optimal-BST solver.

## Complexity

- **Time:** $O(n^2)$ — for each interval length, $\sum_l (opt[l+1][r] - opt[l][r-1] + 1)$ telescopes to $O(n)$.
- **Space:** $O(n^2)$ for `dp` and `opt`.

Versus $O(n^3)$ for the naive full scan: at $n = 2000$, $1.6 \times 10^{10}$ becomes $4 \times 10^6$.

## Relationship to divide-and-conquer DP

Both exploit monotone optima. [Divide and conquer DP](/citadel/cp/divide-and-conquer-dp) handles the *layered* form $dp_i[j] = \min_k dp_{i-1}[k] + C(k, j)$ with one monotone $opt$ per row, in $O(n \log n)$ per row. Knuth handles the *interval* form with a 2-D $opt$ sandwiched by both neighbours, in $O(n^2)$ total. If your DP is interval-shaped, reach for Knuth; if it is "split into $k$ parts", reach for divide-and-conquer.

## Common pitfalls

- **Iteration order.** You must fill by increasing interval length so $opt[l][r-1]$ and $opt[l+1][r]$ exist. A plain `for l: for r:` double loop breaks this.
- **The bracket endpoints.** $k$ ranges over $[opt[l][r-1],\ opt[l+1][r]]$ *inclusive*. Using $opt[l][r]$ before it is computed, or the wrong neighbours, silently gives a wrong (too-large) answer.
- **$w$ not satisfying the conditions.** If the quadrangle inequality fails, $opt$ is not monotone and the pruned scan misses the real optimum. Verify with a brute check on small $n$.
- **Non-adjacent merges.** Knuth is for splitting a *contiguous* interval at one point. "Merge any two" (like [Huffman](/citadel/algorithms/HuffmanCoding)) is a different problem — a greedy heap, not this DP.
- **Base case.** $dp[l][l]$ is often $0$, but for optimal BST it includes the leaf's own frequency — set it to match your recurrence.

## The keystone

For $dp[l][r] = \min_k dp[l][k] + dp[k{+}1][r] + w(l, r)$ with a quadrangle-inequality $w$, the optimal split satisfies $opt[l][r-1] \le opt[l][r] \le opt[l+1][r]$, so scanning $k$ only within that bracket — filling intervals shortest-first — turns the cubic interval DP into a quadratic one.
