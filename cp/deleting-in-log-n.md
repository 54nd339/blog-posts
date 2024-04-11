---
title: Deleting From a Data Structure in O(T(n) log n)
description: A generic recipe that adds deletion to any structure that only supports insertion and rollback — a segment tree over the timeline, each element living in the O(log n) nodes covering its alive interval.
date: 2024-04-11
draft: false
slug: /cp/deleting-in-log-n
tags:
  - Competitive Programming
  - Data Structures
  - Amortized Analysis
---

Many structures add easily but delete painfully: [DSU](/citadel/cp/disjoint-set-union) has no un-union, a convex hull of lines has no un-insert, a knapsack DP has no un-add-item. If the structure supports **insert** and **rollback to a checkpoint**, there is a generic offline trick that gives you deletion at a cost of one extra $\log n$ factor.

## The problem

You have $q$ operations over time: `add(x)`, `remove(x)` (of a previously added $x$), and `query()` asking something about the current multiset. The structure can `insert` and `undo` its last few inserts, but has no direct `delete`. Answer every `query` offline.

Example: maintain the number of connected components of a graph as edges are **added and removed**, answering after each operation. DSU adds edges fine; the removals are the hard part.

## The idea

Each element $x$ is "alive" during a set of time intervals — from its `add` to its matching `remove` (or to the end). Build a **segment tree over the time axis** $[0, q)$. For each alive interval $[l, r]$, attach $x$ to the $O(\log q)$ canonical nodes that exactly cover $[l, r]$ — the same decomposition a segment tree uses for a range update.

Then **DFS the segment tree**:

- entering a node, `insert` every element stored on it, pushing an undo record;
- at a leaf (a single time $t$), the structure contains exactly the elements alive at $t$ — answer `query()` for operation $t$;
- leaving a node, `rollback` the inserts you did on entry.

Each element is inserted $O(\log q)$ times total (once per canonical node), so the whole sweep costs $O(q \log q \cdot T(n))$, where $T(n)$ is the cost of one `insert` + `undo`.

## How it works

Operations: `add A` at $t{=}0$, `add B` at $t{=}1$, `remove A` at $t{=}3$, queries at every $t$. `A` is alive on $[0, 3)$, `B` on $[1, 4)$ (say $q = 4$).

Time-axis segment tree over $[0, 4)$. `A`'s interval $[0, 3)$ decomposes into nodes covering $[0,2)$ and $[2,3)$. `B`'s $[1, 4)$ into $[1,2)$ and $[2,4)$.

DFS: at the root, nothing. Descend to $[0,2)$: insert `A`. Descend to leaf $0$: contents `{A}`, answer query 0. Leaf $1$: still in $[0,2)$ plus node $[1,2)$ carries `B` → contents `{A, B}`, answer query 1. Back up, undo down to `{}`. Node $[2,4)$ carries `B`; node $[2,3)$ carries `A`. Leaf $2$: `{B, A}`. Leaf $3$: only `[2,4)` → `{B}` (A removed at $t=3$). Undo everything.

## Algorithm

```python
def offline_dynamic(q: int, events, ds):
    """
    events: list of (l, r, item) — item alive on time interval [l, r)
    ds: object with .add(item) -> undo_token, .rollback_to(size), .snapshot()->size, .query()
    answers[t] filled for t in [0, q)
    """
    tree = [[] for _ in range(4 * q)]

    def add_interval(node, lo, hi, l, r, item):
        if r <= lo or hi <= l:
            return
        if l <= lo and hi <= r:
            tree[node].append(item)
            return
        mid = (lo + hi) // 2
        add_interval(2 * node, lo, mid, l, r, item)
        add_interval(2 * node + 1, mid, hi, l, r, item)

    for l, r, item in events:
        add_interval(1, 0, q, l, r, item)

    answers = [None] * q

    def dfs(node, lo, hi):
        mark = ds.snapshot()
        for item in tree[node]:
            ds.add(item)
        if hi - lo == 1:
            answers[lo] = ds.query()
        else:
            mid = (lo + hi) // 2
            dfs(2 * node, lo, mid)
            dfs(2 * node + 1, mid, hi)
        ds.rollback_to(mark)

    dfs(1, 0, q)
    return answers
```

The `ds` for dynamic connectivity is a **DSU with rollback** (union by size, no path compression): `add` = union, `snapshot` = current stack length, `rollback_to` = pop parent/size changes until the stack is that length.

## Complexity

- **Time:** $O(q \log q \cdot T(n))$, where $T(n)$ is the cost of one insert plus its undo. For rollback DSU, $T(n) = O(\log n)$, so dynamic connectivity is $O(q \log q \log n)$.
- **Space:** $O(q \log q)$ for the element lists across the tree, plus the undo stack.

## Common pitfalls

- **Path compression in a rollback DSU.** It makes changes you cannot cheaply undo. Use union-by-size only; the $O(\log n)$ `find` is the price.
- **Unmatched add/remove.** An element added but never removed is alive on $[l, q)$. An element removed that was never added is a bug in the event stream — track a map from item to its open start time.
- **`rollback_to` must be exact.** Undo *precisely* the inserts done on entering this node, no more, no less. Snapshot the undo-stack size, restore to it.
- **Online requirement.** This trick is inherently offline — you need all operations up front. For online dynamic connectivity you need link-cut trees or Euler-tour trees.
- **Query at internal nodes.** Only leaves correspond to a real time; do not answer at internal nodes.

## The keystone

Put time on a segment tree, hang each element on the $O(\log q)$ nodes covering its alive interval, then DFS: insert on the way down, `query` at the leaves, `rollback` on the way up. Any structure that can insert and undo gains deletion for one extra $\log$ factor — the standard route to offline dynamic connectivity and dynamic DP.
