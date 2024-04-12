---
title: Dynamic Programming - Recurrences, Tables, and Speed-Ups
description: A map of contest DP — how to find the state and recurrence, the standard table shapes, and the four optimisations (monotone divide-and-conquer, Knuth, convex hull trick, bitmask profile) that turn a too-slow DP into an AC one.
date: 2024-04-12
draft: false
slug: /cp/dynamic-programming
tags:
  - Competitive Programming
  - Dynamic Programming
  - Algorithm Optimization
---

Dynamic programming is: define a state, write the value of that state as a recurrence over smaller states, evaluate every state once. The [1-D](/citadel/dsa/1-d-dynamic-programming) and [2-D](/citadel/dsa/2-d-dynamic-programming) DP guides cover the pattern recognition; this section is about the contest-specific part — the recurrences that are too slow in their natural form, and the four transforms that fix them.

## Finding the DP

1. **State** — the smallest description of a subproblem such that the answer depends only on the state, not the path to it. "Prefix of length $i$", "$i$ items considered, capacity $j$ used", "at node $v$, subtree done", "subset $S$ of tasks assigned".
2. **Transition** — express `dp[state]` from strictly smaller states. This is the whole problem; the code is bookkeeping.
3. **Order** — evaluate states so every dependency is ready. Increasing $i$, increasing subset popcount, post-order on a tree, or memoised recursion.
4. **Answer** — one state, or an aggregate over states.

Cost is (number of states) $\times$ (cost per transition). Contest DP problems usually have an obvious $O(n^2)$ or $O(n \cdot 2^n)$ DP that is a factor of $n$ or $\log n$ too slow — hence the optimisations below.

## The standard shapes

- **Linear** `dp[i]` from `dp[i-1..i-k]` — Fibonacci-shaped, [knapsack](/citadel/algorithms/01Kanpsack), [LIS](/citadel/dsa/longest-increasing-subsequence).
- **Two-pointer grid** `dp[i][j]` over two sequences — [edit distance](/citadel/dsa/edit-distance), LCS, [interleaving](/citadel/dsa/interleaving-string).
- **Interval** `dp[l][r]` combining over a split point $m$ — matrix chain, [burst balloons](/citadel/dsa/burst-balloons), optimal BST.
- **Tree** `dp[v][...]` from children — plus [rerooting](/citadel/cp/rerooting-technique) when you need the answer for every root.
- **Bitmask** `dp[mask]` — assignment, Hamiltonian path, [broken profile](/citadel/cp/broken-profile-dp), and sum-over-subsets DP via [submask enumeration](/citadel/cp/enumerating-submasks).
- **Digit DP** `dp[pos][tight][...]` — counting numbers in $[0, N]$ with a digit property.

## The four optimisations

| technique | recurrence form | brings | condition |
| --- | --- | --- | --- |
| [Divide and conquer DP](/citadel/cp/divide-and-conquer-dp) | $dp[i][j] = \min_k dp[i-1][k] + C(k, j)$ | $O(kn\log n)$ from $O(kn^2)$ | optimal $k$ monotone in $j$ |
| [Knuth's optimisation](/citadel/cp/knuths-optimization) | $dp[l][r] = \min_{l \le k < r} dp[l][k] + dp[k+1][r] + w(l,r)$ | $O(n^2)$ from $O(n^3)$ | $opt[l][r-1] \le opt[l][r] \le opt[l+1][r]$ |
| [Convex hull trick](/citadel/cp/convex-hull-trick) | $dp[i] = \min_j (a_j \cdot x_i + b_j)$ | $O(n)$ or $O(n\log n)$ from $O(n^2)$ | lines added in slope order, or Li Chao tree |
| [Broken profile](/citadel/cp/broken-profile-dp) | tiling / grid state along a frontier | $O(\text{cells} \cdot 2^{\text{width}})$ | narrow dimension $\le \sim 16$ |

There is also **sum-over-subsets (SOS) DP**: computing, for every mask, an aggregate over all its submasks in $O(2^n \cdot n)$ instead of $O(3^n)$, by fixing one bit at a time.

## Recognising which optimisation

- Cost function $C(k, j)$ satisfies the quadrangle inequality, and you have a "$k$ groups" or "$k$ segments" DP → divide and conquer DP; if it is also an interval DP with $dp[l][r]$, → Knuth.
- Transition is "best over previous states of a linear function of the current index" → convex hull trick / Li Chao.
- One grid dimension is tiny → iterate a bitmask profile across the other.
- "For each mask, sum/max over subsets" → SOS DP.

## Where this goes next

The [Combinatorics](/citadel/cp/combinatorics) section leans on counting DPs ([Catalan](/citadel/cp/catalan-numbers), [bracket sequences](/citadel/cp/balanced-bracket-sequences), [bishops](/citadel/cp/placing-bishops-on-a-chessboard)); [Graphs](/citadel/cp/graphs) has DP on the DFS tree and on DAGs. The optimisations here recur in geometry ([convex hull trick](/citadel/cp/convex-hull-trick) is literally a geometry structure) and in string DP.
