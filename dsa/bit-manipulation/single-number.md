---
title: Single Number - XOR Cancels the Pairs
description: Finding the one element that appears once while all others appear twice — XOR every element together, since a value XORed with itself is zero and the duplicates annihilate.
date: 2022-07-25
draft: false
slug: /dsa/single-number
tags:
  - Competitive Programming
  - Bit Manipulation
  - Arrays
---

XOR has two properties that do all the work: `a ^ a == 0` and `a ^ 0 == a`, and it is commutative. Fold the whole array with XOR and every duplicated value cancels itself out, leaving the lone element.

## Description

Given a non-empty array `nums` where every element appears twice except one, find that single one. Do it in linear time with constant extra space.

**Example**

```
Input:  nums = [2,2,1]
Output: 1

Input:  nums = [4,1,2,1,2]
Output: 4
```

**Constraints**

- $1 \le \text{nums.length} \le 3 \cdot 10^4$
- Every element appears twice except one; $-3 \cdot 10^4 \le \text{nums}[i] \le 3 \cdot 10^4$.

## Prerequisites

- XOR identities: `x ^ x = 0`, `x ^ 0 = x`, commutativity and associativity.

## Approach 1: XOR fold

### Intuition

Accumulate `res ^= x` over all elements. Pairs cancel in any order; `res` ends as the unique value.

### Algorithm

1. `res = 0`.
2. For each `x` in `nums`: `res ^= x`.
3. Return `res`.

```python
def singleNumber(nums: list[int]) -> int:
    res = 0
    for x in nums:
        res ^= x
    return res
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Approach 2: Hash set toggle

### Intuition

Add a number on first sight, remove it on the second. The set ends holding just the single number. Uses $O(n)$ space, so it fails the constraint, but it is the obvious baseline.

### Algorithm

1. `seen = set()`.
2. For each `x`: if `x in seen`, discard it; else add it.
3. Return the sole remaining element.

```python
def singleNumber(nums: list[int]) -> int:
    seen = set()
    for x in nums:
        if x in seen:
            seen.discard(x)
        else:
            seen.add(x)
    return seen.pop()
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Approach 3: Math identity

### Intuition

`2 * sum(set(nums)) - sum(nums)` equals the single number, since each distinct value is counted twice in the first term but the duplicated ones appear twice in `sum(nums)`.

### Algorithm

1. Return `2 * sum(set(nums)) - sum(nums)`.

```python
def singleNumber(nums: list[int]) -> int:
    return 2 * sum(set(nums)) - sum(nums)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ for the set.

## Common Pitfalls

- **Initialising `res` to `nums[0]` then re-XORing it.** Start from `0`; XORing `nums[0]` twice cancels it.
- **Assuming XOR needs sorted or grouped input.** Order is irrelevant — that is the point.
- **Generalising blindly.** This trick works when non-unique elements appear an *even* number of times. "Every other appears three times" needs bit-count-mod-3, not a plain XOR.
- **Overflow in the math version.** Fine in Python; watch it elsewhere.

## The keystone

XOR is a parity accumulator: it remembers only whether each bit has been set an odd number of times. That makes "find the unpaired element" a one-line fold. [Missing Number](/citadel/dsa/missing-number) uses the same fold against the full index range.
