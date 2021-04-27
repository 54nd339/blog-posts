---
title: Product of Array Except Self - Prefix and Suffix in One Array
description: For each index, the product of every other element without division and in linear time — brute force, the banned division trick, prefix and suffix arrays, then the same idea in O(1) extra space.
date: 2021-04-27
draft: false
slug: /dsa/product-of-array-except-self
tags:
  - Competitive Programming
  - Arrays & Hashing
  - Prefix Sum
---

Division would make this trivial: total product divided by `nums[i]`. Banning it forces the real insight — the product of everything except `nums[i]` is (everything to its left) times (everything to its right), and both accumulate in a single sweep.

## Description

Given an integer array `nums`, return an array `out` where `out[i]` is the product of all elements of `nums` except `nums[i]`. Run in $O(n)$ time and without using the division operator.

**Example**

```
Input:  nums = [1, 2, 3, 4]
Output: [24, 12, 8, 6]

Input:  nums = [-1, 1, 0, -3, 3]
Output: [0, 0, 9, 0, 0]
```

**Constraints**

- $2 \le \text{nums.length} \le 10^5$
- $-30 \le \text{nums}[i] \le 30$
- Every prefix and suffix product fits in a 32-bit integer.

## Prerequisites

- Prefix / suffix accumulation: `out[i] = f(left of i) ⊕ f(right of i)` for an associative `⊕`.
- Using the output array as scratch space to hit $O(1)$ extra space.

## Approach 1: Brute Force

### Intuition

For each position, multiply together every *other* element.

### Algorithm

1. Create `out` of length `n`.
2. For each `i`: set `p = 1`, loop `j` over all indices, multiply `p` by `nums[j]` when `j != i`, then store `p` in `out[i]`.

```python
def productExceptSelf(nums: list[int]) -> list[int]:
    n = len(nums)
    out = [1] * n
    for i in range(n):
        p = 1
        for j in range(n):
            if j != i:
                p *= nums[j]
        out[i] = p
    return out
```

### Complexity

- **Time:** $O(n^2)$ — an inner pass per element.
- **Space:** $O(1)$ beyond the output.

## Approach 2: Division

### Intuition

Multiply everything once; then `out[i]` is that total divided by `nums[i]`. Zeros need care, and the operator is banned — but it is the baseline the real solution routes around.

### Algorithm

1. Compute the product of all non-zero elements and count the zeros.
2. If more than one zero, every `out[i]` is `0`.
3. If exactly one zero, only its position gets the non-zero product; all others get `0`.
4. If no zeros, `out[i] = total // nums[i]`.

```python
def productExceptSelf(nums: list[int]) -> list[int]:
    zeros = nums.count(0)
    prod = 1
    for x in nums:
        if x != 0:
            prod *= x
    if zeros > 1:
        return [0] * len(nums)
    if zeros == 1:
        return [prod if x == 0 else 0 for x in nums]
    return [prod // x for x in nums]
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$ beyond the output.

## Approach 3: Prefix & Suffix

### Intuition

`prefix[i]` = product of everything left of `i`; `suffix[i]` = product of everything right of `i`. Then `out[i] = prefix[i] * suffix[i]`. Each array is one running-product scan, and zeros just flow through.

### Algorithm

1. Create `prefix`, `suffix`, `out`, each length `n`. Set `prefix[0] = 1`, `suffix[n-1] = 1`.
2. Left to right: `prefix[i] = prefix[i-1] * nums[i-1]`.
3. Right to left: `suffix[i] = suffix[i+1] * nums[i+1]`.
4. `out[i] = prefix[i] * suffix[i]`.

```python
def productExceptSelf(nums: list[int]) -> list[int]:
    n = len(nums)
    prefix = [1] * n
    suffix = [1] * n
    for i in range(1, n):
        prefix[i] = prefix[i - 1] * nums[i - 1]
    for i in range(n - 2, -1, -1):
        suffix[i] = suffix[i + 1] * nums[i + 1]
    return [prefix[i] * suffix[i] for i in range(n)]
```

### Complexity

- **Time:** $O(n)$ — three linear passes.
- **Space:** $O(n)$ for the two helper arrays.

## Approach 4: Prefix & Suffix (Optimal)

### Intuition

You never need both helper arrays stored at once. Write prefix products into `out` on the first pass; on the second pass carry a single `suffix` scalar and fold it in.

### Algorithm

1. Fill `out` with `1`s.
2. Left to right: set `out[i]` to the running prefix, then multiply the prefix by `nums[i]`.
3. Right to left: multiply `out[i]` by the running suffix, then multiply the suffix by `nums[i]`.

```python
def productExceptSelf(nums: list[int]) -> list[int]:
    n = len(nums)
    out = [1] * n
    prefix = 1
    for i in range(n):
        out[i] = prefix
        prefix *= nums[i]
    suffix = 1
    for i in range(n - 1, -1, -1):
        out[i] *= suffix
        suffix *= nums[i]
    return out
```

### Complexity

- **Time:** $O(n)$ — two passes.
- **Space:** $O(1)$ extra — `out` is required output, `prefix` and `suffix` are scalars.

## Common Pitfalls

- **Off-by-one in the recurrences.** `prefix[i]` uses `nums[i-1]`, not `nums[i]` — it is the product of elements strictly *before* `i`. `prefix[0]` and `suffix[n-1]` stay `1`.
- **Updating the rolling scalar too early (Approach 4).** Set `out[i]` from the *current* prefix/suffix, then advance the scalar; do it the other way and you fold in `nums[i]` itself.
- **The division approach on zeros.** One zero, two zeros, and no zeros are three distinct cases — the prefix/suffix method needs none of them.

## The keystone

"For each position, combine everything on the left with everything on the right" is a prefix/suffix problem, and prefix/suffix accumulators make it linear — product here, sum in a prefix-sum problem, min/max elsewhere. Using the output array as scratch keeps it at $O(1)$ extra space.
