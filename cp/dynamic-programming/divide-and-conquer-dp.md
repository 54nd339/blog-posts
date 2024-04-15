---
title: Divide and Conquer DP - Exploiting Monotone Optima
description: When the best split point of a layered DP is monotone in the state, a divide-and-conquer recursion computes a whole row in O(n log n) instead of O(n squared), dropping a k-partition DP from O(k n squared) to O(k n log n).
date: 2024-04-15
draft: false
slug: /cp/divide-and-conquer-dp
tags:
  - Competitive Programming
  - Dynamic Programming
  - Divide and Conquer
---

A common DP shape is "split the first $j$ things into $i$ groups, minimising total cost": $dp_i[j] = \min_{k < j} \big(dp_{i-1}[k] + C(k, j)\big)$. Computing one row naively is $O(n^2)$, so $i$ rows cost $O(k n^2)$. If the optimal $k$ for each $j$ is **monotone** — non-decreasing in $j$ — a divide-and-conquer recursion computes each row in $O(n \log n)$.

## The problem

Given a layered recurrence

$$dp_i[j] = \min_{0 \le k < j} \Big( dp_{i-1}[k] + C(k, j) \Big),$$

where $C$ is a cost you can evaluate in $O(1)$ (often a range cost from prefix sums), compute $dp_k[n]$.

Let $opt_i[j]$ be an argmin $k$ that achieves $dp_i[j]$. The method applies when

$$opt_i[j] \le opt_i[j+1] \quad \text{for all } j.$$

This holds whenever $C$ satisfies the **quadrangle inequality** $C(a, c) + C(b, d) \le C(a, d) + C(b, c)$ for $a \le b \le c \le d$ — true for many natural costs (sum of pairwise distances in a segment, $-(\text{prefix})^2$, entropy-like costs).

## The idea

To fill row $i$, define `solve(jl, jr, kl, kr)`: compute $dp_i[j]$ for $j \in [jl, jr]$, knowing the optimal $k$ lies in $[kl, kr]$.

1. Take the midpoint $jm = (jl + jr)/2$.
2. Scan $k$ from $kl$ to $\min(jm - 1, kr)$, find the best — record $dp_i[jm]$ and its argmin $km$.
3. Recurse on the left half with $k \in [kl, km]$ and the right half with $k \in [km, kr]$.

Because the $k$-ranges of the two halves overlap only at $km$, the total work across one recursion level is $O(n)$, and there are $O(\log n)$ levels — $O(n \log n)$ per row.

## How it works

Row of size $8$, and suppose the true $opt$ array is $[0, 0, 1, 1, 2, 3, 3, 4]$ (monotone). `solve(0, 7, 0, 7)`: $jm = 3$, scan $k \in [0, 3]$ → best at $k = 1$, so $dp[3]$ set, $km = 1$. Recurse `solve(0, 2, 0, 1)` and `solve(4, 7, 1, 7)`. The left recursion only ever scans $k \in \{0, 1\}$; the right, starting from $k = 1$, never rescans $k = 0$. Every $(j, k)$ pair examined lies under the staircase of the monotone $opt$, so the examined pairs total $O(n \log n)$.

## Algorithm

```python
def dc_dp_row(dp_prev, cost, n):
    dp_cur = [float("inf")] * (n + 1)

    def solve(jl, jr, kl, kr):
        if jl > jr:
            return
        jm = (jl + jr) // 2
        best, arg = float("inf"), kl
        for k in range(kl, min(jm - 1, kr) + 1):    # k <= jm - 1 (no empty group)
            cand = dp_prev[k] + cost(k, jm)
            if cand < best:
                best, arg = cand, k
        dp_cur[jm] = best
        solve(jl, jm - 1, kl, arg)
        solve(jm + 1, jr, arg, kr)

    solve(1, n, 0, n)
    return dp_cur

def solve_all(n, k_layers, cost):
    dp = [0] + [float("inf")] * n                   # dp_0[0] = 0
    for _ in range(k_layers):
        dp = dc_dp_row(dp, cost, n)
    return dp[n]
```

## Complexity

- **Time:** $O(k\, n \log n)$ — one $O(n \log n)$ pass per layer.
- **Space:** $O(n)$ for two rows (plus recursion depth $O(\log n)$).

Contrast: the plain $\min_k$ loop is $O(k n^2)$. For $n = 10^5$, $k = 100$ that is $10^{12}$ vs $\sim 1.7 \times 10^8$.

## Checking the monotonicity condition

- If the problem is "partition into $k$ contiguous groups minimising sum of per-group costs" and the per-group cost has the quadrangle inequality, the condition holds — proceed.
- If unsure, **verify empirically**: brute-force $opt_i[j]$ for small $n$ and check it is non-decreasing in $j$ for each $i$. A single violation means the method is unsafe.
- Some costs need a sign flip (maximisation) or a reflection ($j \to n - j$) to become monotone.

## Common pitfalls

- **Wrong $k$-range in the recursion.** Left half gets $[kl, km]$, right half $[km, kr]$ — both *include* $km$. Narrowing them further can miss the true optimum.
- **`k < jm` boundary.** The inner loop must stop at $k = jm - 1$ (a group cannot be empty on the right); `min(jm, kr) + 1` as the `range` end but with the `k < jm` intent — off-by-one here is the classic bug.
- **Cost not $O(1)$.** If `cost(k, j)` is $O(\log n)$ (e.g. a data-structure query) the row is $O(n \log^2 n)$ — still fine, but budget for it.
- **Applying it without the monotone property.** Produces a wrong answer silently. Always justify or test the condition.
- **Layer count vs "exactly $k$" / "at most $k$".** If groups may be fewer than $k$, add a "take dp from the previous layer" option or a zero-cost empty group.

## The keystone

When the optimal split point of $dp_i[j] = \min_k dp_{i-1}[k] + C(k, j)$ is non-decreasing in $j$ (which the quadrangle inequality guarantees), `solve(jl, jr, kl, kr)` computes the midpoint, then recurses with the $k$-range clamped by the midpoint's argmin — $O(n \log n)$ per layer. If the recurrence is instead the interval form $dp[l][r]$, the same monotonicity gives [Knuth's optimisation](/citadel/cp/knuths-optimization) and $O(n^2)$.
