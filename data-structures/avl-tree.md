---
title: AVL Trees - The First Self-Balancing Binary Search Tree
description: A plain binary search tree degrades to a linked list when keys arrive in order. AVL trees stop that by keeping every node's two subtrees within one level of each other, restoring the bound with a local rotation after each insert or delete.
date: 2021-01-31
draft: false
slug: /data-structures/avl-tree
tags:
  - Data Structures
  - Balanced Trees
  - Trees
---

A [binary search tree](/citadel/data-structures/trees) gives you $O(\log n)$ search, insert, and delete — as long as it stays bushy. Insert keys in sorted order and it doesn't: each new key hangs off the rightmost node, the tree becomes a linked list, and every operation is back to $O(n)$.

The **AVL tree**, published by Adelson-Velsky and Landis in 1962 and the first self-balancing BST, prevents this. After every insert or delete it checks a local balance condition and, if it's been violated, fixes it with a *rotation* — a constant-time pointer rearrangement. The height stays $O(\log n)$ no matter what order the keys arrive in.

## The balance condition

An AVL tree is a BST with one extra invariant:

> For every node, the heights of its left and right subtrees differ by at most 1.

That difference — `height(left) − height(right)` — is the node's **balance factor**, always one of −1, 0, or +1 in a valid AVL tree. Because the condition holds at every node, and it holds recursively for both subtrees, the height of an AVL tree with $n$ nodes is provably under $1.44 \log_2 n$. That's the guarantee: search, insert, and delete are all $O(\log n)$ worst case, not just on average.

## Rotations

A rotation restructures three nodes locally, preserving the BST ordering while changing which one is on top. There are two, mirror images of each other. A **right rotation** fixes a left-heavy node `y`:

```
      y                 x
     / \               / \
    x   T3    ──▶     T1   y
   / \                    / \
  T1  T2                 T2  T3
```

`x` moves up, `y` moves down to the right, and `x`'s old right subtree `T2` (all keys between `x` and `y`) becomes `y`'s new left subtree. Every ordering constraint still holds. A **left rotation** is the same operation reflected, for a right-heavy node.

When the imbalance zig-zags — a node is left-heavy but its left child leans right — one rotation isn't enough. You rotate the child first to straighten the line, then rotate the node. That's a **double rotation** (left-right or right-left).

## The four cases

After inserting a key, walk back up to the root updating heights. At the first node whose balance factor hits ±2, the shape of the imbalance tells you which rotation:

| Case | Condition | Fix |
| --- | --- | --- |
| Left-Left | `balance > 1` and new key went left of the left child | right-rotate the node |
| Right-Right | `balance < -1` and new key went right of the right child | left-rotate the node |
| Left-Right | `balance > 1` and new key went right of the left child | left-rotate the left child, then right-rotate the node |
| Right-Left | `balance < -1` and new key went left of the right child | right-rotate the right child, then left-rotate the node |

## A walkthrough

Insert **10, 20, 30** into an empty tree:

- `10` → the root.
- `20` → becomes `10`'s right child. Balance factor of `10` is −1. Fine.
- `30` → becomes `20`'s right child. Now `10` has balance factor −2, and `30 > 20` (the right child's key) — the Right-Right case. Left-rotate at `10`: `20` becomes the root with `10` and `30` as children. Height back to 2.

Now insert **30, 10, 20** into a fresh tree, to see a double rotation:

- `30` → the root.
- `10` → `30`'s left child. Balance factor of `30` is +1.
- `20` → `20 < 30`, go left; `20 > 10`, so `20` becomes `10`'s right child. Walking up, `30` has balance factor +2, but `20 > 10` (the left child's key) — the Left-Right case. First left-rotate at `10` (now `20` is `30`'s left child, `10` is `20`'s left child), then right-rotate at `30`. Result: `20` at the root, `10` and `30` as children.

## The code

Nodes carry a cached height so balance factors are $O(1)$ to compute. Insert and delete are recursive: descend as in a normal BST, then rebalance on the way back up.

