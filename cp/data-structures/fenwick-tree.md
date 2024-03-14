---
title: Fenwick Tree - Prefix Sums With a Binary Indexed Array
description: The smallest structure that does both prefix-sum queries and point updates in log time, how the lowbit trick partitions an index into power-of-two blocks, and the range-update and 2-D extensions.
date: 2024-03-14
draft: false
slug: /cp/fenwick-tree
tags:
  - Competitive Programming
  - Data Structures
  - Range Queries
---

A prefix-sum array answers "sum of the first $i$ elements" in $O(1)$ but needs $O(n)$ to absorb a single update. A Fenwick tree (binary indexed tree) does both in $O(\log n)$, in about six lines and one array — the go-to when the operation is addition and you want the least code.

## The problem

Maintain an array $a[1 \dots n]$ under:

- `update(i, delta)` — add `delta` to $a[i]$,
- `query(i)` — return $a[1] + a[2] + \dots + a[i]$ (a range sum $[l, r]$ is `query(r) - query(l-1)`).

Example: after `update(3, 5)` and `update(7, 2)` on a zero array, `query(7) = 7`, `query(2) = 0`.

## The idea

Store the tree in an array `tree[1 \dots n]` where `tree[i]` holds the sum of the block of $a$ ending at index $i$ and spanning `lowbit(i)` elements, with $\text{lowbit}(i) = i \mathbin{\&} (-i)$ — the value of the least significant set bit of $i$.

- **Query $[1, i]$:** add `tree[i]`, then drop the low bit (`i -= i & -i`), repeat until $i = 0$. Each step jumps to the block covering the next lower chunk; the index loses one set bit per step, so $O(\log n)$ steps.
- **Update $i$:** add `delta` to `tree[i]`, then `i += i & -i` (move to the next block that contains position $i$), repeat until $i > n$.

The two loops walk the set bits of $i$ in opposite directions.

## How it works

$n = 8$. `tree[4]` covers $a[1..4]$ (lowbit $4$), `tree[6]` covers $a[5..6]$, `tree[7]` covers $a[7]$, `tree[8]` covers $a[1..8]$.

`query(7)`: `tree[7]` ($a[7]$), then $7 - 1 = 6$: `tree[6]` ($a[5..6]$), then $6 - 2 = 4$: `tree[4]` ($a[1..4]$), then $4 - 4 = 0$: stop. Total = $a[1..7]$. Three blocks.

`update(3, 5)`: `tree[3]` += 5, then $3 + 1 = 4$: `tree[4]` += 5, then $4 + 4 = 8$: `tree[8]` += 5, then $8 + 8 = 16 > 8$: stop.

## Algorithm

```python
class Fenwick:
    def __init__(self, n: int):
        self.n = n
        self.tree = [0] * (n + 1)               # 1-indexed

    def update(self, i: int, delta: int) -> None:
        while i <= self.n:
            self.tree[i] += delta
            i += i & -i

    def query(self, i: int) -> int:             # sum of a[1..i]
        s = 0
        while i > 0:
            s += self.tree[i]
            i -= i & -i
        return s

    def range_sum(self, l: int, r: int) -> int:
        return self.query(r) - self.query(l - 1)
```

Building from an array in $O(n)$: set `tree[i] = a[i]`, then for each $i$ add `tree[i]` into `tree[i + (i & -i)]` if in range.

## Complexity

- **Time:** $O(\log n)$ per update and per query; $O(n)$ to build.
- **Space:** $O(n)$ — one array, no child pointers, tiny constant. Typically 2–3x faster than a segment tree for plain sums.

## Variations

- **Range update, point query.** Keep a Fenwick over the *difference* array: `update(l, +d)`, `update(r+1, -d)`; then `query(i)` gives $a[i]$.
- **Range update, range query.** Two Fenwicks $B_1, B_2$: to add $d$ on $[l, r]$ do $B_1$: $+d$ at $l$, $-d$ at $r{+}1$; $B_2$: $+d(l-1)$ at $l$, $-d\,r$ at $r{+}1$. Then prefix sum $= B_1.\text{query}(i)\cdot i - B_2.\text{query}(i)$.
- **2-D Fenwick.** Nest the two loops for point-update / prefix-rectangle-sum on a grid in $O(\log n \log m)$.
- **Find-by-prefix (binary lifting on the tree).** Locate the smallest $i$ with `query(i) >= k` in $O(\log n)$ by walking bits from high to low — an order-statistics / k-th-element query when the array holds counts.
- **Non-group operations.** Fenwick needs an *invertible* operation for range queries (sum yes, xor yes, min no). For min/max use a segment tree, or a Fenwick that only supports prefix-min with point *decreases*.

## Common pitfalls

- **0-indexing.** The `i & -i` walk needs 1-based indices; index $0$ makes `update` loop forever. Shift the input array by one.
- **`query` on a min Fenwick.** A max/min Fenwick can answer prefix-max but not arbitrary range-max (no inverse). Do not `query(r) - query(l-1)` it.
- **Size for coordinate compression.** If indices are compressed values, size the tree to the number of distinct coordinates, not the raw value range.
- **Overflow.** Sums of $10^5$ values up to $10^9$ exceed 32 bits — use 64-bit (a non-issue in Python).
- **Off-by-one in range update.** The second point is `r + 1`; forgetting the `+1` leaves the right endpoint unupdated.

## The keystone

A Fenwick tree slices index $i$ into `lowbit`-sized blocks, so a prefix query removes one set bit per step and an update adds one — both $O(\log n)$, in one array with almost no constant factor. When the operation is addition (or any group op) and you want a range structure written in seconds, this is it; anything non-invertible or with range assignment wants a [segment tree](/citadel/data-structures/segment-tree).
