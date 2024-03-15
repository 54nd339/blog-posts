---
title: Sqrt Decomposition - Bucketing an Array for Range Queries
description: Splitting an array into blocks of size root n so a range query touches at most two partial blocks and root n whole ones, plus lazy block tags for range updates and Mo's algorithm for offline queries.
date: 2024-03-15
draft: false
slug: /cp/sqrt-decomposition
tags:
  - Competitive Programming
  - Data Structures
  - Range Queries
---

When a query is too weird for a [segment tree](/citadel/data-structures/segment-tree) — "number of distinct values in a range", "count of elements greater than $x$" — sqrt decomposition is the fallback. Split the array into blocks of length $\approx \sqrt n$, precompute a per-block answer, and any range hits at most two partial blocks (walk element by element) plus $O(\sqrt n)$ whole blocks (use the precomputed value). Everything is $O(\sqrt n)$ and almost nothing has to be clever.

## The problem

Given an array $a[0 \dots n-1]$, support range queries (sum, min, count, …) and updates, when the merge is awkward to express as a clean associative combine but easy to maintain per block.

Example: $a = [1, 5, 2, 8, 3, 9, 4, 7, 6]$, block size $3$: blocks $[1,5,2],\ [8,3,9],\ [4,7,6]$ with block sums $8, 20, 17$. Sum of $a[1 \dots 7]$ = partial $\{5, 2\}$ + whole block $\{8,3,9\}$ + partial $\{4, 7\}$ = $7 + 20 + 11 = 38$.

## The idea

Block size $B = \lceil \sqrt n \rceil$; block $b$ owns indices $[bB,\ (b+1)B)$. Maintain an aggregate `block[b]` for each block.

- **Query $[l, r]$:** if $l, r$ are in the same block, loop over $[l, r]$ directly. Otherwise: loop the partial prefix $[l, \text{end of }l\text{'s block}]$, add `block[b]` for every whole block strictly between, loop the partial suffix.
- **Point update $a[i] \mathrel{+}= d$:** update $a[i]$ and `block[i // B]` in $O(1)$.

At most $2B$ element visits and $n/B$ block visits per query. Both are minimised at $B = \sqrt n$, giving $O(\sqrt n)$.

## How it works

$a = [1, 5, 2, 8, 3, 9, 4, 7, 6]$, $B = 3$, `block = [8, 20, 17]`. Query sum$(1, 7)$: $l = 1$ (block $0$), $r = 7$ (block $2$), different blocks.

- partial prefix $a[1..2] = 5 + 2 = 7$;
- whole block $1$: `block[1] = 20`;
- partial suffix $a[6..7] = 4 + 7 = 11$.

Total $38$. Now `update(4, +10)`: $a[4] = 13$, `block[1] += 10 → 30`.

## Algorithm

```python
import math

class SqrtDecomp:
    def __init__(self, a: list[int]):
        self.a = a[:]
        self.n = len(a)
        self.B = max(1, int(math.isqrt(self.n)))
        self.block = [0] * ((self.n + self.B - 1) // self.B)
        for i, x in enumerate(a):
            self.block[i // self.B] += x

    def update(self, i: int, delta: int) -> None:
        self.a[i] += delta
        self.block[i // self.B] += delta

    def query(self, l: int, r: int) -> int:          # inclusive sum
        B, res = self.B, 0
        bl, br = l // B, r // B
        if bl == br:
            return sum(self.a[l:r + 1])
        res += sum(self.a[l:(bl + 1) * B])
        for b in range(bl + 1, br):
            res += self.block[b]
        res += sum(self.a[br * B:r + 1])
        return res
```

## Range updates with block tags

For "add $d$ to every element of $[l, r]$", give each block a lazy `add[b]`. Whole blocks inside the range just get `add[b] += d`; partial blocks are rebuilt element-by-element (rewrite `a[i]`, recompute `block[b]`). A query reads `a[i] + add[i // B]` for partial elements and `block[b] + add[b] * (block length)` for whole blocks. Still $O(\sqrt n)$.

## Mo's algorithm: offline queries in O((n + q) sqrt n)

When all queries are known up front and there is no update, sort them cleverly and move two pointers `[cur_l, cur_r]` to each query's range, adding/removing one element at a time.

- Sort queries by block of `l`, then by `r` (ascending, or descending on odd blocks for a constant-factor win).
- `cur_r` moves $O(n)$ per block over all queries in it, $O(n \sqrt n)$ total; `cur_l` moves $O(\sqrt n)$ per query, $O(q \sqrt n)$ total.
- Maintain the answer incrementally in an `add(x)` / `remove(x)` pair — e.g. a frequency table for "distinct count".

```python
def mos(n: int, queries: list[tuple[int, int, int]], add, remove, get):
    B = max(1, int(n ** 0.5))
    order = sorted(range(len(queries)),
                   key=lambda i: (queries[i][0] // B,
                                  queries[i][1] if (queries[i][0] // B) % 2 == 0
                                  else -queries[i][1]))
    ans = [0] * len(queries)
    cur_l, cur_r = 0, -1
    for i in order:
        l, r, _ = queries[i]
        while cur_r < r: cur_r += 1; add(cur_r)
        while cur_l > l: cur_l -= 1; add(cur_l)
        while cur_r > r: remove(cur_r); cur_r -= 1
        while cur_l < l: remove(cur_l); cur_l += 1
        ans[i] = get()
    return ans
```

## Complexity

- **Plain sqrt decomposition:** $O(\sqrt n)$ per query/update, $O(n)$ build, $O(\sqrt n)$ extra space.
- **Mo's algorithm:** $O((n + q)\sqrt n)$ total, offline, no updates (a "with updates" variant adds a time dimension for $O(n^{5/3})$).

## Common pitfalls

- **Block size not $\sqrt n$.** Too small → too many whole blocks; too large → too much partial scanning. Tune $B$ (sometimes $B = \sqrt{n}$ times a small constant helps cache).
- **Same-block query falling through.** When $l$ and $r$ share a block there is no "whole block" phase — handle it as a plain loop first.
- **Mo's move order.** Expand the range (`add`) before shrinking it (`remove`), or you can `remove` an element that is not currently in the multiset. The four `while` loops must be in an order that never lets `cur_l > cur_r + 1` with a spurious remove.
- **Mo's on a problem with updates.** Standard Mo's assumes a static array; adding updates needs the 3-D version and different block sizing.
- **Rebuilding partial blocks after a range update.** Forgetting to recompute `block[b]` for the touched partial blocks silently corrupts later whole-block reads.

## The keystone

Cut the array into $\sqrt n$ blocks, keep a per-block aggregate, and answer any range with $\le 2$ partial scans plus $\sqrt n$ block reads — $O(\sqrt n)$, and it works for queries no segment tree merge expresses. Offline and update-free, the same blocking sorts queries for Mo's algorithm at $O((n+q)\sqrt n)$.