```python
class Node:
    def __init__(self, key):
        self.key = key
        self.left = None
        self.right = None
        self.height = 1

def height(node):
    return node.height if node else 0

def balance(node):
    return height(node.left) - height(node.right) if node else 0

def update_height(node):
    node.height = 1 + max(height(node.left), height(node.right))

def rotate_right(y):
    x = y.left
    y.left = x.right
    x.right = y
    update_height(y)          # order matters: the lower node first
    update_height(x)
    return x                  # x is the new subtree root

def rotate_left(x):
    y = x.right
    x.right = y.left
    y.left = x
    update_height(x)
    update_height(y)
    return y

def insert(root, key):
    if root is None:
        return Node(key)
    if key < root.key:
        root.left = insert(root.left, key)
    elif key > root.key:
        root.right = insert(root.right, key)
    else:
        return root          # no duplicates

    update_height(root)
    bf = balance(root)
    if bf > 1 and key < root.left.key:          # Left-Left
        return rotate_right(root)
    if bf < -1 and key > root.right.key:        # Right-Right
        return rotate_left(root)
    if bf > 1 and key > root.left.key:          # Left-Right
        root.left = rotate_left(root.left)
        return rotate_right(root)
    if bf < -1 and key < root.right.key:        # Right-Left
        root.right = rotate_right(root.right)
        return rotate_left(root)
    return root

def min_node(node):
    while node.left:
        node = node.left
    return node

def delete(root, key):
    if root is None:
        return None
    if key < root.key:
        root.left = delete(root.left, key)
    elif key > root.key:
        root.right = delete(root.right, key)
    else:
        if root.left is None or root.right is None:
            root = root.left or root.right       # 0 or 1 child
        else:
            succ = min_node(root.right)          # 2 children: take in-order successor
            root.key = succ.key
            root.right = delete(root.right, succ.key)
    if root is None:
        return None

    update_height(root)
    bf = balance(root)
    # after a delete, decide by the child's balance factor, not a new key
    if bf > 1 and balance(root.left) >= 0:
        return rotate_right(root)
    if bf > 1 and balance(root.left) < 0:
        root.left = rotate_left(root.left)
        return rotate_right(root)
    if bf < -1 and balance(root.right) <= 0:
        return rotate_left(root)
    if bf < -1 and balance(root.right) > 0:
        root.right = rotate_right(root.right)
        return rotate_left(root)
    return root

def search(root, key):
    while root and root.key != key:
        root = root.left if key < root.key else root.right
    return root is not None


root = None
for k in [10, 20, 30, 40, 50, 25]:
    root = insert(root, k)
assert search(root, 25) and not search(root, 99)
root = delete(root, 30)
# in-order traversal is always sorted; the tree stays height-balanced throughout
```

Deletion differs from insertion in one spot: after removing a node, the imbalance isn't tied to a "new key" that took a known path, so each case is chosen by the offending child's own balance factor. A single delete can trigger a rotation at every level up to the root — still $O(\log n)$.

## Cost

Search, insert, delete: $O(\log n)$ time, because the height is $O(\log n)$ and the rebalancing on the way back up does $O(1)$ work per level. Space is $O(n)$ for the tree plus $O(\log n)$ for the recursion stack. Each node carries one extra integer for its height.

## AVL vs red-black

AVL's balance condition is strict — subtree heights within 1 — so lookups are as fast as a balanced BST gets. The cost is more rotations during updates. [Red-black trees](/citadel/data-structures/red-black-tree) allow the longest root-to-leaf path to be up to twice the shortest, which permits fewer rotations per insert or delete but slightly taller trees. Read-heavy workload: AVL. Update-heavy, or you're implementing a standard library container: red-black is the usual pick. For data that lives on disk, neither — that's what [B-trees](/citadel/data-structures/b-tree) are for.

## The one idea to keep

A plain BST's $O(\log n)$ is a hope that depends on insertion order; an AVL tree makes it a guarantee by enforcing one local invariant — every node's subtrees within one level — and repairing any violation with a constant-time rotation on the way back up. The height then provably stays under $1.44\log_2 n$. The trade is explicit: the strict invariant means more rotations on writes than a looser scheme like red-black, bought in exchange for the shortest possible trees and the fastest reads. Pick the balance scheme by your read/write ratio, not by habit.
