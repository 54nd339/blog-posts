---
title: Median of Two Sorted Arrays - Binary Search on the Partition
description: The median of two sorted arrays in logarithmic time — merging is O(m+n), but binary-searching where to cut the smaller array so the left halves hold exactly half the elements gets it to O(log min(m,n)).
date: 2021-08-06
draft: false
slug: /dsa/median-of-two-sorted-arrays
tags:
  - Competitive Programming
  - Binary Search
  - Divide and Conquer
---

The median splits all `m + n` values into a lower half and an upper half. Find a cut through `A` and a matching cut through `B` so that everything left of both cuts is `<=` everything right of both cuts — then the median sits on the border.

## Description

Given two sorted arrays `nums1` and `nums2`, return the median of their combined sorted order. The overall run time should be $O(\log(m + n))$.

**Example**

```
Input:  nums1 = [1, 3], nums2 = [2]
Output: 2.0

Input:  nums1 = [1, 2], nums2 = [3, 4]
Output: 2.5
```

**Constraints**

- $0 \le m, n \le 1000$, and $1 \le m + n \le 2000$
- $-10^6 \le \text{nums1}[i], \text{nums2}[i] \le 10^6$

## Prerequisites

- [Binary search](/citadel/dsa/binary-search) on a partition index.
- The definition of median for even vs odd total length.

## Approach 1: Brute Force

### Intuition

Merge and read the middle.

### Algorithm

1. Concatenate both arrays and sort.
2. If the total length is odd, return the middle element; if even, average the two middle elements.

```python
def findMedianSortedArrays(nums1: list[int], nums2: list[int]) -> float:
    merged = sorted(nums1 + nums2)
    n = len(merged)
    if n % 2:
        return float(merged[n // 2])
    return (merged[n // 2 - 1] + merged[n // 2]) / 2
```

### Complexity

- **Time:** $O((m + n) \log(m + n))$.
- **Space:** $O(m + n)$.

## Approach 2: Two Pointers

### Intuition

You only need the middle one or two values, so merge lazily and stop at the halfway point.

### Algorithm

1. Walk two pointers `i`, `j` through the arrays, always advancing the one with the smaller current value.
2. Take `(m + n) // 2 + 1` steps, remembering the last two values taken.
3. Odd total → the last value; even total → the average of the last two.

```python
def findMedianSortedArrays(nums1: list[int], nums2: list[int]) -> float:
    m, n = len(nums1), len(nums2)
    i = j = 0
    prev = cur = 0
    for _ in range((m + n) // 2 + 1):
        prev = cur
        if i < m and (j >= n or nums1[i] <= nums2[j]):
            cur = nums1[i]
            i += 1
        else:
            cur = nums2[j]
            j += 1
    if (m + n) % 2:
        return float(cur)
    return (prev + cur) / 2
```

### Complexity

- **Time:** $O(m + n)$.
- **Space:** $O(1)$.

## Approach 3: Binary Search (k-th element by recursion)

### Intuition

Finding the median is finding the `k`-th smallest (or the `k`-th and `k+1`-th). To find the `k`-th, compare the `k/2`-th elements of each array: the smaller one's first `k/2` elements cannot contain the `k`-th, so discard them and recurse with a reduced `k`.

### Algorithm

1. `getKth(a, b, k)`: make `a` the shorter array. If `a` is empty, return `b[k - 1]`. If `k == 1`, return `min(a[0], b[0])`.
2. `ia = min(len(a), k // 2)`, `ib = k - ia`. If `a[ia - 1] <= b[ib - 1]`, recurse on `a[ia:], b, k - ia`; else `a, b[ib:], k - ib`.
3. Median: `getKth` at `(m + n + 1) // 2`, and for even totals also at `(m + n) // 2 + 1`.

```python
def findMedianSortedArrays(nums1: list[int], nums2: list[int]) -> float:
    def get_kth(a: list[int], b: list[int], k: int) -> int:
        if len(a) > len(b):
            a, b = b, a
        if not a:
            return b[k - 1]
        if k == 1:
            return min(a[0], b[0])
        ia = min(len(a), k // 2)
        ib = k - ia
        if a[ia - 1] <= b[ib - 1]:
            return get_kth(a[ia:], b, k - ia)
        return get_kth(a, b[ib:], k - ib)

    total = len(nums1) + len(nums2)
    if total % 2:
        return float(get_kth(nums1, nums2, total // 2 + 1))
    lo = get_kth(nums1, nums2, total // 2)
    hi = get_kth(nums1, nums2, total // 2 + 1)
    return (lo + hi) / 2
```

### Complexity

- **Time:** $O(\log(m + n))$ — `k` roughly halves per call.
- **Space:** $O(\log(m + n))$ recursion (slicing also copies; use indices to make it $O(1)$).

## Approach 4: Binary Search (Optimal partition)

### Intuition

Binary-search the number of elements `i` taken from the smaller array `A` into the left half; then `j = half - i` come from `B`. The partition is correct when `A[i-1] <= B[j]` and `B[j-1] <= A[i]`. Out-of-range sides use `-inf` / `+inf`.

### Algorithm

1. Make `A` the shorter array; `total = m + n`; `half = total // 2`.
2. `lo = 0`, `hi = m`. Loop:
   - `i = (lo + hi) // 2`; `j = half - i`.
   - `Aleft = A[i-1] if i > 0 else -inf`, `Aright = A[i] if i < m else +inf`; similarly `Bleft`, `Bright`.
   - If `Aleft <= Bright` and `Bleft <= Aright`: if `total` is odd, return `min(Aright, Bright)`; else return `(max(Aleft, Bleft) + min(Aright, Bright)) / 2`.
   - Else if `Aleft > Bright`, `hi = i - 1`; else `lo = i + 1`.

```python
def findMedianSortedArrays(nums1: list[int], nums2: list[int]) -> float:
    A, B = nums1, nums2
    if len(A) > len(B):
        A, B = B, A
    m, n = len(A), len(B)
    total = m + n
    half = total // 2
    lo, hi = 0, m
    while True:
        i = (lo + hi) // 2
        j = half - i
        a_left = A[i - 1] if i > 0 else float("-inf")
        a_right = A[i] if i < m else float("inf")
        b_left = B[j - 1] if j > 0 else float("-inf")
        b_right = B[j] if j < n else float("inf")
        if a_left <= b_right and b_left <= a_right:
            if total % 2:
                return float(min(a_right, b_right))
            return (max(a_left, b_left) + min(a_right, b_right)) / 2
        if a_left > b_right:
            hi = i - 1
        else:
            lo = i + 1
```

### Complexity

- **Time:** $O(\log(\min(m, n)))$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Binary-searching the larger array (Approach 4).** Search the *smaller* one so `j = half - i` stays in range; otherwise `j` can go negative or exceed `n`.
- **Infinity sentinels.** When a partition puts nothing on one side of an array, treat the missing left element as `-inf` and the missing right element as `+inf`, so the cross-checks pass automatically.
- **Even/odd handling.** With `half = total // 2`, the left side has `half` elements; for odd totals the median is the smallest right-side element, for even totals it is the average of the largest left and smallest right.
- **Slicing in the recursion (Approach 3).** `a[ia:]` copies; pass start indices instead to keep it $O(1)$ space.

## The keystone

Some "find the k-th / the median" problems have no array to bisect directly, but a *derived* quantity — here the partition index of the smaller array — is monotone against the correctness test. Binary-search that quantity. It is the same leap as [Koko Eating Bananas](/citadel/dsa/koko-eating-bananas)'s "binary search on the answer", one level more abstract.
