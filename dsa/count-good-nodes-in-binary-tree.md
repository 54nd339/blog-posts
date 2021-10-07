---
title: Count Good Nodes in a Binary Tree - Carry the Path Maximum Down
description: Counting nodes with no larger value on the path from the root — pass the running maximum downward, and a node is good when it is at least that maximum.
date: 2021-10-07
draft: false
slug: /dsa/count-good-nodes-in-binary-tree
tags:
  - Competitive Programming
  - Trees
  - Depth-First Search
---

A node is "good" if nothing on the root-to-node path is strictly larger than it. So carry the maximum value seen so far *downward* as you recurse; compare each node against it, then pass the updated maximum to the children.

## Description

Given the `root` of a binary tree, return the number of good nodes — nodes `x` such that no node on the path from the root to `x` has a value greater than `x.val`. The root is always good.

**Example**

```
Input:  root = [3,1,4,3,null,1,5]
Output: 4          # nodes 3 (root), 4, 5, and the left-subtree 3

Input:  root = [1,3,2,1,3,null,1,null,null,1]
Output: 3
```

**Constraints**

- `1` to $10^5$ nodes.
- $-10^4 \le \text{Node.val} \le 10^4$

## Prerequisites

- DFS with an accumulator passed *down* (as opposed to returned up).

## Approach 1: Depth First Search

### Intuition

Recurse with the path maximum as a parameter. Count the node if `node.val >= max_so_far`, then recurse with `max(max_so_far, node.val)`.

### Algorithm

1. `dfs(node, max_so_far)`: return `0` if `node` is `None`.
2. `count = 1 if node.val >= max_so_far else 0`.
3. `new_max = max(max_so_far, node.val)`.
4. Return `count + dfs(node.left, new_max) + dfs(node.right, new_max)`.
5. Call `dfs(root, root.val)` (or `-inf`).

```python
def goodNodes(root) -> int:
    def dfs(node, max_so_far):
        if not node:
            return 0
        count = 1 if node.val >= max_so_far else 0
        new_max = max(max_so_far, node.val)
        return count + dfs(node.left, new_max) + dfs(node.right, new_max)

    return dfs(root, root.val)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(h)$ recursion.

## Approach 2: Breadth First Search

### Intuition

Level-order (or any order) with each queued entry carrying its path maximum.

### Algorithm

1. Queue containing `(root, -inf)`. `count = 0`.
2. Pop `(node, mx)`: if `node.val >= mx`, `count += 1`. Enqueue each child with `max(mx, node.val)`.
3. Return `count`.

```python
from collections import deque

def goodNodes(root) -> int:
    count = 0
    q = deque([(root, float("-inf"))])
    while q:
        node, mx = q.popleft()
        if node.val >= mx:
            count += 1
        for child in (node.left, node.right):
            if child:
                q.append((child, max(mx, node.val)))
    return count
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **`>` instead of `>=`.** A node equal to the path maximum is still good; a strict comparison undercounts, and misclassifies the root whenever you seed the max with `root.val`.
- **Updating the max before the comparison.** Compare `node.val` against the max from *ancestors only*, then fold `node.val` in for the children.
- **Passing the max up instead of down.** This is a top-down accumulation; the answer is a simple sum of subtree counts, no post-order return needed.

## The keystone

Some tree facts depend on the path *from the root*, not the subtree below. Those are solved by threading an accumulator downward through the recursion parameters. Contrast [Diameter of Binary Tree](/citadel/dsa/diameter-of-binary-tree), where the needed fact comes from *below* and is returned upward.
