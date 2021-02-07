---
title: Segment Trees - Range Queries and Point Updates in Log Time
description: Answering "sum of elements 3 to 7" by looping is O(n) per query, and a prefix-sum array makes queries O(1) but updates O(n). A segment tree does both the query and the update in O(log n) by storing a partial aggregate for every power-of-two-aligned interval.
date: 2021-02-07
draft: false
slug: /data-structures/segment-tree
tags:
  - Data Structures
  - Trees
  - Range Queries
---

You have an array, and you keep asking two kinds of question: *what's the sum of elements $L$ through $R$*, and *set element $i$ to a new value*. Looping over the range answers a query in $O(n)$. A prefix-sum array answers it in $O(1)$ — but then a single update forces you to rebuild every prefix after $i$, which is $O(n)$.

A **segment tree** balances the two: both the range query and the point update run in $O(\log n)$. It works by precomputing the aggregate (sum, min, max — anything associative) for a set of nested intervals and combining just a few of them per query.

## The structure

A segment tree over an array of $n$ elements is a near-complete binary tree where every node owns an interval:

- The **root** owns the whole array, $[0, n-1]$.
- An **internal node** owning $[l, r]$ splits it at $m = \lfloor (l+r)/2 \rfloor$: its left child owns $[l, m]$, its right child owns $[m+1, r]$. The node stores the aggregate of its interval — for a sum tree, the sum of its two children.
- A **leaf** owns a single index $[i, i]$ and stores that element.

The height is $O(\log n)$. A pointer-based tree has about $2n - 1$ nodes; an array-based one is usually allocated $2n$ (for a compact build) or $4n$ (for a recursive build) slots.

Why $O(\log n)$ per query: any range $[L, R]$ can be covered by at most $O(\log n)$ of these precomputed intervals — a node whose interval sits entirely inside $[L, R]$ contributes its stored aggregate and stops the recursion there; the split only continues where the range boundary cuts through a node.

## A walkthrough

Take the array `[2, 1, 5, 3]`. The sum tree:

```
                 [0,3]=11
              /            \
        [0,1]=3            [2,3]=8
        /    \             /    \
   [0,0]=2 [1,1]=1    [2,2]=5 [3,3]=3
```

**Query: sum of indices 1 to 2.** Start at the root `[0,3]` — it overhangs the range, so recurse. `[0,1]` overhangs (index 0 isn't wanted): recurse, take `[1,1]=1`, skip `[0,0]`. `[2,3]` overhangs (index 3 isn't wanted): recurse, take `[2,2]=5`, skip `[3,3]`. Total `1 + 5 = 6`. Four nodes visited, not the whole array.

**Update: set index 2 to 10.** Change leaf `[2,2]` to 10, then recompute every ancestor: `[2,3]` becomes `10 + 3 = 13`, then the root becomes `3 + 13 = 16`. One path from leaf to root — $O(\log n)$ nodes.

## Array-based, iterative

Store the tree in a flat array of size $2n$. Element $i$ of the input lives at index $n + i$. The two children of node $j$ are $2j$ and $2j + 1$; the parent of $j$ is $j / 2$; the sibling of $j$ is $j \oplus 1$. This layout has no pointers and excellent cache behaviour, which is why competitive programmers favour it.

```python
class SegmentTree:
    def __init__(self, nums):
        self.n = len(nums)
        self.tree = [0] * (2 * self.n)
        for i in range(self.n):                       # leaves
            self.tree[self.n + i] = nums[i]
        for i in range(self.n - 1, 0, -1):            # internal nodes, bottom-up
            self.tree[i] = self.tree[2 * i] + self.tree[2 * i + 1]

    def update(self, index, value):
        pos = index + self.n
        self.tree[pos] = value
        while pos > 1:                                # walk to the root
            self.tree[pos // 2] = self.tree[pos] + self.tree[pos ^ 1]
            pos //= 2

    def query(self, left, right):                     # sum of [left, right)
        left += self.n
        right += self.n
        total = 0
        while left < right:
            if left & 1:                              # left is a right child: take it, move inward
                total += self.tree[left]
                left += 1
            if right & 1:                             # right is a right child: take its left sibling
                right -= 1
                total += self.tree[right]
            left //= 2
            right //= 2
        return total


st = SegmentTree([2, 1, 5, 3])
assert st.query(1, 3) == 6        # indices 1 and 2
st.update(2, 10)
assert st.query(0, 4) == 16
```

The `query` here takes a half-open range `[left, right)`. It walks the two boundaries up the tree simultaneously, and whenever a boundary index is a *right* child, that node's interval is fully inside the range on that side, so it's added and the boundary moves.

## Pointer-based, recursive

More verbose, but each node explicitly carries its interval, which makes it easier to adapt to min, max, or lazy propagation.

```python
class Node:
    def __init__(self, start, end):
        self.start, self.end = start, end
        self.total = 0
        self.left = self.right = None

class SegmentTree:
    def __init__(self, nums):
        self.root = self._build(nums, 0, len(nums) - 1) if nums else None

    def _build(self, nums, lo, hi):
        node = Node(lo, hi)
        if lo == hi:
            node.total = nums[lo]
        else:
            mid = (lo + hi) // 2
            node.left = self._build(nums, lo, mid)
            node.right = self._build(nums, mid + 1, hi)
            node.total = node.left.total + node.right.total
        return node

    def update(self, index, value):
        self._update(self.root, index, value)

    def _update(self, node, i, val):
        if node.start == node.end:
            node.total = val
            return
        mid = (node.start + node.end) // 2
        self._update(node.left if i <= mid else node.right, i, val)
        node.total = node.left.total + node.right.total

    def query(self, lo, hi):                          # inclusive [lo, hi]
        return self._query(self.root, lo, hi)

    def _query(self, node, lo, hi):
        if node is None or hi < node.start or lo > node.end:
            return 0                                  # disjoint: contributes nothing
        if lo <= node.start and node.end <= hi:
            return node.total                         # fully covered: use the stored aggregate
        return self._query(node.left, lo, hi) + self._query(node.right, lo, hi)
```

