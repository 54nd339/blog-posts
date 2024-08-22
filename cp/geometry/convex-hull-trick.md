---
title: Convex Hull Trick and Li Chao Tree - DP With Linear Functions
description: Speeding up DP transitions of the form dp[i] = min over j of (slope_j * x_i + intercept_j) by maintaining the lower envelope of a set of lines, via a monotonic stack or a Li Chao segment tree.
date: 2024-08-22
draft: false
slug: /cp/convex-hull-trick
tags:
  - Competitive Programming
  - Geometry
  - Dynamic Programming
---

A recurring DP shape: $dp[i] = \min_{j < i}\big(k_j \cdot x_i + m_j\big)$, where each earlier state $j$ contributes a **line** $y = k_j x + m_j$ and you query it at $x = x_i$. Evaluating all $j$ is $O(n)$ per state, $O(n^2)$ total. The convex hull trick keeps the **lower envelope** of the lines so each query is $O(\log n)$ (or $O(1)$ with monotone queries).

## The problem

Support:

- **add a line** $y = kx + m$,
- **query** $\min_j (k_j x + m_j)$ at a given $x$ (or $\max$).

The lines that can ever be the minimum at some $x$ form the lower envelope — a convex piecewise-linear function. Non-contributing lines (dominated everywhere) are discarded.

## Variant 1: monotonic-stack CHT (offline-ish)

When lines are added in **decreasing slope order** and queries come in **increasing $x$ order** (for a min envelope), keep the envelope in a stack. Adding a line pops from the back every line that the new one makes redundant — the test: line $b$ (between $a$ and $c$ on the stack) is redundant iff the $a$–$c$ intersection is at or left of the $a$–$b$ intersection, i.e.

$$(m_c - m_a)(k_a - k_b) \le (m_b - m_a)(k_a - k_c)$$

(cross-multiplied to avoid division; both $k_a - k_b$ and $k_a - k_c$ are positive under decreasing slopes). Queries with increasing $x$ advance a pointer over the envelope.

```python
class MonotonicCHT:
    """Lines added by DECREASING slope; queries by INCREASING x; returns min."""
    def __init__(self):
        self.k = []
        self.m = []
        self.ptr = 0

    def _bad(self, a, b, c):
        # line b is redundant given a and c: the a-c crossing is at or left of the a-b crossing
        return (self.m[c] - self.m[a]) * (self.k[a] - self.k[b]) <= \
               (self.m[b] - self.m[a]) * (self.k[a] - self.k[c])

    def add(self, k, m):
        self.k.append(k); self.m.append(m)
        while len(self.k) >= 3 and self._bad(len(self.k) - 3, len(self.k) - 2, len(self.k) - 1):
            self.k.pop(-2); self.m.pop(-2)
        if self.ptr >= len(self.k):
            self.ptr = len(self.k) - 1

    def query(self, x):
        if self.ptr >= len(self.k):
            self.ptr = len(self.k) - 1
        while self.ptr + 1 < len(self.k) and \
              self.k[self.ptr + 1] * x + self.m[self.ptr + 1] <= self.k[self.ptr] * x + self.m[self.ptr]:
            self.ptr += 1
        return self.k[self.ptr] * x + self.m[self.ptr]
```

## Variant 2: Li Chao tree (any order)

A segment tree over the range of query $x$-values (compressed if sparse). Each node stores **one line** — the one that is minimal at the node's midpoint. Inserting a line: at each node, keep whichever of (stored, new) is better at the midpoint, and recurse into the half where the other line *could* still win. Query at $x$: walk root-to-leaf taking the min of every stored line on the path.

```python
NEG = float("inf")

class LiChao:
    def __init__(self, xs):
        self.xs = sorted(set(xs))
        self.n = len(self.xs)
        self.line = [(0, NEG)] * (4 * self.n)      # (k, m); NEG intercept = "no line"

    def _f(self, line, i):
        k, m = line
        return k * self.xs[i] + m if m != NEG else NEG

    def _insert(self, node, lo, hi, nl):
        mid = (lo + hi) // 2
        cur = self.line[node]
        if self._f(nl, mid) < self._f(cur, mid):
            self.line[node], nl, cur = nl, cur, nl
        if lo == hi:
            return
        if self._f(nl, lo) < self._f(self.line[node], lo):
            self._insert(2 * node, lo, mid, nl)
        elif self._f(nl, hi) < self._f(self.line[node], hi):
            self._insert(2 * node + 1, mid + 1, hi, nl)

    def add_line(self, k, m):
        self._insert(1, 0, self.n - 1, (k, m))

    def query(self, x):
        i = self._index(x)
        node, lo, hi = 1, 0, self.n - 1
        best = NEG
        while True:
            best = min(best, self._f(self.line[node], i))
            if lo == hi:
                return best
            mid = (lo + hi) // 2
            if i <= mid:
                node, hi = 2 * node, mid
            else:
                node, lo = 2 * node + 1, mid + 1

    def _index(self, x):
        import bisect
        return bisect.bisect_left(self.xs, x)
```

For $\max$ instead of $\min$, negate $k$ and $m$ on insert and negate the query result, or flip the comparisons.

## When to use which

| | slopes sorted? | queries sorted? | structure |
| --- | --- | --- | --- |
| both | yes | yes | monotonic deque, $O(1)$ amortised per op |
| slopes sorted only | yes | any | deque + binary search, $O(\log n)$ query |
| neither | any | any | Li Chao tree, $O(\log C)$ per op |
| lines change / removed | — | — | Li Chao (persistent), or kinetic segment tree |

Classic DP applications: [divide-and-conquer-DP](/citadel/cp/divide-and-conquer-dp)-style "split into segments" costs where the cost is a product, "minimum time with acceleration choices", and problems where $dp[i] = \min_j dp[j] + (\text{prefix}_i - \text{prefix}_j) \cdot c_i$ rearranges into a line query.

## Common pitfalls

- **Slope order for the monotonic version.** Add lines by *monotone* slope (all increasing or all decreasing, matching your query direction and min/max). Out-of-order additions corrupt the envelope — use Li Chao instead.
- **Division in the "bad line" test.** Compare intersections by cross-multiplication; floating-point intersection $x$-values lose precision and can pop the wrong line.
- **Li Chao range.** The tree must cover every query $x$. If $x$ can be any real, compress to the actual query points, or use a large fixed integer range.
- **`max` vs `min`.** Decide once. Half-converting (flipping some comparisons) silently returns garbage.
- **Empty structure query.** Return $+\infty$ (for min) if no line has been added; guard the DP base case.

## The keystone

$dp[i] = \min_j (k_j x_i + m_j)$ is a "lower envelope of lines" query. With slopes and queries both monotone, a deque gives $O(1)$ amortised; otherwise a Li Chao segment tree — one line per node, recurse where the loser might still win — handles arbitrary insertion and query order in $O(\log C)$.
