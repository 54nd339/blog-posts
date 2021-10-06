---
title: Lowest Common Ancestor of a BST - Walk Until the Paths Split
description: The lowest common ancestor of two nodes in a binary search tree — follow the ordering downward until one target is on each side, and that node is the answer.
date: 2021-10-06
draft: false
slug: /dsa/lowest-common-ancestor-of-a-bst
tags:
  - Competitive Programming
  - Trees
  - Binary Search Tree
---

In a BST, the lowest common ancestor of `p` and `q` is the first node on the way down where `p` and `q` stop agreeing on direction — the split point. Comparing both values to the current node tells you where to go.

## Description

Given a binary search tree and two nodes `p` and `q` in it, return their lowest common ancestor — the deepest node that has both as descendants (a node is a descendant of itself).

**Example**

```
Input:  root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 8
Output: 6

Input:  root = [6,2,8,0,4,7,9,null,null,3,5], p = 2, q = 4
Output: 2
```

**Constraints**

- `2` to $10^5$ nodes; all values unique; `p != q`; both exist in the tree.

## Prerequisites

- The BST ordering invariant: left subtree `<` node `<` right subtree.

## Approach 1: Recursion

### Intuition

If both targets are smaller than the current node, the LCA is in the left subtree; if both are larger, it is in the right; otherwise the current node is the split point (or is one of the targets), so it is the LCA.

### Algorithm

1. If `p.val < node.val` and `q.val < node.val`, recurse left.
2. If `p.val > node.val` and `q.val > node.val`, recurse right.
3. Otherwise return `node`.

```python
def lowestCommonAncestor(root, p, q):
    if p.val < root.val and q.val < root.val:
        return lowestCommonAncestor(root.left, p, q)
    if p.val > root.val and q.val > root.val:
        return lowestCommonAncestor(root.right, p, q)
    return root
```

### Complexity

- **Time:** $O(h)$ — one step per level.
- **Space:** $O(h)$ recursion.

## Approach 2: Iteration

### Intuition

The recursion is tail-recursive, so replace it with a loop and $O(1)$ space.

### Algorithm

1. `node = root`.
2. While `node`: if both values `< node.val`, `node = node.left`; if both `> node.val`, `node = node.right`; else return `node`.

```python
def lowestCommonAncestor(root, p, q):
    node = root
    while node:
        if p.val < node.val and q.val < node.val:
            node = node.left
        elif p.val > node.val and q.val > node.val:
            node = node.right
        else:
            return node
```

### Complexity

- **Time:** $O(h)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Using `<=` / `>=`.** Values are unique and a node is its own descendant, so when `node.val` equals `p.val` or `q.val` you must *stop* — that node is an ancestor of the other target. Strict `<` and `>` handle this by falling through to `return node`.
- **Ignoring the BST property and writing the general-tree LCA.** That works but is $O(n)$; the ordering gives $O(h)$.
- **Assuming `p` is smaller than `q`.** The double comparison (`both <` / `both >`) is symmetric, so order does not matter.

## The keystone

A BST lets you make an $O(1)$ routing decision at each node from a comparison, turning search into a single root-to-node walk. The "descend until the two paths diverge" idea is the LCA special case; the same downward-comparison drive powers [Validate Binary Search Tree](/citadel/dsa/validate-binary-search-tree) and [Kth Smallest Element in a BST](/citadel/dsa/kth-smallest-element-in-a-bst).
