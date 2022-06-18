---
title: Pow(x, n) - Square Your Way Up the Exponent
description: Computing x to the n in logarithmic time — halve the exponent each step, squaring the base, and multiply in the base whenever the current exponent bit is odd, with care for negative n.
date: 2022-06-18
draft: false
slug: /dsa/powx-n
tags:
  - Competitive Programming
  - Math & Geometry
  - Recursion
---

$x^n$ built by repeated multiplication is $O(n)$. Fast exponentiation is $O(\log n)$: $x^n = (x^{n/2})^2$ for even `n`, and $x \cdot x^{n-1}$ for odd `n`. Equivalently, walk the bits of `n`, squaring `x` each step and accumulating `x` whenever a bit is set.

## Description

Implement `pow(x, n)`, computing `x` raised to the integer power `n`.

**Example**

```
Input:  x = 2.00000, n = 10
Output: 1024.00000

Input:  x = 2.00000, n = -2
Output: 0.25000
```

**Constraints**

- $-100.0 < x < 100.0$
- $-2^{31} \le n \le 2^{31} - 1$
- Either `x != 0` or `n > 0`; the result fits within range.

## Prerequisites

- The identity $x^n = (x^{n/2})^2 \cdot x^{n \bmod 2}$.
- Handling negative `n` by inverting the base or the result.

## Approach 1: Iterative binary exponentiation

### Intuition

Let `result = 1`, `base = x`, `e = abs(n)`. While `e > 0`: if `e` is odd, `result *= base`; then `base *= base`, `e //= 2`. Invert at the end if `n < 0`.

### Algorithm

1. `e = abs(n)`, `result = 1.0`, `base = x`.
2. While `e`: if `e & 1`, `result *= base`. `base *= base`. `e >>= 1`.
3. Return `result` if `n >= 0` else `1 / result`.

```python
def myPow(x: float, n: int) -> float:
    e = abs(n)
    result = 1.0
    base = x
    while e:
        if e & 1:
            result *= base
        base *= base
        e >>= 1
    return result if n >= 0 else 1 / result
```

### Complexity

- **Time:** $O(\log n)$.
- **Space:** $O(1)$.

## Approach 2: Recursive fast power

### Intuition

`f(e)`: `f(0) = 1`; `half = f(e // 2)`; result is `half * half`, times an extra `x` when `e` is odd.

### Algorithm

1. Reduce to `e = abs(n)`.
2. `f(e)`: if `e == 0`, return `1`. `half = f(e // 2)`. Return `half * half * (x if e % 2 else 1)`.
3. Return `f(e)` or its reciprocal for negative `n`.

```python
def myPow(x: float, n: int) -> float:
    def f(e):
        if e == 0:
            return 1.0
        half = f(e // 2)
        return half * half * (x if e % 2 else 1.0)

    e = abs(n)
    result = f(e)
    return result if n >= 0 else 1 / result
```

### Complexity

- **Time:** $O(\log n)$.
- **Space:** $O(\log n)$ recursion.

## Common Pitfalls

- **`abs(n)` overflow in fixed-width languages.** `-2^31` has no positive counterpart in 32-bit int; cast to a wider type first. Python integers are unbounded, so `abs(n)` is safe here.
- **Forgetting to invert for negative `n`.** Compute `x^|n|`, then take the reciprocal.
- **`n == 0`.** Must return `1.0` for any `x` (including the recursion's base case).
- **Multiplying in `base` after the last useful step.** Harmless for correctness, but ordering matters: test the bit *before* squaring `base`.

## The keystone

Halving the exponent and squaring the base is the logarithmic-time template for any associative operation raised to a power — matrix exponentiation (Fibonacci in $O(\log n)$), modular exponentiation in cryptography, all use this exact loop.
