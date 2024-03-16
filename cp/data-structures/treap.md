---
title: Treap - A BST That Balances Itself by Random Priority
description: A binary search tree ordered by key and heap-ordered by a random priority, kept balanced by split and merge, plus the implicit treap that turns an array into a sequence with O(log n) range operations.
date: 2024-03-16
draft: false
slug: /cp/treap
tags:
  - Competitive Programming
  - Data Structures
  - Balanced Trees
---

A treap is a binary search tree on the keys and, simultaneously, a heap on a **random** priority assigned to each node. Those two orders together pin down the tree shape uniquely, and since the priorities are random the tree is balanced with high probability — expected height $O(\log n)$ — without a single rotation-balancing case to memorise. Two primitives, `split` and `merge`, build everything else.

## The problem

Maintain a dynamic ordered set (insert, erase, find, k-th smallest, count-less-than) or a dynamic sequence (insert/erase at a position, reverse a subarray, add on a range) in $O(\log n)$ per operation, with far less code than a red-black tree.

## The idea

Each node has `key` (BST order), `prio` (a random number, heap order: a parent's priority exceeds its children's), plus `size` of its subtree. Given the key set and their priorities, the tree is forced: the global-max-priority node is the root, its key splits the rest into left and right subsets, recurse.

Everything reduces to:

- **`split(t, k)`** → `(L, R)` where `L` has all keys $< k$ (or the first $k$ nodes, for an implicit treap) and `R` the rest. Recurse down one side, splicing subtrees.
- **`merge(L, R)`** → one treap, assuming every key in `L` is $<$ every key in `R`. Whichever root has higher priority becomes the new root; recurse on the appropriate child.

`insert(k)`: `split` at `k`, `merge(L, new_node)`, `merge(that, R)`. `erase(k)`: `split` to isolate `k`, `merge` the two sides. Both $O(\log n)$ expected because `split`/`merge` walk one root-to-leaf path.

## How it works

Insert keys $2, 8, 5$ with priorities $9, 3, 7$. Insert $2$: root $(2, 9)$. Insert $8$: BST-right of $2$; priority $3 < 9$, stays as $2$'s right child. Insert $5$: BST between $2$ and $8$; priority $7 > 3$, so $5$ must sit above $8$ — it becomes $2$'s right child with $8$ hanging off $5$'s right. Result: root $(2,9)$, right $(5,7)$, whose right is $(8,3)$. Heap order holds ($9 > 7 > 3$), BST order holds ($2 < 5 < 8$).

## Algorithm (key treap)

```python
import random

class Node:
    __slots__ = ("key", "prio", "size", "l", "r")
    def __init__(self, key):
        self.key = key
        self.prio = random.random()
        self.size = 1
        self.l = self.r = None

def sz(t): return t.size if t else 0
def upd(t):
    if t:
        t.size = 1 + sz(t.l) + sz(t.r)
    return t

def split(t, key):                       # L: keys < key ; R: keys >= key
    if t is None:
        return None, None
    if t.key < key:
        t.r, R = split(t.r, key)
        return upd(t), R
    else:
        L, t.l = split(t.l, key)
        return L, upd(t)

def merge(L, R):
    if not L or not R:
        return L or R
    if L.prio > R.prio:
        L.r = merge(L.r, R)
        return upd(L)
    else:
        R.l = merge(L, R.l)
        return upd(R)

def insert(t, key):
    L, R = split(t, key)
    return merge(merge(L, Node(key)), R)

def erase(t, key):
    L, R = split(t, key)
    _, R2 = split(R, key + 1)             # integer keys; isolate exactly `key`
    return merge(L, R2)

def kth(t, k):                            # 0-indexed k-th smallest
    while t:
        left = sz(t.l)
        if k == left:
            return t.key
        if k < left:
            t = t.l
        else:
            k -= left + 1
            t = t.r
```

## The implicit treap

Drop `key`; the "key" of a node is its **position**, computed on the fly as `sz(left subtree) + (offsets from ancestors)`. `split(t, k)` now means "first $k$ nodes vs the rest". This gives an array with:

- insert/erase at index $i$ — `split` at $i$, `merge` around the new node;
- **reverse $[l, r]$** — split out that segment, set a lazy `rev` flag on its root (swap children on push-down), merge back;
- range add / range assign / range sum — lazy tags exactly like a [segment tree](/citadel/data-structures/segment-tree), but on a structure you can also splice.

Push the lazy flags down at the top of every `split`/`merge` recursion.

## Complexity

- **All operations:** $O(\log n)$ *expected* (randomised over the priorities, not the input) — height is $O(\log n)$ with probability $1 - n^{-c}$.
- **Space:** $O(n)$, plus recursion depth $O(\log n)$ expected.

## Common pitfalls

- **Non-random or low-entropy priorities.** Duplicated priorities or a weak RNG can degrade the tree. Use 64-bit random values, or `random.random()`; never derive priority from the key.
- **Forgetting `upd` (size fix-up).** Every `split`/`merge` that changes a node's children must recompute `size`, or `kth`/`count` break.
- **Lazy push-down placement.** In an implicit treap, push pending tags *before* you read `sz(t.l)` or recurse; pushing after gives stale positions and wrong reversals.
- **`erase` of an absent key.** `split`/`split` isolates a possibly-empty middle; merging back is still correct, but guard if the problem forbids it.
- **Recursion depth in Python.** Expected depth is small, but set a comfortable `sys.setrecursionlimit` for $n \approx 10^5$.

## The keystone

A treap is "BST by key, heap by random priority", and random priorities make it balanced for free. `split` and `merge` — each an $O(\log n)$ walk down one path — express insert, erase, k-th, and, in the implicit version, an array with $O(\log n)$ range reverse and range updates.
