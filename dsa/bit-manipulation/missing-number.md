---
title: Missing Number - XOR the Indices Against the Values
description: Finding the absent value in a permutation of 0 to n — XOR every index and every element together so all present values cancel and only the missing one survives, or subtract from the Gauss sum.
date: 2022-08-02
draft: false
slug: /dsa/missing-number
tags:
  - Competitive Programming
  - Bit Manipulation
  - Math
---

The array holds `n` of the `n + 1` values `0..n`. XOR together all indices `0..n` and all array elements: every value that *is* present appears once as an index and once as a value and cancels, leaving the one missing number. A sum-based version works equally well.

## Description

Given an array `nums` containing `n` distinct numbers drawn from `0, 1, ..., n`, return the only number in that range missing from the array.

**Example**

```
Input:  nums = [3,0,1]
Output: 2

Input:  nums = [0,1]
Output: 2
```

**Constraints**

- $n == \text{nums.length}$, $1 \le n \le 10^4$
- All elements distinct, each in `[0, n]`.

## Prerequisites

- XOR cancellation, as in [Single Number](/citadel/dsa/single-number).
- The arithmetic series $0 + 1 + \dots + n = n(n+1)/2$.

## Approach 1: XOR indices and values

### Intuition

Start `res = n` (covers the index `n` that has no matching array slot). For each `i`, `res ^= i ^ nums[i]`. Present values cancel; the missing one remains.

### Algorithm

1. `res = len(nums)`.
2. For `i, x` in `enumerate(nums)`: `res ^= i ^ x`.
3. Return `res`.

```python
def missingNumber(nums: list[int]) -> int:
    res = len(nums)
    for i, x in enumerate(nums):
        res ^= i ^ x
    return res
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 2: Gauss sum

### Intuition

The expected sum of `0..n` is `n(n+1)/2`. Subtract the actual array sum; the difference is the missing value.

### Algorithm

1. `n = len(nums)`.
2. Return `n * (n + 1) // 2 - sum(nums)`.

```python
def missingNumber(nums: list[int]) -> int:
    n = len(nums)
    return n * (n + 1) // 2 - sum(nums)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 3: Index-as-hash cyclic / boolean scan

### Intuition

Put each value in a `seen` boolean array of length `n + 1`, then find the index never marked.

### Algorithm

1. `seen = [False] * (n + 1)`; for each `x`, `seen[x] = True`.
2. Return the index `i` with `seen[i]` false.

```python
def missingNumber(nums: list[int]) -> int:
    n = len(nums)
    seen = [False] * (n + 1)
    for x in nums:
        seen[x] = True
    for i in range(n + 1):
        if not seen[i]:
            return i
    return -1
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **Starting `res` at 0 in the XOR version.** You must fold in the index `n` (there are `n + 1` indices `0..n` but only `n` array positions); seed `res = n` or loop `i` up to `n` separately.
- **Integer overflow in the Gauss sum.** Safe in Python; in 32-bit languages `n(n+1)/2` can overflow for large `n` — use a wider type or subtract termwise.
- **Assuming the array is sorted.** None of these approaches need it; a binary-search variant does.
- **Duplicates or out-of-range values.** The problem guarantees neither occurs; the math/XOR tricks rely on that.

## The keystone

Pairing each element with its "expected" counterpart (an index, or a term of a known series) so matches cancel is the core trick — XOR for exact cancellation in $O(1)$ space, subtraction from a closed form as the arithmetic twin.
