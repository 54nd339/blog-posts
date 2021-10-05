---
title: Subtree of Another Tree - Same-Tree at Every Anchor
description: Whether one tree appears as a subtree of another — try a full same-tree match at each node of the big tree, or serialise both and run linear string matching.
date: 2021-10-05
draft: false
slug: /dsa/subtree-of-another-tree
tags:
  - Competitive Programming
  - Trees
  - String Matching
---

`subRoot` is a subtree of `root` if some node of `root`, taken with all its descendants, is identical to `subRoot`. The direct approach runs the [Same Tree](/citadel/dsa/same-tree) check anchored at every node of `root`; the linear approach turns both trees into strings and searches one inside the other.

## Description

Given the roots of two binary trees `root` and `subRoot`, return `true` if `subRoot` is a subtree of `root` (a node of `root` plus all its descendants equals `subRoot`). The whole tree counts as a subtree of itself.

**Example**

```
Input:  root = [3,4,5,1,2], subRoot = [4,1,2]
Output: true

Input:  root = [3,4,5,1,2,null,null,null,null,0], subRoot = [4,1,2]
Output: false
```

**Constraints**

- `root` has `1` to `2000` nodes; `subRoot` has `1` to `1000`.
- $-10^4 \le \text{Node.val} \le 10^4$

## Prerequisites

- [Same Tree](/citadel/dsa/same-tree) — the exact-match check.
- Preorder serialisation with null markers; linear string search (KMP or the Z-function).

## Approach 1: Depth First Search

### Intuition

At each node of `root`, ask "is the subtree rooted here identical to `subRoot`?" If any answer is yes, done.

### Algorithm

1. If `subRoot` is `None`, return `true`. If `root` is `None`, return `false`.
2. If `sameTree(root, subRoot)`, return `true`.
3. Otherwise return `isSubtree(root.left, subRoot)` **or** `isSubtree(root.right, subRoot)`.
4. `sameTree(a, b)`: both null → `true`; one null or values differ → `false`; else both subtrees equal.

```python
def isSubtree(root, subRoot) -> bool:
    def same(a, b):
        if not a and not b:
            return True
        if not a or not b or a.val != b.val:
            return False
        return same(a.left, b.left) and same(a.right, b.right)

    if not subRoot:
        return True
    if not root:
        return False
    if same(root, subRoot):
        return True
    return isSubtree(root.left, subRoot) or isSubtree(root.right, subRoot)
```

### Complexity

- **Time:** $O(m \cdot n)$ — `n` nodes in `root`, each an $O(m)$ match against `subRoot`.
- **Space:** $O(m + n)$ recursion.

## Approach 2: Serialization and Pattern Matching

### Intuition

Serialise each tree with a preorder walk that emits an explicit marker for every null. Then `subRoot` is a subtree of `root` iff `subRoot`'s serialization appears as a contiguous substring of `root`'s. The null markers and a delimiter before each value stop `12` from matching inside `123` and force structural alignment.

### Algorithm

1. `serialize(node)`: `"^" + str(node.val) + serialize(left) + serialize(right)` for a node, `"$"` for `None` (the `^` prefix prevents value-prefix collisions).
2. Compute `s = serialize(root)` and `t = serialize(subRoot)`.
3. Return whether `t` is a substring of `s`, found with a linear matcher (`in` is acceptable; KMP / Z-function guarantees $O(m + n)$).

```python
def isSubtree(root, subRoot) -> bool:
    def serialize(node):
        if not node:
            return "$"
        return f"^{node.val}" + serialize(node.left) + serialize(node.right)

    s = serialize(root)
    t = serialize(subRoot)

    # Z-function substring search: is t inside s?
    combined = t + "\x00" + s
    z = [0] * len(combined)
    l = r = 0
    for i in range(1, len(combined)):
        if i < r:
            z[i] = min(r - i, z[i - l])
        while i + z[i] < len(combined) and combined[z[i]] == combined[i + z[i]]:
            z[i] += 1
        if i + z[i] > r:
            l, r = i, i + z[i]
        if z[i] >= len(t):
            return True
    return False
```

### Complexity

- **Time:** $O(m + n)$ — serialisation plus linear matching.
- **Space:** $O(m + n)$ for the strings and the Z-array.

## Common Pitfalls

- **Serialising without null markers.** `[1,2]` and `[1,null,2]` both preorder to `1 2`; the `$` markers make them `^1^2$$$` vs `^1$^2$$`, which differ.
- **No value delimiter.** Without the `^` (or a comma), `subRoot` value `2` matches inside `root` value `12`. Prefix every value.
- **`same(root, subRoot)` before the null checks in Approach 1.** Guard `subRoot`/`root` for `None` first.
- **Empty `subRoot`.** By the usual convention an empty tree is a subtree of anything; return `true` early.

## The keystone

Turning a tree into a canonical string with null markers converts "is this a substructure?" into "is this a substring?", unlocking linear string-matching algorithms. That serialise-then-search idea recurs whenever structural equality can be encoded as sequence equality — it is also the core of [Serialize and Deserialize Binary Tree](/citadel/dsa/serialize-and-deserialize-binary-tree).
