---
title: Binary Tree Right Side View - The Last Node of Each Level
description: The values visible when looking at a tree from the right — take the rightmost node of every level, via BFS or a right-first DFS that records the first node seen at each new depth.
date: 2021-10-06
draft: false
slug: /dsa/binary-tree-right-side-view
tags:
  - Competitive Programming
  - Trees
  - Breadth-First Search
---

From the right, you see exactly one node per level, namely the rightmost. So this is level-order traversal keeping only the last element of each level — or a DFS that visits the right child first and records the first node it meets at each depth.

## Description

Given the `root` of a binary tree, return the values of the nodes you can see ordered top to bottom when looking at the tree from the right side.

**Example**

```
Input:  root = [1,2,3,null,5,null,4]
Output: [1,3,4]

Input:  root = [1,null,3]
Output: [1,3]
```

**Constraints**

- `0` to `100` nodes.
- $-100 \le \text{Node.val} \le 100$

## Prerequisites

- [Binary Tree Level Order Traversal](/citadel/dsa/binary-tree-level-order-traversal).
- DFS with a depth parameter and a chosen child-visit order.

## Approach 1: Depth First Search

### Intuition

Visit the right child before the left. Then at each depth, the *first* node you encounter is the rightmost one on that level.

### Algorithm

1. `res = []`.
2. `dfs(node, depth)`: if `node` is `None`, return. If `depth == len(res)`, append `node.val`. Recurse right, then left, with `depth + 1`.
3. `dfs(root, 0)`; return `res`.

```python
def rightSideView(root):
    res = []

    def dfs(node, depth):
        if not node:
            return
        if depth == len(res):
            res.append(node.val)
        dfs(node.right, depth + 1)
        dfs(node.left, depth + 1)

    dfs(root, 0)
    return res
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(h)$ recursion.

## Approach 2: Breadth First Search

### Intuition

Level-order traversal; append the last node dequeued from each level.

### Algorithm

1. If `root` is `None`, return `[]`. Queue containing `root`.
2. While non-empty: `size = len(queue)`; pop `size` nodes, enqueuing children; the value of the `size`-th (last) node goes into `res`.
3. Return `res`.

```python
from collections import deque

def rightSideView(root):
    if not root:
        return []
    res = []
    q = deque([root])
    while q:
        size = len(q)
        for i in range(size):
            node = q.popleft()
            if i == size - 1:
                res.append(node.val)
            if node.left:
                q.append(node.left)
            if node.right:
                q.append(node.right)
    return res
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **DFS visiting left before right.** Then you record the *left* side view. Recurse right first.
- **BFS taking `node.left` or the first node as the visible one.** It is the last node dequeued at that level (`i == size - 1`).
- **Assuming the rightmost node is always a right child.** In `[1,2,3,null,5]` the level-2 view is `5`, a left child, because node `3` has no children.

## The keystone

"Right-first DFS, record the first hit per depth" and "BFS, keep the last per level" are two angles on the same fact: the right side view is a one-per-level projection. Swapping the recursion order to control which node you see first is a handy trick for level-indexed problems.
