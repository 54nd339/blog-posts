---
title: Counting Bits - Build the Table From Smaller Entries
description: The set-bit count for every integer from 0 to n — a DP where each value reuses an already-computed smaller one, either bits[i >> 1] plus the low bit or bits[i and i minus 1] plus one.
date: 2022-07-28
draft: false
slug: /dsa/counting-bits
tags:
  - Competitive Programming
  - Bit Manipulation
  - Dynamic Programming
---

Computing popcount for each of `0..n` independently is $O(n \log n)$. But every `i` relates to a smaller, already-solved value: `bits[i] = bits[i >> 1] + (i & 1)` (drop the low bit) or `bits[i] = bits[i & (i - 1)] + 1` (drop the lowest set bit). Either gives an $O(n)$ fill.

## Description

Given an integer `n`, return an array `ans` of length `n + 1` where `ans[i]` is the number of `1` bits in the binary representation of `i`.

**Example**

```
Input:  n = 2
Output: [0,1,1]

Input:  n = 5
Output: [0,1,1,2,1,2]
```

**Constraints**

- $0 \le n \le 10^5$

## Prerequisites

- `i >> 1` is `i` with its lowest bit removed; `i & 1` is that bit.
- `i & (i - 1)` is `i` with its lowest *set* bit removed.

## Approach 1: DP with the high bits (`i >> 1`)

### Intuition

`i` in binary is `(i >> 1)` shifted left, plus the last bit. So its popcount is `bits[i >> 1]` plus `i & 1`, and `i >> 1 < i`, so it is already computed.

### Algorithm

1. `bits = [0] * (n + 1)`.
2. For `i` from `1` to `n`: `bits[i] = bits[i >> 1] + (i & 1)`.
3. Return `bits`.

```python
def countBits(n: int) -> list[int]:
    bits = [0] * (n + 1)
    for i in range(1, n + 1):
        bits[i] = bits[i >> 1] + (i & 1)
    return bits
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ for the output.

## Approach 2: DP with the lowest set bit (`i & (i - 1)`)

### Intuition

`i & (i - 1)` clears one set bit, so `bits[i] = bits[i & (i - 1)] + 1`, and the operand is smaller than `i`.

### Algorithm

1. `bits = [0] * (n + 1)`.
2. For `i` from `1` to `n`: `bits[i] = bits[i & (i - 1)] + 1`.
3. Return `bits`.

```python
def countBits(n: int) -> list[int]:
    bits = [0] * (n + 1)
    for i in range(1, n + 1):
        bits[i] = bits[i & (i - 1)] + 1
    return bits
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Approach 3: DP with an offset power of two

### Intuition

Track the largest power of two `<= i` (call it `offset`). Then `bits[i] = 1 + bits[i - offset]`. Update `offset` when `i` is itself a power of two.

### Algorithm

1. `bits = [0] * (n + 1)`, `offset = 1`.
2. For `i` from `1` to `n`: if `offset * 2 == i`, `offset = i`. `bits[i] = 1 + bits[i - offset]`.
3. Return `bits`.

```python
def countBits(n: int) -> list[int]:
    bits = [0] * (n + 1)
    offset = 1
    for i in range(1, n + 1):
        if offset * 2 == i:
            offset = i
        bits[i] = 1 + bits[i - offset]
    return bits
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Common Pitfalls

- **Array size `n` instead of `n + 1`.** You need entries `0` through `n` inclusive.
- **Recomputing popcount per element.** That is the $O(n \log n)$ approach the DP exists to beat.
- **`i >> 1` vs `i // 2`.** Identical for non-negative integers; use whichever reads clearest.
- **Offset update timing.** Update `offset` *before* using it, and only exactly when `i` is the next power of two.

## The keystone

Whenever `f(i)` has a cheap relation to `f` of a strictly smaller argument, an array fill beats independent computation. Here the "smaller argument" is `i` with one bit removed — the DP is a single application of a bit-clearing identity per entry.
