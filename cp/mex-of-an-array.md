---
title: MEX - The Minimal Excluded Non-Negative Integer
description: Computing the smallest non-negative integer absent from a collection - the O(n) static scan, maintaining it under insertions and deletions with a heap of missing values, and answering it over arbitrary subarrays with a segment tree of last occurrences.
date: 2024-10-31
draft: false
slug: /cp/mex-of-an-array
tags:
  - Competitive Programming
  - Miscellaneous
  - Sequences
---

The **MEX** (minimum excludant) of a set of non-negative integers is the smallest non-negative integer it does not contain: $\operatorname{mex}\{0,1,2,4\} = 3$, $\operatorname{mex}\{1,2\} = 0$, $\operatorname{mex}\{\} = 0$. It is the heart of [Sprague-Grundy theory](/citadel/cp/sprague-grundy-nim) and shows up directly in constructive and counting problems.

## The key bound

The MEX of a multiset of $n$ elements is at most $n$: the $n+1$ values $0, 1, \dots, n$ cannot all be present. So you never need to look past index $n$, and a boolean array of size $n + 1$ suffices.

## Static array: O(n)

```python
def mex(a):
    n = len(a)
    present = [False] * (n + 1)
    for x in a:
        if 0 <= x <= n:
            present[x] = True
    for i in range(n + 1):
        if not present[i]:
            return i
```

## Under insertions and deletions

Keep a count per value and a min-heap of *candidate missing* values. A value re-enters the "missing" pool when its count drops to $0$; on query, discard heap-top entries that are actually present.

```python
import heapq

class MexMultiset:
    def __init__(self, limit):
        self.cnt = [0] * (limit + 2)
        self.missing = list(range(limit + 2))      # every value starts missing
        heapq.heapify(self.missing)

    def add(self, x):
        self.cnt[x] += 1

    def remove(self, x):
        self.cnt[x] -= 1
        if self.cnt[x] == 0:
            heapq.heappush(self.missing, x)        # x might be the new mex

    def mex(self):
        while self.cnt[self.missing[0]] > 0:
            heapq.heappop(self.missing)
        return self.missing[0]
```

Each `add`/`remove` is $O(1)$ amortised plus a possible $O(\log n)$ push; `mex` is $O(\log n)$ amortised. A `SortedList` of missing values works too, with the same complexity and easier "smallest missing $\ge k$" queries. If the whole array is a permutation-like set with only value toggles, a Fenwick tree over "is value $v$ present" plus a binary search for the first prefix sum $< v+1$ also gives $O(\log n)$.

## MEX of a subarray (offline)

For queries "MEX of $a[l \dots r]$", sort them by $r$ and sweep $r$ from left to right. Maintain, for each value $v$, `last[v]` = the largest index $\le r$ where $v$ occurs (or $-1$). Then

$$\operatorname{mex}(a[l \dots r]) = \min \{\, v : \texttt{last}[v] < l \,\}.$$

Put `last[]` in a segment tree that supports "leftmost position holding a value $< l$" (a descent from the root, going left whenever the left child's minimum is below $l$). Each query is $O(\log n)$.

```python
class MinSeg:                                    # over values 0..V, holds last-occurrence index
    def __init__(self, size):
        self.n = 1
        while self.n < size:
            self.n <<= 1
        self.t = [-1] * (2 * self.n)

    def set(self, i, val):
        i += self.n
        self.t[i] = val
        i >>= 1
        while i:
            self.t[i] = min(self.t[2 * i], self.t[2 * i + 1])
            i >>= 1

    def first_below(self, x):                    # smallest value index whose last-occ < x
        if self.t[1] >= x:
            return self.n                        # nothing missing below the cap -> mex is n
        i = 1
        while i < self.n:
            i = 2 * i if self.t[2 * i] < x else 2 * i + 1
        return i - self.n

def subarray_mex(a, queries):
    V = len(a) + 2
    seg = MinSeg(V)
    ans = [0] * len(queries)
    by_r = sorted(range(len(queries)), key=lambda k: queries[k][1])
    r = -1
    for qi in by_r:
        l, qr = queries[qi]
        while r < qr:
            r += 1
            if a[r] < V:
                seg.set(a[r], r)
        ans[qi] = seg.first_below(l)
    return ans
```

For an **online** version, persist the segment tree (one version per prefix) and query version $r$ with threshold $l$: $O(\log n)$ per query, $O(n \log n)$ memory.

## Complexity

| Task | Time |
| --- | --- |
| static MEX | $O(n)$ |
| MEX with point updates | $O(\log n)$ per op |
| MEX of subarray, offline | $O((n + q)\log n)$ |
| MEX of subarray, online | $O((n + q)\log n)$, $O(n\log n)$ memory (persistent) |

## Common pitfalls

- **Sizing the structure past $n$.** Values larger than $n$ can never be the MEX; clamp or ignore them, but keep room for value $n$ itself.
- **Stale heap entries.** After a deletion re-adds $x$ to `missing`, a later re-insertion of $x$ must not be trusted from the heap — always re-check `cnt` at the top before returning.
- **Subarray MEX with the wrong comparison.** It is `last[v] < l` (strictly), because an occurrence *at* $l$ still lies in the subarray.
- **Sweeping queries unsorted.** The last-occurrence sweep only works if $r$ is non-decreasing; sort the queries first and scatter answers back by original index.
- **Empty range.** $\operatorname{mex}$ of an empty subarray is $0$; make sure $l \le r$ is guaranteed or handle it.

## The keystone

MEX never exceeds the element count, so it lives in a size-$(n{+}1)$ window. Static: one scan. Dynamic: a heap (or ordered set) of values currently missing. Over subarrays: sweep $r$, keep each value's last occurrence in a segment tree, and the MEX is the smallest value whose last occurrence fell out of the window.
