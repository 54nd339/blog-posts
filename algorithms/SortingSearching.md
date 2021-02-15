---
title: Sorting and Searching - Seven Sorts and Two Searches
description: The foundational algorithms for ordering data and finding it again. Three quadratic sorts, three O(n log n) sorts, one linear-time non-comparison sort, and why binary search needs the data sorted first.
date: 2021-02-15
draft: false
slug: /algorithms/SortingSearching
tags:
  - Algorithms
  - Sorting and Searching
---

Sorting data makes almost everything else faster: searching, deduplication, finding medians, detecting duplicates, database joins. And once data is sorted, **binary search** finds any element in $O(\log n)$ instead of scanning. This is a tour of the seven sorts and two searches everyone should know, with the reason behind each complexity.

## Comparison sorts

A comparison sort orders elements using only pairwise `<` comparisons. No comparison sort can beat $O(n \log n)$ in the worst case — there are $n!$ possible orderings and each comparison halves the possibilities, so you need at least $\log_2(n!) \approx n \log n$ of them.

### The quadratic three

**Bubble sort** repeatedly sweeps the list, swapping adjacent out-of-order pairs; after pass $k$ the largest $k$ elements are in place. If a sweep makes no swaps, it stops early.

```python
def bubble_sort(a):
    for i in range(len(a) - 1):
        swapped = False
        for j in range(len(a) - 1 - i):
            if a[j] > a[j + 1]:
                a[j], a[j + 1] = a[j + 1], a[j]
                swapped = True
        if not swapped:
            break
    return a
```

**Selection sort** finds the minimum of the unsorted suffix and swaps it to the front. Always $O(n^2)$ — it can't detect a sorted input — but it does the fewest swaps ($n-1$).

```python
def selection_sort(a):
    for i in range(len(a)):
        m = min(range(i, len(a)), key=a.__getitem__)
        a[i], a[m] = a[m], a[i]
    return a
```

**Insertion sort** grows a sorted prefix, sliding each new element back to its place. $O(n^2)$ worst case, but $O(n)$ on nearly-sorted input, which makes it the standard base case inside faster sorts for small subarrays.

```python
def insertion_sort(a):
    for i in range(1, len(a)):
        key, j = a[i], i - 1
        while j >= 0 and a[j] > key:
            a[j + 1] = a[j]
            j -= 1
        a[j + 1] = key
    return a
```

### The n log n three

**Merge sort** splits the list in half, sorts each half recursively, and merges the two sorted halves in linear time. $O(n \log n)$ always ($\log n$ levels, $O(n)$ merge per level), stable, but needs $O(n)$ scratch space.

```python
def merge_sort(a):
    if len(a) <= 1:
        return a
    mid = len(a) // 2
    left, right = merge_sort(a[:mid]), merge_sort(a[mid:])
    out, i, j = [], 0, 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            out.append(left[i]); i += 1
        else:
            out.append(right[j]); j += 1
    out.extend(left[i:]); out.extend(right[j:])
    return out
```

**Quick sort** picks a pivot, partitions the array into "less than pivot" and "greater than pivot" (the pivot lands in its final spot), and recurses on each side. $O(n \log n)$ on average, but $O(n^2)$ if the pivot is consistently the smallest or largest element — mitigated by choosing the pivot randomly or as a median-of-three. In place, not stable.

```python
def quick_sort(a, lo=0, hi=None):
    if hi is None:
        hi = len(a) - 1
    if lo >= hi:
        return a
    pivot = a[hi]
    i = lo - 1
    for j in range(lo, hi):
        if a[j] < pivot:
            i += 1
            a[i], a[j] = a[j], a[i]
    a[i + 1], a[hi] = a[hi], a[i + 1]
    p = i + 1
    quick_sort(a, lo, p - 1)
    quick_sort(a, p + 1, hi)
    return a
```

