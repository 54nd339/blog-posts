---
title: Reverse Integer - Pop Digits, Push Digits, Watch the Bound
description: Reversing the digits of a signed 32-bit integer and returning zero on overflow — peel the last digit with mod ten, append it to the result, and check the 32-bit limit before each append.
date: 2022-08-20
draft: false
slug: /dsa/reverse-integer
tags:
  - Competitive Programming
  - Bit Manipulation
  - Math
---

Peel digits off the end of `x` with `% 10` and `// 10`, and build `res = res * 10 + digit`. The only subtlety is the signed 32-bit range: before each `res * 10 + digit`, verify the multiplication and addition will not exceed `2^31 - 1` (or drop below `-2^31`), and return 0 if it would.

## Description

Given a signed 32-bit integer `x`, return `x` with its digits reversed. If reversing causes the value to fall outside `[-2^31, 2^31 - 1]`, return `0`. Assume the environment does not allow 64-bit integers.

**Example**

```
Input:  x = 123
Output: 321

Input:  x = -123
Output: -321

Input:  x = 120
Output: 21
```

**Constraints**

- $-2^{31} \le x \le 2^{31} - 1$

## Prerequisites

- Digit peeling with `divmod` and sign handling.
- The overflow pre-check against `INT_MAX // 10` / `INT_MIN // 10`.

## Approach 1: Digit peel with an overflow guard

### Intuition

Work with the sign separated out. Each step, before `res = res * 10 + d`, check whether `res` already exceeds `INT_MAX // 10`, or equals it with `d` beyond the last allowed digit. If so, return 0.

### Algorithm

1. `INT_MAX = 2**31 - 1`, `INT_MIN = -2**31`.
2. `sign = -1 if x < 0 else 1`; `x = abs(x)`; `res = 0`.
3. While `x`: `x, d = divmod(x, 10)`. If `res > INT_MAX // 10` or (`res == INT_MAX // 10` and `d > 7`), return `0`. `res = res * 10 + d`.
4. Return `sign * res`.

```python
def reverse(x: int) -> int:
    INT_MAX = 2**31 - 1
    sign = -1 if x < 0 else 1
    x = abs(x)
    res = 0
    while x:
        x, d = divmod(x, 10)
        if res > INT_MAX // 10 or (res == INT_MAX // 10 and d > 7):
            return 0
        res = res * 10 + d
    return sign * res
```

### Complexity

- **Time:** $O(\log_{10} x)$ — the number of digits.
- **Space:** $O(1)$.

## Approach 2: Build then range-check

### Intuition

If wider integers *are* available (or in Python, always), just reverse via string or arithmetic and check the final value against the 32-bit bounds.

### Algorithm

1. `sign = -1 if x < 0 else 1`; `rev = int(str(abs(x))[::-1]) * sign`.
2. Return `rev` if `-2**31 <= rev <= 2**31 - 1` else `0`.

```python
def reverse(x: int) -> int:
    sign = -1 if x < 0 else 1
    rev = sign * int(str(abs(x))[::-1])
    return rev if -2**31 <= rev <= 2**31 - 1 else 0
```

### Complexity

- **Time:** $O(\log_{10} x)$.
- **Space:** $O(\log_{10} x)$ for the string.

## Common Pitfalls

- **Checking overflow *after* `res * 10 + d`.** In a true 32-bit environment the overflow has already happened. Check *before* the update.
- **The magic digit `7` / `8`.** `INT_MAX` ends in `7`, `INT_MIN` ends in `8`. When `res == INT_MAX // 10`, only `d <= 7` is safe (for negatives, `d <= 8`); folding both into `d > 7` works because a reversed valid input rarely hits the negative edge, but handle it explicitly if strict.
- **`abs(INT_MIN)` in fixed-width languages.** It overflows; process digits on the negative value or use a wider accumulator.
- **Losing trailing zeros silently — which is correct.** `120` reverses to `21`; that is expected, not a bug.

## The keystone

Reversing a number is `res = res * 10 + (x % 10)` with `x //= 10`, and the interview's real content is the pre-multiplication overflow check against `INT_MAX // 10`. This closes [Bit Manipulation](/citadel/dsa/bit-manipulation) and the NeetCode 150 — the same digit-peel loop drove [Plus One](/citadel/dsa/plus-one) and [Happy Number](/citadel/dsa/happy-number) back in Math &amp; Geometry.
