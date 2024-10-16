---
title: The Assignment Problem - The Hungarian Algorithm in O(n^3)
description: Assigning n workers to n jobs at minimum total cost with the Hungarian (Kuhn-Munkres) algorithm's potential-and-augmenting-path formulation, plus the min-cost-flow alternative.
date: 2024-10-16
draft: false
slug: /cp/assignment-problem
tags:
  - Competitive Programming
  - Graphs
  - Matching
---

Given an $n \times n$ cost matrix, assign each row (worker) to a distinct column (job) so the total cost is minimum. Brute force is $n!$; the **Hungarian algorithm** does it in $O(n^3)$ by maintaining dual potentials on rows and columns and repeatedly finding a shortest augmenting path in the equality subgraph.

## The problem

Find a permutation $\pi$ minimising $\sum_{i} \text{cost}[i][\pi(i)]$. Variants: rectangular matrices ($n$ workers, $m \ge n$ jobs — pad or handle directly), maximisation (negate), and "assign as many as possible then minimise cost".

Example: $\begin{pmatrix} 9 & 2 & 7 \\ 6 & 4 & 3 \\ 5 & 8 & 1 \end{pmatrix}$ → assign row 0→col 1 (2), row 1→col 2 (3), row 2→... col 0 (5)? total $10$; or 0→1, 1→2, 2→... the optimum is $2 + 3 + 5 = 10$ vs $2 + 6 + 1 = 9$ (0→1, 1→0, 2→2). The Hungarian algorithm finds $9$.

## The idea (potentials + augmenting paths)

Maintain potentials $u_i$ (rows) and $v_j$ (columns) with the invariant $u_i + v_j \le \text{cost}[i][j]$ for all $i, j$. An edge is **tight** when $u_i + v_j = \text{cost}[i][j]$. A perfect matching using only tight edges is optimal (LP duality / complementary slackness).

Add rows one at a time. For the new row, run a Dijkstra-like search over columns in the reduced-cost metric $\text{cost}[i][j] - u_i - v_j$, tracking the minimum "slack" to reach each unmatched column. When you reach a free column, you have a shortest augmenting path; adjust potentials by the found distances (keeping the invariant), and flip the matching along the path. Each row costs $O(n^2)$, so $O(n^3)$ total.

## Algorithm (1-indexed potentials, the classic e-maxx form)

```python
def hungarian(cost):
    n = len(cost)
    m = len(cost[0])
    assert m >= n
    INF = float("inf")
    u = [0] * (n + 1)                            # row potentials
    v = [0] * (m + 1)                            # column potentials
    p = [0] * (m + 1)                            # p[j] = row assigned to column j (0 = none)
    way = [0] * (m + 1)                          # for path reconstruction

    for i in range(1, n + 1):
        p[0] = i
        j0 = 0
        minv = [INF] * (m + 1)
        used = [False] * (m + 1)
        while True:
            used[j0] = True
            i0 = p[j0]
            delta = INF
            j1 = -1
            for j in range(1, m + 1):
                if not used[j]:
                    cur = cost[i0 - 1][j - 1] - u[i0] - v[j]
                    if cur < minv[j]:
                        minv[j] = cur
                        way[j] = j0
                    if minv[j] < delta:
                        delta = minv[j]
                        j1 = j
            for j in range(m + 1):
                if used[j]:
                    u[p[j]] += delta
                    v[j] -= delta
                else:
                    minv[j] -= delta
            j0 = j1
            if p[j0] == 0:
                break
        while j0:                               # augment along the path
            j1 = way[j0]
            p[j0] = p[j1]
            j0 = j1

    assignment = [0] * n                         # assignment[i] = column for row i (0-indexed)
    for j in range(1, m + 1):
        if p[j] != 0:
            assignment[p[j] - 1] = j - 1
    total = sum(cost[i][assignment[i]] for i in range(n))
    return total, assignment
```

`-v[0]` at the end also equals the optimal cost (a nice self-check).

## Alternatives

- **[Min-cost max-flow](/citadel/cp/min-cost-flow):** source → each worker (cap 1, cost 0), worker → job (cap 1, cost = matrix entry), job → sink (cap 1, cost 0). Max flow $n$ at min cost = the assignment. $O(n \cdot E \log V)$ with SSP; simpler if you already have an MCMF template, slightly slower.
- **Auction algorithm:** workers "bid" on jobs, raising prices; $\varepsilon$-scaling gives $O(n^3 \log(nC))$ and parallelises well. Rare in contests.
- **Small $n$ ($\le 20$):** bitmask DP `dp[mask]` = min cost to assign the first `popcount(mask)` workers to the jobs in `mask`. $O(2^n \cdot n)$.

## Applications

- **Minimum-weight perfect matching in a bipartite graph** (the assignment problem *is* this).
- **Minimum total distance to pair up two point sets** (cost = Euclidean distance).
- **Tracking / data association** (assign detections to tracks minimising error).
- **Scheduling** $n$ jobs on $n$ machines with a per-(job, machine) cost.
- Subroutine in **branch-and-bound for TSP** (the assignment relaxation is a lower bound).

## Complexity

- **Time:** $O(n^3)$ (or $O(n^2 m)$ for $n \times m$, $m \ge n$).
- **Space:** $O(n m)$ for the matrix, $O(n + m)$ working arrays.

## Common pitfalls

- **Maximisation.** Negate the matrix (or subtract from a large constant); the algorithm minimises.
- **Rectangular input.** Require $m \ge n$; if $n > m$, transpose. Padding with a constant column also works but wastes an $O(n)$.
- **The dummy index 0.** `p[0]`, `way`, and the `u[0]/v[0]` slots are part of the algorithm's bookkeeping — don't trim them.
- **Integer vs float costs.** Works with either; with floats, the `delta` comparisons need care near ties. Prefer integer costs.
- **Off-by-one from the 1-indexed potentials.** `cost[i0-1][j-1]` — the matrix is 0-indexed, the potentials are 1-indexed. Mixing them is the classic bug.

## The keystone

The Hungarian algorithm keeps row/column potentials with $u_i + v_j \le \text{cost}[i][j]$ and adds rows one at a time, each via a shortest augmenting path in the reduced-cost metric, adjusting potentials to keep the invariant. A perfect matching on the tight ($u_i + v_j = \text{cost}[i][j]$) edges is provably optimal — $O(n^3)$.
