---
title: Serialize and Deserialize Binary Tree - Null Markers Make It Reversible
description: Encoding a tree as a string and rebuilding it — a preorder walk that writes a marker for every missing child is enough to reconstruct the exact shape; BFS works the same way.
date: 2021-10-24
draft: false
slug: /dsa/serialize-and-deserialize-binary-tree
tags:
  - Competitive Programming
  - Trees
  - Design
---

A plain preorder list of values is ambiguous — many trees share it. Add an explicit token for every `None` child and the traversal becomes a faithful blueprint. Deserialization replays the traversal, consuming one token at a time.

## Description

Design `serialize(root)` returning a string and `deserialize(data)` returning the reconstructed tree, such that `deserialize(serialize(root))` is structurally identical to `root` with the same values.

**Example**

```
Input:  root = [1,2,3,null,null,4,5]
serialize   -> "1,2,N,N,3,4,N,N,5,N,N"   (preorder with null markers)
deserialize -> [1,2,3,null,null,4,5]

Input:  root = []
serialize -> "N"
```

**Constraints**

- `0` to $10^4$ nodes.
- $-1000 \le \text{Node.val} \le 1000$

## Prerequisites

- Preorder and level-order traversal.
- Length- or marker-delimited framing (compare [Encode and Decode Strings](/citadel/dsa/encode-and-decode-strings)).

## Approach 1: Depth First Search

### Intuition

Serialize with preorder, writing the value for a node and a sentinel `N` for each `None`. To deserialize, read tokens left to right: `N` means "no node here"; a number means "make a node, then build its left subtree, then its right".

### Algorithm

1. **serialize:** preorder recursion appends `str(node.val)` or `"N"`; join with commas.
2. **deserialize:** split on commas into a list; keep an index. `build()`: if the current token is `"N"`, advance and return `None`; else create a node with that value, advance, set `node.left = build()`, `node.right = build()`, return `node`.

```python
class Codec:
    def serialize(self, root) -> str:
        out = []

        def dfs(node):
            if not node:
                out.append("N")
                return
            out.append(str(node.val))
            dfs(node.left)
            dfs(node.right)

        dfs(root)
        return ",".join(out)

    def deserialize(self, data: str):
        tokens = iter(data.split(","))

        def build():
            val = next(tokens)
            if val == "N":
                return None
            node = TreeNode(int(val))
            node.left = build()
            node.right = build()
            return node

        return build()
```

### Complexity

- **Time:** $O(n)$ for both directions.
- **Space:** $O(n)$ for the string and $O(h)$ recursion.

## Approach 2: Breadth First Search

### Intuition

Level-order with a marker for every `None` child (including children of leaf nodes). Deserialization walks the token list, attaching two children to each real node from the queue.

### Algorithm

1. **serialize:** if `root` is `None`, return `"N"`. Queue with `root`; while non-empty, pop a node; if it is `None`, append `"N"`; else append its value and enqueue both children (even `None`).
2. **deserialize:** split tokens; if the first is `"N"`, return `None`. Make the root, enqueue it. Walk tokens in pairs: for each dequeued node, the next two tokens are its left and right; create real nodes for non-`"N"` tokens and enqueue them.

```python
from collections import deque

class Codec:
    def serialize(self, root) -> str:
        if not root:
            return "N"
        out = []
        q = deque([root])
        while q:
            node = q.popleft()
            if not node:
                out.append("N")
                continue
            out.append(str(node.val))
            q.append(node.left)
            q.append(node.right)
        return ",".join(out)

    def deserialize(self, data: str):
        tokens = data.split(",")
        if tokens[0] == "N":
            return None
        root = TreeNode(int(tokens[0]))
        q = deque([root])
        i = 1
        while q:
            node = q.popleft()
            if tokens[i] != "N":
                node.left = TreeNode(int(tokens[i]))
                q.append(node.left)
            i += 1
            if tokens[i] != "N":
                node.right = TreeNode(int(tokens[i]))
                q.append(node.right)
            i += 1
        return root
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **Omitting null markers.** Without `N`, the two trees `[1,2]` and `[1,null,2]` serialize identically and deserialization cannot recover the shape.
- **Not handling the empty tree.** `serialize(None)` must produce a token (`"N"`) that `deserialize` recognises.
- **BFS: forgetting to enqueue `None` children on serialize.** The deserializer expects exactly two tokens per real node; skipping null children desynchronises the index.
- **Negative values with a bad delimiter.** Use commas (or spaces); a `-` inside values is fine as long as tokens are split on the delimiter, not parsed character by character.

## The keystone

A reversible serialization needs enough structure in the byte stream to reconstruct shape unambiguously — here, a marker per missing child. That is the tree analogue of length-prefix framing in [Encode and Decode Strings](/citadel/dsa/encode-and-decode-strings): make the encoding self-describing and decoding is a straight replay.
