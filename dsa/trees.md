---
title: Trees - Recursion With Two Branches
description: A guide to the Trees section of NeetCode 150 — almost every problem is one recursion, and the only real choices are what each call returns, what it accumulates on the side, and which direction the extra information flows.
date: 2021-09-24
draft: false
slug: /dsa/trees
tags:
  - Competitive Programming
  - Trees
  - Recursion
---

A binary-tree problem is usually a single recursive function. The design questions are always the same three: what does a call **return** to its parent, what does it **update** on the side, and does the extra fact flow **down** (from ancestors) or **up** (from descendants)?

## The traversal shells

Every problem here is a traversal plus a small amount of work per node. The pending nodes live somewhere:

- **Call stack** — plain recursion. The default.
- **Explicit stack** — the same order, iterative; useful when recursion depth is a concern or you want manual control.
- **Queue** — level-order (BFS). Use it when the answer is organised by depth.

[Invert Binary Tree](/citadel/dsa/invert-binary-tree), [Maximum Depth of Binary Tree](/citadel/dsa/maximum-depth-of-binary-tree), and [Same Tree](/citadel/dsa/same-tree) show all three shells on trivial per-node work.

## Information flowing up

The recursion returns a value computed from the children.

- [Maximum Depth of Binary Tree](/citadel/dsa/maximum-depth-of-binary-tree) — return `1 + max(left, right)`.
- [Diameter of Binary Tree](/citadel/dsa/diameter-of-binary-tree) — return the height, update a global with `left + right`.
- [Balanced Binary Tree](/citadel/dsa/balanced-binary-tree) — return the height, or `-1` as a "give up" sentinel.
- [Binary Tree Maximum Path Sum](/citadel/dsa/binary-tree-maximum-path-sum) — return the best single branch, update a global with the bent path.
- [Count Good Nodes in a Binary Tree](/citadel/dsa/count-good-nodes-in-binary-tree) is the counterpoint: the fact (path maximum) flows *down* as a parameter.

The recurring trick: **return the extendable quantity, record the complete quantity separately.**

## BST-specific structure

The ordering invariant turns search into an $O(h)$ walk and makes in-order traversal a sorted stream.

- [Lowest Common Ancestor of a BST](/citadel/dsa/lowest-common-ancestor-of-a-bst) — descend until the two targets split.
- [Validate Binary Search Tree](/citadel/dsa/validate-binary-search-tree) — carry an open interval `(low, high)` downward.
- [Kth Smallest Element in a BST](/citadel/dsa/kth-smallest-element-in-a-bst) — in-order, stop at the k-th.

## Serialisation and reconstruction

- [Construct Binary Tree from Preorder and Inorder](/citadel/dsa/construct-binary-tree-from-preorder-and-inorder-traversal) — one traversal names the root, the other splits the rest.
- [Serialize and Deserialize Binary Tree](/citadel/dsa/serialize-and-deserialize-binary-tree) — preorder plus a null marker per missing child is a reversible blueprint.
- [Subtree of Another Tree](/citadel/dsa/subtree-of-another-tree) — run the same-tree check at every anchor, or serialise both and string-match.

## Recognising the approach

- Answer depends on the subtree below a node → **return a value up**, maybe with a global for the "through" case.
- Answer depends on the path from the root → **pass an accumulator down**.
- Answer is grouped by level → **BFS**.
- Tree is a BST and the question says "k-th", "range", "sorted", "closest" → **in-order traversal**.
- Need to compare or match structure → **parallel traversal** or **serialise-and-search**.

## Where this goes next

The two-way branching recursion here generalises to the many-way branching of [Backtracking](/citadel/dsa/backtracking) and to graph DFS, where the only new complication is a `visited` set because the structure can have cycles.
