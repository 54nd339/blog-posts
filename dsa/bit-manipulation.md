---
title: Bit Manipulation - A Handful of Identities
description: A guide to the Bit Manipulation section of NeetCode 150 — XOR as a parity accumulator, n and n minus 1 to strip a bit, mask-and-shift for reversal and addition, and the 32-bit sign caveat in Python.
date: 2022-07-25
draft: false
slug: /dsa/bit-manipulation
tags:
  - Competitive Programming
  - Bit Manipulation
  - Math
---

Every problem here rests on one or two bit identities. Learn the identities and the solutions are short. The only recurring nuisance is that Python integers are arbitrary-precision, so emulating fixed-width signed arithmetic needs an explicit `& 0xFFFFFFFF` mask.

## The identities

- **XOR is a parity accumulator.** `x ^ x = 0`, `x ^ 0 = x`, commutative. Folding a list with XOR leaves the bits set an odd number of times.
- **`n & (n - 1)` clears the lowest set bit.** Loop it to count set bits; the count of iterations is the popcount.
- **`n & (-n)` isolates the lowest set bit.** (Not needed below, but the companion identity.)
- **`i >> 1` and `i & 1`** split off the low bit — the basis of the `countBits` recurrence.
- **Build MSB-first with `res = (res << 1) | bit`**; build LSB-first with `res = res * base + digit`.
- **Addition = XOR (sum) + AND-shifted (carry)**, iterated to a fixed point.

## Problem → identity

- [Single Number](/citadel/dsa/single-number) — XOR the whole array; pairs cancel.
- [Missing Number](/citadel/dsa/missing-number) — XOR all indices `0..n` with all values; or subtract from `n(n+1)/2`.
- [Number of 1 Bits](/citadel/dsa/number-of-1-bits) — `n &= n - 1` until zero, counting steps.
- [Counting Bits](/citadel/dsa/counting-bits) — `bits[i] = bits[i >> 1] + (i & 1)` (or `bits[i & (i-1)] + 1`), an $O(n)$ fill.
- [Reverse Bits](/citadel/dsa/reverse-bits) — 32× shift-and-OR, or divide-and-conquer mask swaps.
- [Sum of Two Integers](/citadel/dsa/sum-of-two-integers) — `a ^ b` and `(a & b) << 1`, masked to 32 bits, looped.
- [Reverse Integer](/citadel/dsa/reverse-integer) — digit peel with a pre-multiplication overflow check against `INT_MAX // 10`.

## The Python caveat

In C or Java these operate on 32-bit two's-complement words. In Python:

- Mask every intermediate with `& 0xFFFFFFFF` when carries or shifts could grow the value.
- After the computation, a result above `0x7FFFFFFF` is a negative number: recover it with `~(x ^ 0xFFFFFFFF)`.
- `abs(INT_MIN)` has no positive 32-bit representation — avoid it, or use a wider accumulator.

## Recognising it

- "without using `+`/`-`", "constant space", "linear time and O(1) space" on an array with a pairing structure → XOR.
- "count / reverse / manipulate the bits of ..." → shift-and-mask.
- "find the one that appears once / is missing" → XOR fold or Gauss sum.

## Where this goes next

These identities recur throughout competitive programming: XOR bases and linear algebra over GF(2), `n & -n` in Fenwick trees, bitmask DP over subsets, and SIMD-within-a-register tricks. They are small, but they compound.
