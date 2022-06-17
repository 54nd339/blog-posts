---
title: Plus One - Ripple the Carry From the Last Digit
description: Incrementing a number stored as a digit array — walk from the last digit, turning 9s into 0s until a digit under 9 absorbs the carry, and prepend a 1 if every digit was a 9.
date: 2022-06-17
draft: false
slug: /dsa/plus-one
tags:
  - Competitive Programming
  - Math & Geometry
  - Arrays
---

Adding one only affects a run of trailing 9s. Walk backwards: each 9 becomes 0 and the carry continues; the first digit below 9 gets `+1` and you are done. If you fall off the front, every digit was a 9, so the answer is `1` followed by all zeros.

## Description

Given a non-empty array `digits` representing a non-negative integer (most significant digit first, no leading zeros except the number 0 itself), increment it by one and return the resulting digit array.

**Example**

```
Input:  digits = [1,2,3]
Output: [1,2,4]

Input:  digits = [9,9,9]
Output: [1,0,0,0]
```

**Constraints**

- $1 \le \text{digits.length} \le 100$
- $0 \le \text{digits}[i] \le 9$; no leading zeros.

## Prerequisites

- Right-to-left carry propagation.
- The all-nines special case needing one extra leading digit.

## Approach 1: Backward carry

### Intuition

From the last index toward the first: if the digit is less than 9, increment it and return. If it is 9, set it to 0 and continue. If the loop completes, prepend `1`.

### Algorithm

1. For `i` from `len(digits) - 1` down to `0`: if `digits[i] < 9`, `digits[i] += 1`, return `digits`. Else `digits[i] = 0`.
2. Return `[1] + digits`.

```python
def plusOne(digits: list[int]) -> list[int]:
    for i in range(len(digits) - 1, -1, -1):
        if digits[i] < 9:
            digits[i] += 1
            return digits
        digits[i] = 0
    return [1] + digits
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$ (or $O(n)$ only in the all-nines case).

## Approach 2: Explicit carry variable

### Intuition

Standard addition with `carry = 1` injected at the least significant digit; stop early once the carry is absorbed.

### Algorithm

1. `carry = 1`.
2. For `i` from the end: `total = digits[i] + carry`; `digits[i] = total % 10`; `carry = total // 10`; if `carry == 0`, break.
3. If `carry`, prepend it.

```python
def plusOne(digits: list[int]) -> list[int]:
    carry = 1
    for i in range(len(digits) - 1, -1, -1):
        total = digits[i] + carry
        digits[i] = total % 10
        carry = total // 10
        if carry == 0:
            break
    if carry:
        digits.insert(0, carry)
    return digits
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$ amortised.

## Common Pitfalls

- **Not handling all-nines.** `[9,9]` → `[1,0,0]`; the prepend step is essential.
- **Continuing the loop after absorbing the carry.** Return (or break) as soon as a digit is incremented without carry — otherwise you keep processing needlessly, and in the carry-variable version you might mis-handle later digits.
- **Converting to `int` and back.** Works for the given sizes but defeats the point and overflows in fixed-width languages for a 100-digit input.
- **Mutating vs returning a new list.** Either is accepted here; be consistent.

## The keystone

Big-integer arithmetic on digit arrays is carry propagation from least to most significant, with a possible extra digit at the top. [Multiply Strings](/citadel/dsa/multiply-strings) is the same carry mechanics scaled up to a full multiplication.
