---
title: Sum of Two Integers - Add Without a Plus Sign
description: Adding two integers using only bitwise operators — XOR gives the sum without carries, AND shifted left gives the carries, and you repeat until no carry remains.
date: 2022-08-20
draft: false
slug: /dsa/sum-of-two-integers
tags:
  - Competitive Programming
  - Bit Manipulation
  - Math
---

`a ^ b` adds every bit position while ignoring carries. `(a & b) << 1` is exactly the carries those additions produced. Feed the partial sum and the carry back in; when the carry becomes zero, the XOR is the answer. Negative numbers need a fixed-width mask to emulate two's-complement wraparound.

## Description

Given two integers `a` and `b`, return their sum without using the `+` or `-` operators.

**Example**

```
Input:  a = 1, b = 2
Output: 3

Input:  a = 2, b = 3
Output: 5
```

**Constraints**

- $-1000 \le a, b \le 1000$

## Prerequisites

- `sum_without_carry = a ^ b`; `carry = (a & b) << 1`.
- Masking to 32 bits and reinterpreting the sign for negative results.

## Approach 1: XOR sum with carry loop (32-bit masked)

### Intuition

Repeat: `a, b = (a ^ b), (a & b) << 1`, all masked to 32 bits, until `b` (the carry) is 0. Then, if `a` has its 32nd bit set, it represents a negative number in two's complement — convert it.

### Algorithm

1. `mask = 0xFFFFFFFF`.
2. While `b != 0`: `a, b = (a ^ b) & mask, ((a & b) << 1) & mask`.
3. If `a <= 0x7FFFFFFF`, return `a`; else return `~(a ^ mask)` (sign-extend the 32-bit value).

```python
def getSum(a: int, b: int) -> int:
    mask = 0xFFFFFFFF
    while b != 0:
        a, b = (a ^ b) & mask, ((a & b) << 1) & mask
    return a if a <= 0x7FFFFFFF else ~(a ^ mask)
```

### Complexity

- **Time:** $O(1)$ — at most 32 carry propagations.
- **Space:** $O(1)$.

## Approach 2: Recursive form

### Intuition

`getSum(a, b)` = `getSum(a ^ b, (a & b) << 1)`, base case `b == 0` returns `a`. Same masking caveat.

### Algorithm

1. `mask = 0xFFFFFFFF`.
2. If `b == 0`: return `a` if `a <= 0x7FFFFFFF` else `~(a ^ mask)`.
3. Return `getSum((a ^ b) & mask, ((a & b) << 1) & mask)`.

```python
def getSum(a: int, b: int) -> int:
    mask = 0xFFFFFFFF
    if b == 0:
        return a if a <= 0x7FFFFFFF else ~(a ^ mask)
    return getSum((a ^ b) & mask, ((a & b) << 1) & mask)
```

### Complexity

- **Time:** $O(1)$.
- **Space:** $O(1)$ (bounded recursion depth of 32).

## Common Pitfalls

- **No 32-bit mask in Python.** Without `& mask`, the carry `(a & b) << 1` keeps growing and negative inputs never terminate. Mask every step.
- **Not reinterpreting the sign at the end.** A masked result above `0x7FFFFFFF` is a negative two's-complement number; `~(a ^ mask)` recovers Python's signed value.
- **Swapping XOR and AND roles.** `^` is the carry-less sum; `&` (shifted) is the carry. Reversing them computes nonsense.
- **Loop condition on `a` instead of `b`.** Terminate when the *carry* `b` is 0.

## The keystone

A binary adder is XOR for the digit sum and AND-shift for the carry, iterated to a fixed point — this is literally what a hardware ripple-carry adder does. The 32-bit mask is how you simulate fixed-width two's-complement in a language with big integers.
