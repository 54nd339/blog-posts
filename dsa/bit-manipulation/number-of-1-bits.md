---
title: Number of 1 Bits - Clear the Lowest Set Bit Each Step
description: Counting set bits in an integer — either shift and test each of the 32 bits, or repeatedly apply n and n minus 1 to erase the lowest set bit and count the iterations.
date: 2022-07-26
draft: false
slug: /dsa/number-of-1-bits
tags:
  - Competitive Programming
  - Bit Manipulation
  - Divide and Conquer
---

`n & (n - 1)` clears the lowest set bit of `n` and leaves every other bit untouched. Loop that until `n` is zero, counting the steps — the count is exactly the number of 1 bits, and the loop runs once per set bit rather than 32 times.

## Description

Write a function that takes an unsigned integer and returns the number of `'1'` bits it has (its Hamming weight).

**Example**

```
Input:  n = 11   (binary 1011)
Output: 3

Input:  n = 128  (binary 10000000)
Output: 1
```

**Constraints**

- The input is a 32-bit unsigned integer.

## Prerequisites

- `n & (n - 1)` removes the lowest set bit.
- Bit masking with `n & 1` and right shift.

## Approach 1: Clear the lowest set bit (Brian Kernighan)

### Intuition

Each iteration removes one set bit. When `n` reaches 0, the number of iterations equals the original popcount.

### Algorithm

1. `count = 0`.
2. While `n`: `n &= n - 1`; `count += 1`.
3. Return `count`.

```python
def hammingWeight(n: int) -> int:
    count = 0
    while n:
        n &= n - 1
        count += 1
    return count
```

### Complexity

- **Time:** $O(k)$ where `k` is the number of set bits (at most 32).
- **Space:** $O(1)$.

## Approach 2: Shift and test 32 bits

### Intuition

Look at each bit position once: add `n & 1`, then shift right.

### Algorithm

1. `count = 0`.
2. Repeat 32 times: `count += n & 1`; `n >>= 1`.
3. Return `count`.

```python
def hammingWeight(n: int) -> int:
    count = 0
    for _ in range(32):
        count += n & 1
        n >>= 1
    return count
```

### Complexity

- **Time:** $O(32) = O(1)$.
- **Space:** $O(1)$.

## Approach 3: Parallel bit-count (SWAR)

### Intuition

Add adjacent bits in pairs, then nibbles, then bytes, using masks — a divide-and-conquer sum that finishes in a fixed handful of operations.

### Algorithm

1. `n = n - ((n >> 1) & 0x55555555)`.
2. `n = (n & 0x33333333) + ((n >> 2) & 0x33333333)`.
3. `n = (n + (n >> 4)) & 0x0F0F0F0F`.
4. Return `(n * 0x01010101) >> 24`.

```python
def hammingWeight(n: int) -> int:
    n = n - ((n >> 1) & 0x55555555)
    n = (n & 0x33333333) + ((n >> 2) & 0x33333333)
    n = (n + (n >> 4)) & 0x0F0F0F0F
    return ((n * 0x01010101) & 0xFFFFFFFF) >> 24
```

### Complexity

- **Time:** $O(1)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **`n & (n - 1)` on a signed negative in a fixed-width language.** Use the unsigned/logical form; Python's arbitrary-precision ints make `n` non-negative here so it is fine.
- **Infinite loop from an arithmetic right shift.** In languages where `>>` sign-extends, a negative `n` never reaches 0; mask to 32 bits or use `>>>`.
- **Counting 64 iterations.** The input is 32-bit; looping 32 times is enough.
- **Forgetting to mask the SWAR multiply.** In Python, `& 0xFFFFFFFF` after the multiply keeps it a 32-bit result.

## The keystone

`n & (n - 1)` is the fundamental "strip the lowest set bit" move, and `n & (-n)` isolates it. Both underpin fast popcount, bit-subset enumeration, and Fenwick trees. [Counting Bits](/citadel/dsa/counting-bits) builds a whole table on top of this one operation.
