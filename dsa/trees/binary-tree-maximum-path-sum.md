---
title: Binary Tree Maximum Path Sum - Return a Branch, Record a Peak
description: The maximum sum of any node-to-node path — each recursion returns the best single downward branch, while a global tracks the best path that bends through the current node.
date: 2021-10-16
draft: false
slug: /dsa/binary-tree-maximum-path-sum
tags:
  - Competitive Programming
  - Trees
  - Depth-First Search
---

A path has one highest node. There, its sum is `node.val + bestLeftBranch + bestRightBranch`. But a node can only *contribute upward* through one of its children. So the recursion returns "best sum of a downward branch from here" and, as a side effect, updates a global with the full bent path.

## Description

Given the `root` of a binary tree, return the maximum path sum of any non-empty path. A path is a sequence of nodes connected by edges, each node appearing once, and it need not pass through the root.

**Example**

```
Input:  root = [1,2,3]
Output: 6          # 2 + 1 + 3

Input:  root = [-10,9,20,null,null,15,7]
Output: 42         # 15 + 20 + 7
```

**Constraints**

- `1` to $3 \times 10^4$ nodes.
- $-1000 \le \text{Node.val} \le 1000$

## Prerequisites

- [Diameter of Binary Tree](/citadel/dsa/diameter-of-binary-tree) — the "return one thing, update a global with another" shape.
- Clamping negative contributions to zero.

## Approach 1: Depth First Search

### Intuition

Straightforward but wasteful: a helper computes the best downward path from a node; the main recursion, at each node, forms `node.val + down(left) + down(right)`, updates the answer, and recurses into the children — recomputing `down` each time.

### Algorithm

1. `down(node)`: `0` if `None` or if `node.val + max(down(left), down(right))` is negative; else `node.val + max(0, down(left), down(right))`.
2. For each node: `through = node.val + max(0, down(left)) + max(0, down(right))`; update `best`; recurse into children.

```python
def maxPathSum(root) -> int:
    best = float("-inf")

    def down(node):
        if not node:
            return 0
        return max(0, node.val + max(down(node.left), down(node.right)))

    def visit(node):
        nonlocal best
        if not node:
            return
        through = node.val + max(0, down(node.left)) + max(0, down(node.right))
        best = max(best, through)
        visit(node.left)
        visit(node.right)

    visit(root)
    return best
```

### Complexity

- **Time:** $O(n^2)$ — `down` is recomputed at every node.
- **Space:** $O(n)$ recursion.

## Approach 2: Depth First Search (Optimal)

### Intuition

Fuse the two recursions. One post-order pass: compute each child's best downward branch, clamp negatives to `0`, update the global with the bent path `node.val + leftGain + rightGain`, and return `node.val + max(leftGain, rightGain)` for the parent to extend.

### Algorithm

1. `best = root.val`.
2. `dfs(node)`: return `0` if `None`.
3. `left_gain = max(dfs(node.left), 0)`; `right_gain = max(dfs(node.right), 0)`.
4. `best = max(best, node.val + left_gain + right_gain)`.
5. Return `node.val + max(left_gain, right_gain)`.

```python
def maxPathSum(root) -> int:
    best = root.val

    def dfs(node):
        nonlocal best
        if not node:
            return 0
        left_gain = max(dfs(node.left), 0)
        right_gain = max(dfs(node.right), 0)
        best = max(best, node.val + left_gain + right_gain)
        return node.val + max(left_gain, right_gain)

    dfs(root)
    return best
```

### Complexity

- **Time:** $O(n)$ — one visit per node.
- **Space:** $O(h)$ recursion.

## Common Pitfalls

- **Returning the bent path instead of the branch.** The parent can only attach through *one* child, so `dfs` returns `node.val + max(left_gain, right_gain)`; the two-child sum is only recorded in `best`.
- **Not clamping negative gains.** A subtree with a negative best contribution should be dropped (`max(gain, 0)`), not added.
- **Initialising `best` to `0`.** All-negative trees (`[-3]`) have answer `-3`; seed `best` with `root.val` or `-inf`, not `0`.

## The keystone

"Return the extendable part, record the complete part" resolves the tension between what a parent can use (a single branch) and what the answer allows (a path bending at one node). This is the same engine as [Diameter of Binary Tree](/citadel/dsa/diameter-of-binary-tree), with sums instead of lengths and a negative-clamp added.
