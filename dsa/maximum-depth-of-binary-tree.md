---
title: Maximum Depth of Binary Tree - One Plus the Deeper Child
description: The height of a binary tree — recursion returns one plus the max of its two subtrees, and a stack or queue does the same iteratively.
date: 2021-09-28
draft: false
slug: /dsa/maximum-depth-of-binary-tree
tags:
  - Competitive Programming
  - Trees
  - Recursion
---

The depth of a tree is `1 + max(depth of left, depth of right)`, with an empty tree at depth `0`. That recurrence is the entire problem; the approaches differ only in how the pending subtrees are tracked.

## Description

Given the `root` of a binary tree, return its maximum depth — the number of nodes on the longest root-to-leaf path.

**Example**

```
Input:  root = [3,9,20,null,null,15,7]
Output: 3

Input:  root = [1,null,2]
Output: 2
```

**Constraints**

- `0` to $10^4$ nodes.
- $-100 \le \text{Node.val} \le 100$

## Prerequisites

- The height recurrence for trees.
- DFS (recursive or stack) and level-order BFS.

## Approach 1: Recursive DFS

### Intuition

Depth of a node is one more than the depth of its deeper subtree.

### Algorithm

1. If `root` is `None`, return `0`.
2. Return `1 + max(maxDepth(root.left), maxDepth(root.right))`.

```python
def maxDepth(root) -> int:
    if not root:
        return 0
    return 1 + max(maxDepth(root.left), maxDepth(root.right))
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(h)$ recursion.

## Approach 2: Iterative DFS (Stack)

### Intuition

Carry each node's depth alongside it on the stack; track the largest depth seen.

### Algorithm

1. If `root` is `None`, return `0`. Stack containing `(root, 1)`, `best = 0`.
2. While non-empty: pop `(node, depth)`; `best = max(best, depth)`; push each child with `depth + 1`.
3. Return `best`.

```python
def maxDepth(root) -> int:
    if not root:
        return 0
    stack = [(root, 1)]
    best = 0
    while stack:
        node, depth = stack.pop()
        best = max(best, depth)
        if node.left:
            stack.append((node.left, depth + 1))
        if node.right:
            stack.append((node.right, depth + 1))
    return best
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Approach 3: Breadth First Search

### Intuition

Count how many complete levels the queue processes.

### Algorithm

1. If `root` is `None`, return `0`. Queue containing `root`, `levels = 0`.
2. While non-empty: for each of the current `len(queue)` nodes, pop it and enqueue its children; then `levels += 1`.
3. Return `levels`.

```python
from collections import deque

def maxDepth(root) -> int:
    if not root:
        return 0
    q = deque([root])
    levels = 0
    while q:
        for _ in range(len(q)):
            node = q.popleft()
            if node.left:
                q.append(node.left)
            if node.right:
                q.append(node.right)
        levels += 1
    return levels
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ — the widest level.

## Common Pitfalls

- **BFS: not snapshotting the level size.** You must capture `len(q)` before the inner loop, or you process nodes from the next level in the same round and the count is wrong.
- **Recursion on `None` returning `1`.** The empty tree is depth `0`; a single node is depth `1`.
- **Confusing depth (node count) with edge count.** LeetCode's "depth" here counts nodes on the path.

## The keystone

`f(node) = combine(f(node.left), f(node.right))` is the shape of nearly every tree problem. Max depth is the simplest instance — `combine` is `1 + max(...)`. [Diameter of Binary Tree](/citadel/dsa/diameter-of-binary-tree) and [Balanced Binary Tree](/citadel/dsa/balanced-binary-tree) reuse this exact height computation while accumulating an extra fact on the side.
