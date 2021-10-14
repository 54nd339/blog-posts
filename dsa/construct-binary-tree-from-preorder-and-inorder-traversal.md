---
title: Construct Binary Tree from Preorder and Inorder - The Root Splits Inorder
description: Rebuilding a tree from its preorder and inorder sequences — preorder gives the next root, inorder splits into left and right subtrees, and a value-to-index map removes the search.
date: 2021-10-14
draft: false
slug: /dsa/construct-binary-tree-from-preorder-and-inorder-traversal
tags:
  - Competitive Programming
  - Trees
  - Divide and Conquer
---

Preorder is `root, (left subtree), (right subtree)`. Inorder is `(left subtree), root, (right subtree)`. So preorder's first element is the root; finding it in inorder tells you exactly how many nodes are in the left subtree, and recursion does the rest.

## Description

Given two integer arrays `preorder` and `inorder`, the preorder and inorder traversals of a binary tree with unique values, construct and return the tree.

**Example**

```
Input:  preorder = [3,9,20,15,7], inorder = [9,3,15,20,7]
Output: [3,9,20,null,null,15,7]

Input:  preorder = [-1], inorder = [-1]
Output: [-1]
```

**Constraints**

- `1` to `3000` nodes; all values unique; the arrays are consistent traversals of the same tree.

## Prerequisites

- What preorder and inorder traversals look like.
- Divide and conquer with array-slice boundaries; a hash map for $O(1)$ index lookup.

## Approach 1: Depth First Search

### Intuition

Take `preorder[0]` as the root. Locate it in `inorder`; everything to its left is the left subtree, everything to its right is the right subtree. Recurse on the matching preorder segments.

### Algorithm

1. If either array is empty, return `None`.
2. `root = TreeNode(preorder[0])`; `mid = inorder.index(preorder[0])`.
3. `root.left = build(preorder[1:mid+1], inorder[:mid])`.
4. `root.right = build(preorder[mid+1:], inorder[mid+1:])`.

```python
def buildTree(preorder, inorder):
    if not preorder or not inorder:
        return None
    root = TreeNode(preorder[0])
    mid = inorder.index(preorder[0])
    root.left = buildTree(preorder[1:mid + 1], inorder[:mid])
    root.right = buildTree(preorder[mid + 1:], inorder[mid + 1:])
    return root
```

### Complexity

- **Time:** $O(n^2)$ — `inorder.index` is $O(n)$ and the slices copy.
- **Space:** $O(n^2)$ from slicing (plus recursion).

## Approach 2: Hash Map + Depth First Search

### Intuition

Two costs to remove: the linear search for the root in `inorder`, and the slicing. Precompute `value → inorder index`, and pass integer boundaries instead of slices. A single moving pointer walks `preorder`.

### Algorithm

1. `idx = {v: i for i, v in enumerate(inorder)}`; `pre = 0` (index into `preorder`).
2. `dfs(l, r)` over an inorder range: if `l > r`, return `None`.
3. `root = TreeNode(preorder[pre])`; `pre += 1`; `mid = idx[root.val]`.
4. `root.left = dfs(l, mid - 1)`; `root.right = dfs(mid + 1, r)`; return `root`.
5. Call `dfs(0, len(inorder) - 1)`.

```python
def buildTree(preorder, inorder):
    idx = {v: i for i, v in enumerate(inorder)}
    pre = 0

    def dfs(l, r):
        nonlocal pre
        if l > r:
            return None
        root = TreeNode(preorder[pre])
        pre += 1
        mid = idx[root.val]
        root.left = dfs(l, mid - 1)
        root.right = dfs(mid + 1, r)
        return root

    return dfs(0, len(inorder) - 1)
```

### Complexity

- **Time:** $O(n)$ — each node created once, $O(1)$ index lookup.
- **Space:** $O(n)$ for the map and recursion.

## Approach 3: Depth First Search (Optimal, no hash map)

### Intuition

Walk both `preorder` and `inorder` with two pointers. Build nodes from `preorder`; use the current `inorder` value as a *limit* that signals "the current subtree is finished". When `preorder`'s current value equals the limit... actually the check is: build left subtree bounded by the current root's value; when the next `inorder` value equals a passed-in `stop` value, that branch is done and you consume the `inorder` pointer.

### Algorithm

1. Indices `pre = 0`, `ino = 0`.
2. `build(stop)`: if `pre == len(preorder)`, return `None`. If `inorder[ino] == stop`, advance `ino` and return `None`.
3. `root = TreeNode(preorder[pre])`; `pre += 1`.
4. `root.left = build(root.val)`; `root.right = build(stop)`; return `root`.
5. Call `build(None)`.

```python
def buildTree(preorder, inorder):
    pre = ino = 0

    def build(stop):
        nonlocal pre, ino
        if pre == len(preorder):
            return None
        if inorder[ino] == stop:
            ino += 1
            return None
        root = TreeNode(preorder[pre])
        pre += 1
        root.left = build(root.val)
        root.right = build(stop)
        return root

    return build(None)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(h)$ recursion — no hash map.

## Common Pitfalls

- **Preorder segment lengths for the recursion (Approach 1).** The left subtree has `mid` nodes, so its preorder slice is `preorder[1 : mid + 1]`, not `preorder[1 : mid]`.
- **`inorder.index` inside the recursion.** Precompute the map; the repeated linear search is the $O(n^2)$ culprit.
- **Sharing the `pre` pointer correctly (Approaches 2, 3).** It must be a single counter advanced once per node, in preorder — a per-call local resets it and produces a wrong tree.
- **This needs *unique* values.** Duplicate values make the inorder split ambiguous; the problem guarantees uniqueness.

## The keystone

Reconstruction from traversals is divide and conquer keyed on "one traversal names the root, another says where it splits the rest". The same root-splits-the-array idea rebuilds a tree from postorder + inorder (root is postorder's *last* element) and from a preorder with null markers ([Serialize and Deserialize Binary Tree](/citadel/dsa/serialize-and-deserialize-binary-tree)).
