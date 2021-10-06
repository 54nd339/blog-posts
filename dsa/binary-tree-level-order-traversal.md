---
title: Binary Tree Level Order Traversal - One List Per Depth
description: Grouping node values by depth — a queue processed one level at a time is the natural fit, and a DFS that appends to result[depth] works too.
date: 2021-10-06
draft: false
slug: /dsa/binary-tree-level-order-traversal
tags:
  - Competitive Programming
  - Trees
  - Breadth-First Search
---

Level-order output is a list of lists, one per depth. BFS with a queue produces it directly by draining a whole level before starting the next; DFS produces it by indexing into `result` with the current depth.

## Description

Given the `root` of a binary tree, return its level-order traversal — a list where entry `i` is the list of values at depth `i`, left to right.

**Example**

```
Input:  root = [3,9,20,null,null,15,7]
Output: [[3], [9,20], [15,7]]

Input:  root = []
Output: []
```

**Constraints**

- `0` to `2000` nodes.
- $-1000 \le \text{Node.val} \le 1000$

## Prerequisites

- Level-order BFS with a queue.
- DFS carrying a depth parameter.

## Approach 1: Depth First Search

### Intuition

Carry the current depth. The first time you reach a new depth, start a fresh list; append every node's value to the list at its depth.

### Algorithm

1. `res = []`.
2. `dfs(node, depth)`: if `node` is `None`, return. If `depth == len(res)`, append `[]` to `res`. Append `node.val` to `res[depth]`. Recurse left then right with `depth + 1`.
3. `dfs(root, 0)`; return `res`.

```python
def levelOrder(root):
    res = []

    def dfs(node, depth):
        if not node:
            return
        if depth == len(res):
            res.append([])
        res[depth].append(node.val)
        dfs(node.left, depth + 1)
        dfs(node.right, depth + 1)

    dfs(root, 0)
    return res
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(h)$ recursion plus $O(n)$ output.

## Approach 2: Breadth First Search

### Intuition

A queue holds one level at a time. Snapshot its size, pop exactly that many nodes into a level list, enqueuing their children for the next round.

### Algorithm

1. If `root` is `None`, return `[]`. Queue containing `root`.
2. While non-empty: `size = len(queue)`; build `level` by popping `size` nodes and enqueuing their children; append `level` to `res`.
3. Return `res`.

```python
from collections import deque

def levelOrder(root):
    if not root:
        return []
    res = []
    q = deque([root])
    while q:
        level = []
        for _ in range(len(q)):
            node = q.popleft()
            level.append(node.val)
            if node.left:
                q.append(node.left)
            if node.right:
                q.append(node.right)
        res.append(level)
    return res
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ — the widest level.

## Common Pitfalls

- **Not snapshotting `len(q)` before the inner loop.** Enqueuing children during the loop grows `q`, so a live `while q` inner condition mixes levels.
- **DFS: `depth < len(res)` instead of `== len(res)`.** Since depth increases by one and `res` grows by one list at a time, the new-level condition is exact equality.
- **Returning `[[]]` for an empty tree.** Guard `root is None` and return `[]`.

## The keystone

BFS with a per-level size snapshot is the template for every "do something per level" tree problem — [Binary Tree Right Side View](/citadel/dsa/binary-tree-right-side-view) is the same loop keeping only the last value of each level.
