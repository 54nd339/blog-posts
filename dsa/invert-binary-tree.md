---
title: Invert a Binary Tree - Swap Every Pair of Children
description: Mirroring a binary tree — visit every node and swap its two children, by recursion, an explicit stack, or a queue, all in O(n).
date: 2021-09-26
draft: false
slug: /dsa/invert-binary-tree
tags:
  - Competitive Programming
  - Trees
  - Recursion
---

Inverting a tree is one operation applied everywhere, namely swap `left` and `right`. Any traversal that visits every node once will do; the choice is only whether the pending nodes live on the call stack, an explicit stack, or a queue.

## Description

Given the `root` of a binary tree, invert it (mirror it left-to-right) and return the root.

**Example**

```
Input:  root = [4,2,7,1,3,6,9]
Output: [4,7,2,9,6,3,1]

Input:  root = []
Output: []
```

**Constraints**

- `0` to `100` nodes.
- $-100 \le \text{Node.val} \le 100$

## Prerequisites

- Binary tree node with `val`, `left`, `right`.
- Any one of pre/post-order DFS or level-order BFS.

## Approach 1: Breadth First Search

### Intuition

Process nodes level by level with a queue; swap each node's children as it comes off the queue, then enqueue the (now swapped) children.

### Algorithm

1. If `root` is `None`, return `None`.
2. Queue containing `root`.
3. While the queue is non-empty: pop a node, swap its `left` and `right`, enqueue any non-null children.
4. Return `root`.

```python
from collections import deque

def invertTree(root):
    if not root:
        return None
    q = deque([root])
    while q:
        node = q.popleft()
        node.left, node.right = node.right, node.left
        if node.left:
            q.append(node.left)
        if node.right:
            q.append(node.right)
    return root
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ — the queue holds up to a full level.

## Approach 2: Depth First Search

### Intuition

Swap the current node's children, then recurse into each.

### Algorithm

1. If `root` is `None`, return `None`.
2. Swap `root.left` and `root.right`.
3. Recurse on `root.left`, then `root.right`.
4. Return `root`.

```python
def invertTree(root):
    if not root:
        return None
    root.left, root.right = root.right, root.left
    invertTree(root.left)
    invertTree(root.right)
    return root
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(h)$ recursion, $h$ the height — $O(\log n)$ balanced, $O(n)$ degenerate.

## Approach 3: Iterative DFS

### Intuition

The recursion with an explicit stack instead of the call stack.

### Algorithm

1. If `root` is `None`, return `None`.
2. Stack containing `root`.
3. While non-empty: pop a node, swap its children, push non-null children.
4. Return `root`.

```python
def invertTree(root):
    if not root:
        return None
    stack = [root]
    while stack:
        node = stack.pop()
        node.left, node.right = node.right, node.left
        if node.left:
            stack.append(node.left)
        if node.right:
            stack.append(node.right)
    return root
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ for the stack.

## Common Pitfalls

- **Recursing before swapping, then swapping the results incorrectly.** Swap first, then recurse into the new positions — or recurse first and swap the returned subtrees; just be consistent.
- **BFS/iterative: enqueuing children before swapping.** Swap `node.left`/`node.right` *then* read them to enqueue, or you traverse the un-inverted structure (harmless for a full inversion, but a bug if you also read values).

## The keystone

"Apply one local operation at every node" is the essence of tree recursion — `invert` is the minimal example. The three shells (call stack / explicit stack / queue) are interchangeable here and reused across [Maximum Depth of Binary Tree](/citadel/dsa/maximum-depth-of-binary-tree) and the rest of the section.
