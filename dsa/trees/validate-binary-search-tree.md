---
title: Validate Binary Search Tree - Every Node Has a Range
description: Checking the BST property globally, not just between parent and child — carry an open interval down the recursion and require each node's value to fall strictly inside it.
date: 2021-10-07
draft: false
slug: /dsa/validate-binary-search-tree
tags:
  - Competitive Programming
  - Trees
  - Binary Search Tree
---

The common wrong answer checks only `left < node < right` locally. That misses violations like a deep left-subtree value that exceeds an ancestor. The fix: each node inherits an allowed open interval `(low, high)` from its ancestors, and must lie strictly inside it.

## Description

Given the `root` of a binary tree, return `true` if it is a valid binary search tree — for every node, all values in its left subtree are strictly less, all in its right subtree strictly greater, and both subtrees are themselves valid BSTs.

**Example**

```
Input:  root = [2,1,3]
Output: true

Input:  root = [5,1,4,null,null,3,6]
Output: false          # 3 and 6 are in 5's right subtree but 3 < 5
```

**Constraints**

- `1` to $10^4$ nodes.
- $-2^{31} \le \text{Node.val} \le 2^{31} - 1$

## Prerequisites

- The BST ordering invariant applied *transitively*, not just parent-child.
- DFS with `(low, high)` bounds passed downward; in-order traversal yields sorted values.

## Approach 1: Brute Force

### Intuition

At each node, explicitly verify every left-subtree value is `< node.val` and every right-subtree value is `> node.val`, then recurse.

### Algorithm

1. `all_less(node, v)` / `all_greater(node, v)`: traverse the subtree checking the bound.
2. `isValidBST(node)`: `true` if `None`; else `all_less(node.left, node.val)` and `all_greater(node.right, node.val)` and both children valid.

```python
def isValidBST(root) -> bool:
    def all_less(node, v):
        if not node:
            return True
        return node.val < v and all_less(node.left, v) and all_less(node.right, v)

    def all_greater(node, v):
        if not node:
            return True
        return node.val > v and all_greater(node.left, v) and all_greater(node.right, v)

    if not root:
        return True
    if not all_less(root.left, root.val) or not all_greater(root.right, root.val):
        return False
    return isValidBST(root.left) and isValidBST(root.right)
```

### Complexity

- **Time:** $O(n^2)$ — a subtree scan at every node.
- **Space:** $O(n)$ recursion.

## Approach 2: Depth First Search (bounds)

### Intuition

Pass a valid open interval down. The root may be anything (`(-inf, +inf)`); going left tightens the upper bound to the node's value, going right tightens the lower bound.

### Algorithm

1. `dfs(node, low, high)`: return `true` if `node` is `None`.
2. If `node.val <= low` or `node.val >= high`, return `false`.
3. Return `dfs(node.left, low, node.val)` and `dfs(node.right, node.val, high)`.
4. Call `dfs(root, -inf, +inf)`.

```python
def isValidBST(root) -> bool:
    def dfs(node, low, high):
        if not node:
            return True
        if not (low < node.val < high):
            return False
        return dfs(node.left, low, node.val) and dfs(node.right, node.val, high)

    return dfs(root, float("-inf"), float("inf"))
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(h)$ recursion.

## Approach 3: Breadth First Search (bounds)

### Intuition

The same bounds check, but with a queue of `(node, low, high)` triples instead of recursion.

### Algorithm

1. Queue containing `(root, -inf, +inf)`.
2. Pop `(node, low, high)`: if `not (low < node.val < high)`, return `false`. Enqueue `(left, low, node.val)` and `(right, node.val, high)` for non-null children.
3. Return `true`.

```python
from collections import deque

def isValidBST(root) -> bool:
    if not root:
        return True
    q = deque([(root, float("-inf"), float("inf"))])
    while q:
        node, low, high = q.popleft()
        if not (low < node.val < high):
            return False
        if node.left:
            q.append((node.left, low, node.val))
        if node.right:
            q.append((node.right, node.val, high))
    return True
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **Only comparing a node to its parent.** `[5,1,4,null,null,3,6]` passes every local check but is invalid — `3` sits in `5`'s right subtree.
- **Using `<=` for the subtree relation.** Duplicates are not allowed; bounds must be strict (`low < val < high`).
- **Seeding bounds with `Node.val`'s numeric limits.** Values can reach $\pm(2^{31}-1)$, so use `float("-inf")` / `float("inf")` (or `None` with explicit checks), not `-2**31`.

## The keystone

When a local invariant must hold globally, thread the *tightened constraint* down the recursion so every node is checked against everything above it. The `(low, high)` interval is the constraint here; an in-order traversal is the alternative view, since a valid BST emits strictly increasing values.
