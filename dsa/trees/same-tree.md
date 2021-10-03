---
title: Same Tree - Compare Structure and Values in Lockstep
description: Deciding whether two binary trees are identical — walk both at once, and at each step require both nodes null, or both present with equal values and matching subtrees.
date: 2021-10-03
draft: false
slug: /dsa/same-tree
tags:
  - Competitive Programming
  - Trees
  - Recursion
---

Two trees are the same when, at every position, either both have no node or both have a node with the same value. Traverse the two trees together and check that condition everywhere.

## Description

Given the roots of two binary trees `p` and `q`, return `true` if they are structurally identical and every corresponding pair of nodes has the same value.

**Example**

```
Input:  p = [1,2,3], q = [1,2,3]
Output: true

Input:  p = [1,2], q = [1,null,2]
Output: false
```

**Constraints**

- Each tree has `0` to `100` nodes.
- $-10^4 \le \text{Node.val} \le 10^4$

## Prerequisites

- Simultaneous traversal of two trees.
- DFS (recursive or stack) and level-order BFS.

## Approach 1: Depth First Search

### Intuition

Recurse on both trees together; the base cases handle "both empty" and "one empty", the recursive case checks value equality and both subtrees.

### Algorithm

1. If both `p` and `q` are `None`, return `true`.
2. If exactly one is `None`, or `p.val != q.val`, return `false`.
3. Return `isSameTree(p.left, q.left)` **and** `isSameTree(p.right, q.right)`.

```python
def isSameTree(p, q) -> bool:
    if not p and not q:
        return True
    if not p or not q or p.val != q.val:
        return False
    return isSameTree(p.left, q.left) and isSameTree(p.right, q.right)
```

### Complexity

- **Time:** $O(n)$ — `n` the size of the smaller tree (mismatch ends early).
- **Space:** $O(h)$ recursion.

## Approach 2: Iterative DFS

### Intuition

The recursion with an explicit stack of node pairs.

### Algorithm

1. Stack containing `(p, q)`.
2. While non-empty: pop `(a, b)`. If both `None`, continue. If one `None` or `a.val != b.val`, return `false`. Push `(a.left, b.left)` and `(a.right, b.right)`.
3. Return `true`.

```python
def isSameTree(p, q) -> bool:
    stack = [(p, q)]
    while stack:
        a, b = stack.pop()
        if not a and not b:
            continue
        if not a or not b or a.val != b.val:
            return False
        stack.append((a.left, b.left))
        stack.append((a.right, b.right))
    return True
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Approach 3: Breadth First Search

### Intuition

Level-order walk of both trees with two queues (or one queue of pairs), comparing dequeued nodes.

### Algorithm

1. Queue of `(p, q)` pairs.
2. Dequeue a pair: if both `None`, continue; if one `None` or values differ, return `false`; enqueue `(a.left, b.left)` and `(a.right, b.right)`.
3. Return `true`.

```python
from collections import deque

def isSameTree(p, q) -> bool:
    dq = deque([(p, q)])
    while dq:
        a, b = dq.popleft()
        if not a and not b:
            continue
        if not a or not b or a.val != b.val:
            return False
        dq.append((a.left, b.left))
        dq.append((a.right, b.right))
    return True
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **Checking `p.val != q.val` before the null checks.** If one node is `None`, `.val` throws. Order the conditions: both-null, then either-null, then value.
- **Comparing serialised forms without null markers.** `[1,2]` and `[1,null,2]` serialise to the same preorder value list unless you emit a marker for missing children.
- **Forgetting one subtree.** Both `left` and `right` recursions must succeed.

## The keystone

Parallel traversal — advancing two structures in lockstep and asserting a relation at each step — is the base technique for tree comparison. [Subtree of Another Tree](/citadel/dsa/subtree-of-another-tree) calls this exact check at every node of the larger tree.
