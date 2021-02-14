---
title: Algorithmic Paradigms - Greedy, Divide and Conquer, DP, and Backtracking
description: Four ways to design an algorithm, and how to measure one. Big-O for time and space, then the four paradigms — greedy, divide and conquer, dynamic programming, backtracking — with what each is for and where it wins.
date: 2021-02-14
draft: false
slug: /algorithms/Algorithms
tags:
  - Algorithms
  - Complexity
---

Most algorithms you'll meet are instances of one of four design patterns. Knowing the patterns turns "how do I solve this?" into "which of these four fits, and does it apply here?" This is a map of the paradigms — greedy, divide and conquer, dynamic programming, backtracking — plus the Big-O vocabulary for judging the result. Each links to a worked example.

## Measuring an algorithm

**Time complexity** describes how the running time grows with input size $n$, in the worst case, using Big-O.

| Notation | Name | Where it shows up |
| --- | --- | --- |
| $O(1)$ | constant | array index, hash lookup |
| $O(\log n)$ | logarithmic | binary search, balanced-tree operations |
| $O(n)$ | linear | one pass over the input |
| $O(n \log n)$ | linearithmic | merge sort, heap sort, FFT |
| $O(n^2)$ | quadratic | bubble/insertion sort worst case, all pairs |
| $O(n^3)$ | cubic | schoolbook matrix multiply, Floyd-Warshall |
| $O(2^n)$ | exponential | all subsets, naive 0/1 knapsack |
| $O(n!)$ | factorial | all permutations, brute-force TSP |

**Space complexity** is the same idea for memory: $O(1)$ for a fixed set of variables, $O(n)$ for a copy of the input or a 1-D table, $O(n^2)$ for an adjacency matrix or a 2-D DP table, $O(\log n)$ for a balanced recursion's call stack.

## Greedy

Build the solution one piece at a time, always taking the choice that looks best right now, and never reconsider. Fast and simple — but only correct when the problem has the **greedy choice property** (a locally optimal choice is part of some global optimum) and **optimal substructure**. When those hold, an exchange argument proves it; when they don't, greedy quietly returns a wrong answer.

- [Activity selection](/citadel/algorithms/ActivitySelection) — most non-overlapping intervals.
- [Fractional knapsack](/citadel/algorithms/FractionalKnapsack) — but *not* [0/1 knapsack](/citadel/algorithms/01Kanpsack), where greedy fails.
- [Huffman coding](/citadel/algorithms/HuffmanCoding) — optimal prefix codes.
- [Minimum spanning tree](/citadel/algorithms/MinimumSpanningTree) — Kruskal's and Prim's.

## Divide and conquer

Split the problem into independent subproblems of the same kind, solve them recursively, and combine. The running time follows a recurrence $T(n) = a\,T(n/b) + f(n)$, solved by the master theorem. The wins come from a clever *divide* or *combine* step that does less work than the obvious one.

- [Sorting and searching](/citadel/algorithms/SortingSearching) — merge sort, quick sort, binary search.
- [Karatsuba multiplication](/citadel/algorithms/Karatsuba) — three half-size multiplies, not four.
- [Strassen's algorithm](/citadel/algorithms/Strassen) — seven block multiplies, not eight.
- [Closest pair of points](/citadel/algorithms/ClosestPair) — a linear-time strip merge.
- [Fast Fourier transform](/citadel/algorithms/FastFourierTransform) — even/odd split with twiddle factors.

## Dynamic programming

For problems with **overlapping subproblems** (a naive recursion re-solves the same cases) *and* optimal substructure: solve each subproblem once and store it, either top-down (recursion plus a memo) or bottom-up (fill a table in dependency order). It turns an exponential search into a polynomial table fill.

- [0/1 knapsack](/citadel/algorithms/01Kanpsack) — value under a weight limit.
- [Longest common subsequence](/citadel/algorithms/LongestCommonSubsequence) — the engine behind `diff`.
- [Matrix chain multiplication](/citadel/algorithms/MatrixChainMultiplication) — the cheapest parenthesization.
- [Bellman-Ford and Floyd-Warshall](/citadel/algorithms/PathFinding) — shortest paths as DP.

## Backtracking

A depth-first walk of the solution space that abandons a partial candidate the instant it can't be completed. The four moves: extend the candidate, check consistency, recurse, undo. Worst case is exponential, but the "check" prunes most of the tree.

- [N-queens](/citadel/algorithms/NQueens) — one queen per row, prune on conflict.
- [Sudoku solver](/citadel/algorithms/SudokuSolver) — fill a cell, prune on rule violation.
- [Graph coloring](/citadel/algorithms/GraphColoring) — color a vertex, prune on neighbour clash.
- [Hamiltonian cycle](/citadel/algorithms/HamiltonianCycle) — extend the tour, prune on missing edge.

## Key takeaways

- **Big-O** classifies growth: aim for $O(n \log n)$ or better, treat $O(2^n)$ and $O(n!)$ as "small inputs only".
- **Greedy** — one-shot local choices; fastest, but correct only with the greedy choice property.
- **Divide and conquer** — recursive split and combine; the speedup is in a smarter merge, and the cost is a recurrence.
- **Dynamic programming** — cache overlapping subproblems; turns exponential into polynomial.
- **Backtracking** — pruned DFS of the candidate space; exponential worst case, practical with good pruning.
- Many of the hard problems above ([0/1 knapsack](/citadel/algorithms/01Kanpsack), [Hamiltonian cycle](/citadel/algorithms/HamiltonianCycle), [graph coloring](/citadel/algorithms/GraphColoring)) are [NP-complete](/citadel/algorithms/ComplexityClasses) — the paradigm gives you a practical attack, not a fast exact algorithm.
