---
title: Balanced Binary Tree - Return a Height, Signal Imbalance
description: Whether every node's two subtrees differ in height by at most one — checking heights per node is O(n squared), but one bottom-up pass returns heights and short-circuits on the first violation.
date: 2021-10-01
draft: false
slug: /dsa/balanced-binary-tree
tags:
  - Competitive Programming
  - Trees
  - Depth-First Search
---

The naive check recomputes subtree heights at every node. The linear version computes each height exactly once, bottom-up, and returns a sentinel the moment it finds a node whose children's heights differ by more than one.

## Description

Given a binary tree, return `true` if it is height-balanced — for every node, the heights of its left and right subtrees differ by at most `1`.

**Example**

```
Input:  root = [3,9,20,null,null,15,7]
Output: true

Input:  root = [1,2,2,3,3,null,null,4,4]
Output: false
```

**Constraints**

- `0` to `5000` nodes.
- $-10^4 \le \text{Node.val} \le 10^4$

## Prerequisites

- [Maximum Depth of Binary Tree](/citadel/dsa/maximum-depth-of-binary-tree) — the height computation.
- Returning a compound result (or a sentinel) up the recursion.

## Approach 1: Brute Force

### Intuition

At each node, compute both subtree heights, check the difference, and recurse — heights get recomputed all the way down.

### Algorithm

1. `height(node)`: `0` if `None`, else `1 + max(height(left), height(right))`.
2. `isBalanced(node)`: `True` if `None`; else `abs(height(left) - height(right)) <= 1` **and** `isBalanced(left)` **and** `isBalanced(right)`.

```python
def isBalanced(root) -> bool:
    def height(node):
        if not node:
            return 0
        return 1 + max(height(node.left), height(node.right))

    if not root:
        return True
    if abs(height(root.left) - height(root.right)) > 1:
        return False
    return isBalanced(root.left) and isBalanced(root.right)
```

### Complexity

- **Time:** $O(n^2)$ — `height` at every node.
- **Space:** $O(n)$ recursion.

## Approach 2: Depth First Search

### Intuition

Have one recursion return the height, but return `-1` as a poison value if any subtree is unbalanced. Once `-1` appears it propagates straight to the root.

### Algorithm

1. `dfs(node)`: return `0` if `None`.
2. `L = dfs(left)`; if `L == -1`, return `-1`. `R = dfs(right)`; if `R == -1`, return `-1`.
3. If `abs(L - R) > 1`, return `-1`. Else return `1 + max(L, R)`.
4. The tree is balanced iff `dfs(root) != -1`.

```python
def isBalanced(root) -> bool:
    def dfs(node):
        if not node:
            return 0
        L = dfs(node.left)
        if L == -1:
            return -1
        R = dfs(node.right)
        if R == -1:
            return -1
        if abs(L - R) > 1:
            return -1
        return 1 + max(L, R)

    return dfs(root) != -1
```

### Complexity

- **Time:** $O(n)$ — one visit per node.
- **Space:** $O(h)$ recursion.

## Approach 3: Iterative DFS

### Intuition

Post-order traversal with a stack and a height map; a parent reads its children's heights and marks the whole tree unbalanced on the first violation.

### Algorithm

1. Post-order stack; `heights = {}`; `balanced = True`.
2. When finalising a node: `L = heights.get(left, 0)`, `R = heights.get(right, 0)`; if `abs(L - R) > 1`, set `balanced = False`; `heights[node] = 1 + max(L, R)`.
3. Return `balanced`.

```python
def isBalanced(root) -> bool:
    stack = [(root, False)]
    heights = {}
    balanced = True
    while stack:
        node, processed = stack.pop()
        if not node:
            continue
        if processed:
            L = heights.get(node.left, 0)
            R = heights.get(node.right, 0)
            if abs(L - R) > 1:
                balanced = False
            heights[node] = 1 + max(L, R)
        else:
            stack.append((node, True))
            stack.append((node.left, False))
            stack.append((node.right, False))
    return balanced
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **Not short-circuiting on `-1`.** If you compute `R` even after `L == -1`, you still get $O(n)$ time here, but the early returns keep the intent clear and avoid pointless work in a big unbalanced subtree.
- **Checking balance top-down.** A node can satisfy the height-difference rule while a descendant violates it; the check must run at *every* node, which the bottom-up return does for free.
- **Using `0` as the poison value.** `0` is a legitimate height (empty subtree); pick a value no real height can take, like `-1`.

## The keystone

Overloading a recursion's return value with a sentinel is a clean way to fuse "compute a quantity" and "detect a violation" into one pass. The same trick — return the useful number, or a flag that means "give up" — recurs in [Validate Binary Search Tree](/citadel/dsa/validate-binary-search-tree).
