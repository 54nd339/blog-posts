---
title: Randomized Heap - A Mergeable Priority Queue
description: A binary-tree heap whose merge recursion picks a child by coin flip, giving expected O(log n) for merge, push, and pop with no balancing metadata — the simplest mergeable heap to implement.
date: 2024-03-17
draft: false
slug: /cp/randomized-heap
tags:
  - Competitive Programming
  - Data Structures
  - Heaps
---

A binary heap in an array does `push` and `pop` in $O(\log n)$ but cannot merge two heaps faster than $O(n)$. Leftist heaps and skew heaps fix that with structural rules you have to maintain. A randomized meldable heap does it with a coin flip: at each step of the merge, recurse into a **random** child. No ranks, no balancing — expected $O(\log n)$ for everything.

## The problem

Support a priority queue with:

- `push(x)`, `pop_min()` / `top()`,
- **`merge(h1, h2)`** — combine two heaps into one,

all in $O(\log n)$ expected. `push` is `merge` with a singleton; `pop_min` is `merge` of the root's two children.

## The idea

Nodes form a binary tree with the heap property (parent $\le$ children). `merge(a, b)`:

1. If either is empty, return the other.
2. Let `a` be the smaller root (swap if not). `a` stays the root.
3. Randomly pick one of `a`'s two children and replace it with `merge(that child, b)`.

The random choice keeps the expected length of any root-to-leaf path that the recursion follows at $O(\log n)$: each level, the recursion goes into a uniformly random subtree, so the expected number of steps to reach a `None` is logarithmic in the total size. No node stores anything but its value and two child pointers.

## How it works

`merge(A, B)` where `A` has root $2$, `B` has root $5$. Root $2 \le 5$, so $2$ stays on top. Flip a coin → "right". Recurse `merge(A.right, B)`: say `A.right` has root $9$; now $5 \le 9$, so $5$ becomes this subtree's root, flip again, and $9$ (with its subtree) merges into one of $5$'s child slots. The result hangs off $2$'s right. Heap order holds at every step because we always keep the smaller root.

## Algorithm

```python
import random

class Node:
    __slots__ = ("val", "l", "r")
    def __init__(self, val):
        self.val = val
        self.l = self.r = None

def merge(a: "Node | None", b: "Node | None") -> "Node | None":
    if a is None:
        return b
    if b is None:
        return a
    if a.val > b.val:
        a, b = b, a                       # a is the smaller root
    if random.getrandbits(1):
        a.l = merge(a.l, b)
    else:
        a.r = merge(a.r, b)
    return a

class RandomHeap:
    def __init__(self):
        self.root = None
        self.n = 0

    def push(self, x: int) -> None:
        self.root = merge(self.root, Node(x))
        self.n += 1

    def top(self) -> int:
        return self.root.val

    def pop_min(self) -> int:
        v = self.root.val
        self.root = merge(self.root.l, self.root.r)
        self.n -= 1
        return v

    def absorb(self, other: "RandomHeap") -> None:
        self.root = merge(self.root, other.root)
        self.n += other.n
        other.root, other.n = None, 0
```

## Complexity

- **`merge`, `push`, `pop_min`:** $O(\log n)$ expected; the randomness is internal, so there is no bad input, only a bad run of coin flips (probability $n^{-c}$ of exceeding $c\log n$ depth).
- **Space:** $O(n)$, two pointers per node, no rank/rank-difference field.

## When to use it

- **Mergeable priority queue** — the textbook case is Dijkstra-with-decrease-key replaced by "insert duplicates", or union-of-heaps in a small-to-large merge over a tree.
- **Kinetic / offline problems** that repeatedly split off "everything below a threshold" — pop while `top() < x`.
- As a simpler stand-in for a **leftist or pairing heap** when you do not need the worst-case (only expected) bound.

If you never merge, a plain array binary heap (or `heapq`) is faster by a constant factor and simpler still.

## Common pitfalls

- **Recursion depth.** Expected $O(\log n)$, but the recursion is real; for $n \approx 10^6$ raise `sys.setrecursionlimit` or write `merge` iteratively along the chosen path.
- **Reusing a merged heap.** After `absorb`, the donor's root must be cleared — otherwise two heaps share nodes and the next `merge` corrupts both.
- **`pop_min` on an empty heap.** `self.root` is `None`; guard before dereferencing.
- **Weak RNG.** `random.getrandbits(1)` is fine; a low-period or predictable bit source can be adversarially defeated in theory, though rarely in contest practice.
- **Expecting worst-case bounds.** If a problem needs a guaranteed $O(\log n)$ per op (not expected), use a leftist heap instead.

## The keystone

Meld two heaps by keeping the smaller root and recursing into a coin-flipped child; that single random choice keeps every path the merge touches $O(\log n)$ long in expectation, so `push`, `pop`, and `merge` are all logarithmic with zero balancing bookkeeping — the least code for a mergeable priority queue.
