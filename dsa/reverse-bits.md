---
title: Reverse Bits - Pull Bits Off One End, Push Them On the Other
description: Reversing the 32 bits of an unsigned integer — shift the result left and OR in the input's lowest bit 32 times, or swap bit groups with masks in a divide-and-conquer.
date: 2022-08-02
draft: false
slug: /dsa/reverse-bits
tags:
  - Competitive Programming
  - Bit Manipulation
  - Divide and Conquer
---

Bit `i` of the input becomes bit `31 - i` of the output. The simple way: 32 times, shift `result` left by one and OR in `n & 1`, then shift `n` right. The fast way: swap halves, then quarters, then eighths, with fixed masks.

## Description

Reverse the bits of a given 32-bit unsigned integer.

**Example**

```
Input:  n = 43261596  (00000010100101000001111010011100)
Output: 964176192     (00111001011110000010100101000000)
```

**Constraints**

- The input is a 32-bit unsigned integer.

## Prerequisites

- Building a result by `result = (result << 1) | bit`.
- Mask-and-swap for the divide-and-conquer version.

## Approach 1: Bit-by-bit

### Intuition

Repeatedly take the low bit of `n`, append it to `result` (which is being built most-significant-first), and drop that bit from `n`.

### Algorithm

1. `result = 0`.
2. Repeat 32 times: `result = (result << 1) | (n & 1)`; `n >>= 1`.
3. Return `result`.

```python
def reverseBits(n: int) -> int:
    result = 0
    for _ in range(32):
        result = (result << 1) | (n & 1)
        n >>= 1
    return result
```

### Complexity

- **Time:** $O(32) = O(1)$.
- **Space:** $O(1)$.

## Approach 2: Divide-and-conquer mask swaps

### Intuition

Swap the two 16-bit halves, then swap 8-bit groups within each, then 4-bit, then 2-bit, then 1-bit. Five constant steps, no loop.

### Algorithm

1. `n = (n >> 16) | (n << 16)`, masked to 32 bits.
2. `n = ((n & 0xFF00FF00) >> 8) | ((n & 0x00FF00FF) << 8)`.
3. `n = ((n & 0xF0F0F0F0) >> 4) | ((n & 0x0F0F0F0F) << 4)`.
4. `n = ((n & 0xCCCCCCCC) >> 2) | ((n & 0x33333333) << 2)`.
5. `n = ((n & 0xAAAAAAAA) >> 1) | ((n & 0x55555555) << 1)`.

```python
def reverseBits(n: int) -> int:
    n = ((n >> 16) | (n << 16)) & 0xFFFFFFFF
    n = ((n & 0xFF00FF00) >> 8) | ((n & 0x00FF00FF) << 8)
    n = ((n & 0xF0F0F0F0) >> 4) | ((n & 0x0F0F0F0F) << 4)
    n = ((n & 0xCCCCCCCC) >> 2) | ((n & 0x33333333) << 2)
    n = ((n & 0xAAAAAAAA) >> 1) | ((n & 0x55555555) << 1)
    return n & 0xFFFFFFFF
```

### Complexity

- **Time:** $O(1)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Not masking to 32 bits in Python.** `n << 16` on an arbitrary-precision int keeps growing; `& 0xFFFFFFFF` after each widening step keeps it a 32-bit value.
- **Shifting `result` right instead of left.** You build `result` MSB-first, so it shifts left while `n` shifts right.
- **Running 31 iterations.** All 32 bits must move, including a leading zero that lands in the low position.
- **Sign extension.** In fixed-width signed languages, use the logical right shift for `n`.

## The keystone

"Consume from one end, emit to the other" is the bit-level mirror of reversing a list; the mask-swap version is the same halving recursion as [Number of 1 Bits](/citadel/dsa/number-of-1-bits)' parallel popcount. If reversal is called repeatedly, precompute a 256-entry byte table and reverse four bytes per call.