**Heap sort** builds a max-heap in $O(n)$, then repeatedly swaps the root (the maximum) to the end and re-heapifies the shrinking prefix. $O(n \log n)$ always, in place, not stable.

```python
def heap_sort(a):
    n = len(a)
    def sift(root, size):
        while True:
            largest, l, r = root, 2 * root + 1, 2 * root + 2
            if l < size and a[l] > a[largest]:
                largest = l
            if r < size and a[r] > a[largest]:
                largest = r
            if largest == root:
                return
            a[root], a[largest] = a[largest], a[root]
            root = largest
    for i in range(n // 2 - 1, -1, -1):
        sift(i, n)
    for end in range(n - 1, 0, -1):
        a[0], a[end] = a[end], a[0]
        sift(0, end)
    return a
```

Merge sort's recurrence, $2T(n/2) + O(n)$, is the same one behind [closest pair of points](/citadel/algorithms/ClosestPair) and [the FFT](/citadel/algorithms/FastFourierTransform).

## Non-comparison: radix sort

By sorting on the *structure* of keys instead of comparing them, you can beat $O(n \log n)$. **LSD radix sort** sorts integers one digit at a time, least-significant first, using a stable counting sort for each digit. With $d$ digits it's $O(d(n + b))$ for base $b$ — linear in $n$ when $d$ is a small constant.

```python
def radix_sort(a):
    if not a:
        return a
    exp = 1
    hi = max(a)
    while hi // exp > 0:
        buckets = [[] for _ in range(10)]
        for x in a:
            buckets[(x // exp) % 10].append(x)     # stable: preserves prior order
        a = [x for bucket in buckets for x in bucket]
        exp *= 10
    return a
```

## Searching

**Linear search** scans until it finds the target: $O(n)$, works on any list.

```python
def linear_search(a, target):
    for i, x in enumerate(a):
        if x == target:
            return i
    return -1
```

**Binary search** needs a **sorted** list. Compare the target to the middle element and discard half the range each step: $O(\log n)$.

```python
def binary_search(a, target):
    lo, hi = 0, len(a) - 1
    while lo <= hi:
        mid = (lo + hi) // 2
        if a[mid] == target:
            return mid
        if a[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return -1
```

## The reference table

| Algorithm | Best | Average | Worst | Space | Stable |
| --- | --- | --- | --- | --- | --- |
| Bubble sort | $O(n)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | yes |
| Selection sort | $O(n^2)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | no |
| Insertion sort | $O(n)$ | $O(n^2)$ | $O(n^2)$ | $O(1)$ | yes |
| Merge sort | $O(n \log n)$ | $O(n \log n)$ | $O(n \log n)$ | $O(n)$ | yes |
| Quick sort | $O(n \log n)$ | $O(n \log n)$ | $O(n^2)$ | $O(\log n)$ | no |
| Heap sort | $O(n \log n)$ | $O(n \log n)$ | $O(n \log n)$ | $O(1)$ | no |
| Radix sort (LSD) | $O(nk)$ | $O(nk)$ | $O(nk)$ | $O(n + b)$ | yes |
| Linear search | $O(1)$ | $O(n)$ | $O(n)$ | $O(1)$ | — |
| Binary search | $O(1)$ | $O(\log n)$ | $O(\log n)$ | $O(1)$ | — |

## Choosing

In practice you call the language's built-in sort, which is a hybrid — Python's Timsort merges runs of already-ordered data and falls back to insertion sort on small pieces; C++'s `std::sort` is introsort (quicksort that switches to heapsort if recursion goes too deep). Reach for a specific algorithm when you have a specific reason: **insertion sort** for tiny or nearly-sorted arrays, **merge sort** when you need stability or are sorting a linked list, **heap sort** when $O(1)$ space matters, **radix sort** for fixed-width integer keys. The reason to sort at all is usually the last row of the table: once the data is ordered, every lookup drops from $O(n)$ to $O(\log n)$.