Both versions were checked against a brute-force `sum(arr[l:r+1])` over thousands of random arrays and interleaved updates.

## Cost

| Operation | Time | Why |
| --- | --- | --- |
| Build | $O(n)$ | each of the $\sim 2n$ nodes is computed once |
| Point update | $O(\log n)$ | only the leaf-to-root path changes |
| Range query | $O(\log n)$ | the range splits into $O(\log n)$ covered intervals |

Space is $O(n)$.

## Lazy propagation: range updates

Point updates walk one leaf-to-root path. A *range* update — "add 5 to every element in $[L, R]$" — would touch $O(n)$ leaves if done naively. **Lazy propagation** keeps it $O(\log n)$: when an update fully covers a node's interval, apply it to that node's aggregate and stash a *pending* amount in a `lazy` array instead of recursing. The pending value is pushed down to the children only when a later operation actually needs to descend past that node.

Two helpers do the work: `_apply` folds a pending amount into one node (for range-add on a sum tree, that is `+= (interval length) * add`), and `_push` moves a node's pending amount onto its two children and clears it.

```python
class LazySegmentTree:
    def __init__(self, nums):
        self.n = len(nums)
        self.tree = [0] * (4 * self.n)
        self.lazy = [0] * (4 * self.n)
        self._build(nums, 1, 0, self.n - 1)

    def _build(self, nums, node, lo, hi):
        if lo == hi:
            self.tree[node] = nums[lo]
            return
        mid = (lo + hi) // 2
        self._build(nums, 2 * node, lo, mid)
        self._build(nums, 2 * node + 1, mid + 1, hi)
        self.tree[node] = self.tree[2 * node] + self.tree[2 * node + 1]

    def _apply(self, node, lo, hi, add):
        self.tree[node] += (hi - lo + 1) * add       # aggregate for a full interval
        self.lazy[node] += add                       # remember it for the children

    def _push(self, node, lo, hi):
        if self.lazy[node]:
            mid = (lo + hi) // 2
            self._apply(2 * node, lo, mid, self.lazy[node])
            self._apply(2 * node + 1, mid + 1, hi, self.lazy[node])
            self.lazy[node] = 0

    def update(self, l, r, add, node=1, lo=0, hi=None):   # add 'add' to [l, r]
        if hi is None:
            hi = self.n - 1
        if r < lo or hi < l:
            return
        if l <= lo and hi <= r:
            self._apply(node, lo, hi, add)
            return
        self._push(node, lo, hi)
        mid = (lo + hi) // 2
        self.update(l, r, add, 2 * node, lo, mid)
        self.update(l, r, add, 2 * node + 1, mid + 1, hi)
        self.tree[node] = self.tree[2 * node] + self.tree[2 * node + 1]

    def query(self, l, r, node=1, lo=0, hi=None):         # sum of [l, r]
        if hi is None:
            hi = self.n - 1
        if r < lo or hi < l:
            return 0
        if l <= lo and hi <= r:
            return self.tree[node]
        self._push(node, lo, hi)
        mid = (lo + hi) // 2
        return (self.query(l, r, 2 * node, lo, mid)
                + self.query(l, r, 2 * node + 1, mid + 1, hi))
```

For "*assign* a value to a range" instead of "add", the lazy entry becomes an optional set-value and `_apply` overwrites rather than accumulates; combining an add-lazy and an assign-lazy on the same tree needs a defined order (assign clears any pending add). This version was checked against a brute `arr[l:r+1]` with interleaved range-adds over thousands of random cases.

## Other variants

- **Non-commutative combine.** The iterative `query` adds nodes in a mixed order, which is fine for sum/min/max/gcd but wrong when `op` is order-sensitive (matrix product, "assign then add" affine maps, hash of a substring). Use the recursive form and keep separate left- and right-side accumulators, combining them in index order at the end.
- **Persistent segment tree.** Instead of mutating nodes, each update creates $O(\log n)$ new nodes and returns a new root, keeping every past version alive. Enables "query the array as it was at version $t$" and, via a version per array prefix, offline range-$k$-th-smallest.
- **Merge-sort tree.** Store at each node the *sorted* list of its interval's elements. Answers "how many elements in $[L, R]$ are $\le x$" in $O(\log^2 n)$, at $O(n \log n)$ space.
- **2D segment tree** (a segment tree of segment trees) for range queries over a grid in $O(\log^2 n)$, and **implicit / dynamic** trees that allocate nodes only where touched, for coordinate ranges up to $10^{18}$.

## Uses

Segment trees show up in range-sum and range-min problems, maximum-subarray variants, computational geometry with interval sweeps, some dynamic-programming speedups, and time-series systems that aggregate over sliding windows. When the array never changes, a plain prefix-sum array (or a [sparse table](/citadel/cp/sparse-table) for idempotent range-min) is simpler and faster; the segment tree earns its complexity precisely when updates and queries are interleaved.

## The one idea to keep

The segment tree beats both naive alternatives because any range $[L, R]$ decomposes into at most $O(\log n)$ pre-aggregated intervals, and any single element sits on just one leaf-to-root path — so query and update are both $O(\log n)$ where a loop is $O(n)$ on queries and a prefix-sum array is $O(n)$ on updates. Everything else — lazy propagation for range updates, persistence, merge-sort trees — is the same interval-decomposition idea carrying extra state per node. If the array is static, you do not need any of it; a prefix-sum array wins.
