---
title: Koko Eating Bananas - Binary Search on the Answer
description: The slowest eating speed that clears the piles in time — the feasible speeds form a sorted yes/no boundary, so you binary-search the speed itself rather than any array.
date: 2021-07-22
draft: false
slug: /dsa/koko-eating-bananas
tags:
  - Competitive Programming
  - Binary Search
  - Greedy
---

There is no array to search — the thing you binary-search is `k` itself. If speed `k` is fast enough, so is every speed above it. That monotonic yes/no structure is all binary search needs.

## Description

Koko has `piles` of bananas and `h` hours before the guards return. At speed `k` bananas/hour she picks one pile per hour and eats `k` from it (or the whole pile if it has fewer). Return the minimum integer `k` that lets her finish all piles within `h` hours.

**Example**

```
Input:  piles = [3,6,7,11], h = 8
Output: 4

Input:  piles = [30,11,23,4,20], h = 5
Output: 30
```

**Constraints**

- $1 \le \text{piles.length} \le 10^4$
- $\text{piles.length} \le h \le 10^9$
- $1 \le \text{piles}[i] \le 10^9$

## Prerequisites

- [Binary search](/citadel/dsa/binary-search) on a monotone predicate ("binary search on the answer").
- Ceiling division without floats: `(p + k - 1) // k`.

## Approach 1: Brute Force

### Intuition

Try `k = 1, 2, 3, ...`; the first that fits within `h` hours wins.

### Algorithm

1. For `k` from `1` upward, compute `sum(ceil(p / k) for p in piles)`.
2. Return the first `k` with that sum `<= h`.

```python
def minEatingSpeed(piles: list[int], h: int) -> int:
    k = 1
    while True:
        hours = sum((p + k - 1) // k for p in piles)
        if hours <= h:
            return k
        k += 1
```

### Complexity

- **Time:** $O(m \cdot n)$ where `m = max(piles)`, `n = len(piles)`.
- **Space:** $O(1)$.

## Approach 2: Binary Search

### Intuition

`feasible(k)` is `False` for small `k` and `True` for all `k` at or above the answer — one flip. Bisect `k` in `[1, max(piles)]` for the leftmost `True`.

### Algorithm

1. `lo = 1`, `hi = max(piles)`.
2. While `lo < hi`: `mid = (lo + hi) // 2`. If `hours_needed(mid) <= h`, `hi = mid`; else `lo = mid + 1`.
3. Return `lo`.

```python
def minEatingSpeed(piles: list[int], h: int) -> int:
    def hours_needed(k: int) -> int:
        return sum((p + k - 1) // k for p in piles)

    lo, hi = 1, max(piles)
    while lo < hi:
        mid = (lo + hi) // 2
        if hours_needed(mid) <= h:
            hi = mid
        else:
            lo = mid + 1
    return lo
```

### Complexity

- **Time:** $O(n \log m)$ — $O(\log m)$ iterations, each an $O(n)$ sum.
- **Space:** $O(1)$.

## Common Pitfalls

- **Floor division for the hours.** `p // k` undercounts a partly-eaten pile. Use ceiling: `(p + k - 1) // k` or `-(-p // k)`.
- **Wrong search bounds.** `lo` must be `1` (speed `0` is nonsensical), `hi` is `max(piles)` (a faster speed never helps — one pile per hour is the cap).
- **Using the exact-match binary-search template.** This is a "leftmost value satisfying a predicate" search: `while lo < hi`, `hi = mid` (not `mid - 1`, since `mid` may be the answer), `lo = mid + 1` otherwise.

## The keystone

"Binary search on the answer" applies when you cannot search a list but can cheaply test a candidate answer whose pass/fail is monotone in the candidate. Bisect the value range for the smallest (or largest) passing value — the same shape as minimum-capacity, minimum-largest-sum, and "minimum days" problems, and [Swim in Rising Water](/citadel/dsa/swim-in-rising-water).
