---
title: Kth Smallest Element in a BST - In-Order Gives Sorted Order
description: The k-th smallest value in a binary search tree — an in-order traversal visits values in ascending order, so stop at the k-th; Morris traversal does it in O(1) space.
date: 2021-10-13
draft: false
slug: /dsa/kth-smallest-element-in-a-bst
tags:
  - Competitive Programming
  - Trees
  - Binary Search Tree
---

An in-order traversal of a BST — left subtree, node, right subtree — produces the values in sorted order. So the k-th smallest is simply the k-th node visited in-order, and you can stop as soon as you reach it.

## Description

Given the `root` of a binary search tree and an integer `k`, return the k-th smallest value (1-indexed).

**Example**

```
Input:  root = [3,1,4,null,2], k = 1
Output: 1

Input:  root = [5,3,6,2,4,null,null,1], k = 3
Output: 3
```

**Constraints**

- `1` to $10^4$ nodes; $1 \le k \le n$.
- $0 \le \text{Node.val} \le 10^4$

## Prerequisites

- In-order traversal of a BST yields ascending values.
- Iterative in-order with an explicit stack; Morris threading (Approach 4).

## Approach 1: Brute Force

### Intuition

Collect all values, sort, index.

### Algorithm

1. DFS to gather every value into a list.
2. Sort it; return `values[k - 1]`.

```python
def kthSmallest(root, k: int) -> int:
    vals = []

    def dfs(node):
        if not node:
            return
        dfs(node.left)
        vals.append(node.val)
        dfs(node.right)

    dfs(root)
    return sorted(vals)[k - 1]
```

### Complexity

- **Time:** $O(n \log n)$.
- **Space:** $O(n)$.

## Approach 2: In-Order Traversal

### Intuition

In-order already visits in sorted order, so no sort is needed — just take element `k - 1` of the in-order list.

### Algorithm

1. In-order DFS appending values.
2. Return `values[k - 1]`.

```python
def kthSmallest(root, k: int) -> int:
    vals = []

    def inorder(node):
        if not node:
            return
        inorder(node.left)
        vals.append(node.val)
        inorder(node.right)

    inorder(root)
    return vals[k - 1]
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ for the list (plus $O(h)$ recursion).

## Approach 3: Recursive DFS with Early Stop

### Intuition

Do not build the whole list. Count nodes as in-order visits them; when the count hits `k`, record the value and unwind without further work.

### Algorithm

1. `count = 0`, `answer = None`.
2. `inorder(node)`: return early if `answer` is set or `node` is `None`. Recurse left; increment `count`; if `count == k`, set `answer = node.val` and return; recurse right.

```python
def kthSmallest(root, k: int) -> int:
    count = 0
    answer = None

    def inorder(node):
        nonlocal count, answer
        if not node or answer is not None:
            return
        inorder(node.left)
        if answer is not None:
            return
        count += 1
        if count == k:
            answer = node.val
            return
        inorder(node.right)

    inorder(root)
    return answer
```

### Complexity

- **Time:** $O(h + k)$ — descend to the smallest, then `k` steps.
- **Space:** $O(h)$ recursion.

## Approach 4: Iterative DFS with a Stack

### Intuition

Explicit in-order: push all left descendants, pop (that is the next-smallest), decrement `k`, then move right.

### Algorithm

1. `stack = []`, `curr = root`.
2. While `stack` or `curr`: push `curr` and go left until `None`; pop a node; `k -= 1`; if `k == 0`, return its value; `curr = node.right`.

```python
def kthSmallest(root, k: int) -> int:
    stack = []
    curr = root
    while stack or curr:
        while curr:
            stack.append(curr)
            curr = curr.left
        curr = stack.pop()
        k -= 1
        if k == 0:
            return curr.val
        curr = curr.right
```

### Complexity

- **Time:** $O(h + k)$.
- **Space:** $O(h)$ for the stack.

## Approach 5: Morris Traversal

### Intuition

In-order with $O(1)$ space by temporarily threading each node's in-order predecessor's right pointer back to it, then undoing the thread.

### Algorithm

1. `curr = root`, `count = 0`.
2. While `curr`:
   - No left child: `count += 1`; if `count == k`, return `curr.val`; `curr = curr.right`.
   - Has left child: find the rightmost node `pred` of `curr.left`. If `pred.right` is `None`, set `pred.right = curr`, `curr = curr.left`. If `pred.right is curr`, unthread (`pred.right = None`), `count += 1`; if `count == k`, return `curr.val`; `curr = curr.right`.

```python
def kthSmallest(root, k: int) -> int:
    curr = root
    count = 0
    while curr:
        if not curr.left:
            count += 1
            if count == k:
                return curr.val
            curr = curr.right
        else:
            pred = curr.left
            while pred.right and pred.right is not curr:
                pred = pred.right
            if not pred.right:
                pred.right = curr
                curr = curr.left
            else:
                pred.right = None
                count += 1
                if count == k:
                    return curr.val
                curr = curr.right
```

### Complexity

- **Time:** $O(n)$ — each edge is traversed a constant number of times.
- **Space:** $O(1)$.

## Common Pitfalls

- **1- vs 0-indexing.** `k` is 1-indexed; the k-th in-order node is `vals[k - 1]`, or you decrement `k` and stop at `0`.
- **Not short-circuiting (Approach 3).** Without the `answer is not None` guards you still traverse the whole tree, losing the $O(h + k)$ benefit.
- **Morris: forgetting to restore pointers.** Every thread you add must be removed, or the tree is left corrupted and later reads loop.
- **Follow-up with frequent modifications.** If the BST changes often and `kthSmallest` is called repeatedly, augment nodes with subtree sizes for $O(h)$ queries.

## The keystone

"In-order traversal of a BST is a sorted stream" converts order-statistic queries into a bounded walk. Whenever a BST question mentions "k-th", "median", or "sorted", reach for in-order first — and the stack-based in-order is the reusable engine.
