---
title: Diameter of Binary Tree - Compute Height, Track the Best Split
description: The longest path between any two nodes — recomputing heights per node is O(n squared), but one post-order pass returns heights while updating a global best of left-height plus right-height.
date: 2021-09-28
draft: false
slug: /dsa/diameter-of-binary-tree
tags:
  - Competitive Programming
  - Trees
  - Depth-First Search
---

The longest path passes through some node as its highest point, and there its length is `leftHeight + rightHeight` (in edges). So compute every subtree's height once, and while doing so keep a running maximum of that sum.

## Description

Given the `root` of a binary tree, return the length of its diameter — the number of edges on the longest path between any two nodes. The path need not pass through the root.

**Example**

```
Input:  root = [1,2,3,4,5]
Output: 3          # path 4 - 2 - 1 - 3 (or 5 - 2 - 1 - 3)

Input:  root = [1,2]
Output: 1
```

**Constraints**

- `1` to $10^4$ nodes.
- $-100 \le \text{Node.val} \le 100$

## Prerequisites

- [Maximum Depth of Binary Tree](/citadel/dsa/maximum-depth-of-binary-tree) — the height computation reused here.
- The "return one value, update a global with another" post-order pattern.

## Approach 1: Brute Force

### Intuition

For every node, its "through" diameter is `height(left) + height(right)`. Compute that for all nodes and take the max — but each height call re-walks a subtree.

### Algorithm

1. `height(node)`: `0` if `None`, else `1 + max(height(left), height(right))`.
2. `diameter(node)`: `0` if `None`, else `max(height(left) + height(right), diameter(left), diameter(right))`.

```python
def diameterOfBinaryTree(root) -> int:
    def height(node):
        if not node:
            return 0
        return 1 + max(height(node.left), height(node.right))

    def diameter(node):
        if not node:
            return 0
        through = height(node.left) + height(node.right)
        return max(through, diameter(node.left), diameter(node.right))

    return diameter(root)
```

### Complexity

- **Time:** $O(n^2)$ — `height` is $O(n)$ and is called at every node.
- **Space:** $O(n)$ recursion.

## Approach 2: Depth First Search

### Intuition

Fold the two jobs into one post-order pass. Each call returns the node's height; as a side effect it updates a global `best` with `leftHeight + rightHeight`.

### Algorithm

1. `best = 0`.
2. `dfs(node)`: `0` if `None`. Compute `L = dfs(left)`, `R = dfs(right)`. Update `best = max(best, L + R)`. Return `1 + max(L, R)`.
3. Run `dfs(root)`; return `best`.

```python
def diameterOfBinaryTree(root) -> int:
    best = 0

    def dfs(node):
        nonlocal best
        if not node:
            return 0
        L = dfs(node.left)
        R = dfs(node.right)
        best = max(best, L + R)
        return 1 + max(L, R)

    dfs(root)
    return best
```

### Complexity

- **Time:** $O(n)$ — one visit per node.
- **Space:** $O(h)$ recursion.

## Approach 3: Iterative DFS

### Intuition

A post-order traversal with an explicit stack and a map from node to its computed height, so a parent can read its children's heights.

### Algorithm

1. Stack of nodes for post-order; `heights = {None: 0}` (or default `0`); `best = 0`.
2. Traverse so that a node is finalised only after both children: pop it, read `L = heights[node.left]`, `R = heights[node.right]`; set `heights[node] = 1 + max(L, R)`; update `best` with `L + R`.
3. Return `best`.

```python
def diameterOfBinaryTree(root) -> int:
    best = 0
    stack = [(root, False)]
    heights = {}
    while stack:
        node, processed = stack.pop()
        if not node:
            continue
        if processed:
            L = heights.get(node.left, 0)
            R = heights.get(node.right, 0)
            heights[node] = 1 + max(L, R)
            best = max(best, L + R)
        else:
            stack.append((node, True))
            stack.append((node.left, False))
            stack.append((node.right, False))
    return best
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ for the stack and height map.

## Common Pitfalls

- **Returning `best` from `dfs` instead of the height.** The recursion must return the height for the parent's arithmetic; the diameter is accumulated separately in `best`.
- **Edges vs nodes.** The diameter here is counted in *edges*, so `L + R` (not `L + R + 1`) is the path length through a node when `L`, `R` are subtree *heights in edges* — and with the "height = 1 + max" convention above, `L + R` already equals the edge count of the through-path.
- **Brute force TLE.** The $O(n^2)$ version passes small trees but not $10^4$ nodes in a skewed shape.

## The keystone

"Return one quantity to your parent, and on the way update a global with a different quantity" is the workhorse pattern for tree problems whose answer is a path or a subtree property. [Balanced Binary Tree](/citadel/dsa/balanced-binary-tree) and [Binary Tree Maximum Path Sum](/citadel/dsa/binary-tree-maximum-path-sum) are the same shape.
