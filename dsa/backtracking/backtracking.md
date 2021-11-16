---
title: Backtracking - One Tree, Many Constraints
description: A guide to the Backtracking section of NeetCode 150 — every problem is the same choose, recurse, undo tree, and the differences are which choices are legal and how duplicates are pruned.
date: 2021-11-16
draft: false
slug: /dsa/backtracking
tags:
  - Competitive Programming
  - Backtracking
  - Recursion
---

Backtracking is depth-first search over a tree of partial solutions. At each node you make a choice, recurse, then undo the choice so the next sibling starts from a clean state. Every problem in this section is that loop; what changes is the shape of the choice and which branches you refuse to enter.

## The rhythm

```python
def dfs(state):
    if is_complete(state):
        record(state)
        return
    for choice in choices(state):
        if not allowed(choice, state):
            continue
        apply(choice, state)     # mark
        dfs(state)               # recurse
        undo(choice, state)      # restore
```

The `undo` is what makes it *backtracking* rather than plain recursion — one mutable `state` is reused across the whole tree, so every mutation must be reversible.

## Two base trees

- **The $2^n$ tree — include or exclude.** [Subsets](/citadel/dsa/subsets) is the pure form: for each element, branch on "in" or "out". [Combination Sum](/citadel/dsa/combination-sum) and [Combination Sum II](/citadel/dsa/combination-sum-ii) are this tree with a running sum and a forward-only index so unordered combinations are not counted twice. [Palindrome Partitioning](/citadel/dsa/palindrome-partitioning) reads it as "where is the next cut", legal only when the prefix is a palindrome.
- **The $n!$ tree — pick one of the remaining.** [Permutations](/citadel/dsa/permutations) fills each position with an unused element, tracked by a boolean array, a bitmask, or in-place swaps. [N-Queens](/citadel/dsa/n-queens) is one queen per row with three $O(1)$ consistency checks (column and both diagonals).

## The recurring constraints

- **Unordered, so dedupe by direction.** Only ever move forward through the input; never revisit an earlier index on a sibling branch. This turns `[2,3]` and `[3,2]` into one path.
- **Duplicate values in the input.** Sort first, then at each recursion *level* skip a value equal to its left neighbour (`j > start and a[j] == a[j-1]`). Used in [Subsets II](/citadel/dsa/subsets-ii), [Combination Sum II](/citadel/dsa/combination-sum-ii), and duplicate-tolerant permutations.
- **A feasibility gate on each choice.** [Word Search](/citadel/dsa/word-search) only steps onto a cell not already on the path; [Letter Combinations of a Phone Number](/citadel/dsa/letter-combinations-of-a-phone-number) has *no* gate, so it is a full Cartesian product and brute force is optimal.
- **Pruning.** Sort so a hopeless branch becomes an early `break` (Combination Sum); precompute a table so a repeated check becomes a lookup (Palindrome Partitioning's `dp`, Word Search II's trie).

## The nine problems

- [Subsets](/citadel/dsa/subsets) — the $2^n$ include/exclude tree.
- [Combination Sum](/citadel/dsa/combination-sum) — target sum, reuse allowed, forward-only.
- [Combination Sum II](/citadel/dsa/combination-sum-ii) — no reuse, duplicate candidates, level-skip.
- [Permutations](/citadel/dsa/permutations) — the $n!$ tree; `used` marker or swap.
- [Subsets II](/citadel/dsa/subsets-ii) — power set of a multiset; sort and level-skip.
- [Word Search](/citadel/dsa/word-search) — grid DFS with a reversible mark on the current path.
- [Palindrome Partitioning](/citadel/dsa/palindrome-partitioning) — cut positions, palindrome gate.
- [Letter Combinations of a Phone Number](/citadel/dsa/letter-combinations-of-a-phone-number) — unconstrained product of per-digit letter sets.
- [N-Queens](/citadel/dsa/n-queens) — one queen per row, column and diagonal sets.

## Recognising it

- "all combinations / all subsets / all permutations / all ways to ..." with `n` small (often $\le 20$) → backtracking.
- The answer is a *list of configurations*, not a count or a single optimum (those want DP).
- Each partial solution can be extended one choice at a time and checked for validity as you go.

## Where this goes next

The grid-DFS problems here are one step from [Graphs](/citadel/dsa/graphs), where the same "visit, recurse, the visited set need not be undone" traversal counts components and detects cycles. And when you only need *how many* valid configurations exist, or the *best* one, the exponential tree usually collapses into [1-D Dynamic Programming](/citadel/dsa/1-d-dynamic-programming) by memoizing on the state instead of enumerating every leaf.
