---
title: Happy Number - Detect the Cycle in Digit-Square Sums
description: Whether repeatedly summing the squares of a number's digits reaches 1 — the sequence either hits 1 or loops, so it is cycle detection by a hash set or Floyd's two pointers.
date: 2022-06-17
draft: false
slug: /dsa/happy-number
tags:
  - Competitive Programming
  - Math & Geometry
  - Hash Table
---

Replacing `n` with the sum of the squares of its digits eventually either reaches 1 (happy) or enters a cycle that never contains 1 (unhappy). Bounded values mean a repeat is guaranteed, so this is exactly linked-list cycle detection: a `seen` set, or Floyd's slow/fast pointers.

## Description

A number is happy if repeatedly replacing it with the sum of the squares of its digits eventually yields 1. Return `true` if `n` is happy.

**Example**

```
Input:  n = 19
Output: true   (19 → 82 → 68 → 100 → 1)

Input:  n = 2
Output: false
```

**Constraints**

- $1 \le n \le 2^{31} - 1$

## Prerequisites

- The digit-square-sum step and why the values stay bounded.
- Cycle detection: hash set, or Floyd's tortoise and hare.

## Approach 1: Hash set

### Intuition

Iterate the transformation, storing every value seen. If you reach 1, happy. If you revisit a value, you are in a loop — unhappy.

### Algorithm

1. `seen = set()`.
2. While `n != 1` and `n not in seen`: add `n` to `seen`; `n = sum of d*d for each digit d`.
3. Return `n == 1`.

```python
def isHappy(n: int) -> bool:
    def next_num(x):
        total = 0
        while x:
            x, d = divmod(x, 10)
            total += d * d
        return total

    seen = set()
    while n != 1 and n not in seen:
        seen.add(n)
        n = next_num(n)
    return n == 1
```

### Complexity

- **Time:** $O(\log n)$ per step; the number of steps is bounded by a constant (values quickly fall below ~243).
- **Space:** $O(1)$ effectively — the set holds a bounded number of small values.

## Approach 2: Floyd's cycle detection

### Intuition

Advance `slow` one step and `fast` two steps per iteration. If they meet at 1, happy. If they meet anywhere else, there is a cycle without 1 — unhappy.

### Algorithm

1. `slow = n`, `fast = next_num(n)`.
2. While `fast != 1` and `slow != fast`: `slow = next_num(slow)`; `fast = next_num(next_num(fast))`.
3. Return `fast == 1`.

```python
def isHappy(n: int) -> bool:
    def next_num(x):
        total = 0
        while x:
            x, d = divmod(x, 10)
            total += d * d
        return total

    slow, fast = n, next_num(n)
    while fast != 1 and slow != fast:
        slow = next_num(slow)
        fast = next_num(next_num(fast))
    return fast == 1
```

### Complexity

- **Time:** $O(\log n)$ per step, constant number of steps.
- **Space:** $O(1)$.

## Common Pitfalls

- **Assuming non-happy numbers diverge.** They do not — digit-square sums are bounded (for any `n`, the next value is at most `81 * digits`), so the sequence always cycles; you are detecting that cycle, not an overflow.
- **Floyd's initialisation.** Start `fast` one step ahead (`next_num(n)`), or the `slow != fast` check trips immediately.
- **Recomputing digits inefficiently.** `divmod(x, 10)` in a loop is fine; converting to a string each step is also acceptable at this scale.
- **Returning `n != 1`.** The happy condition is reaching 1 — return `n == 1` (set version) or `fast == 1` (Floyd).

## The keystone

Any deterministic "replace x with f(x)" over a bounded domain must eventually repeat, which makes it a cycle-detection problem — solvable with a set or, in `O(1)` space, Floyd's pointers, exactly as in [Linked List Cycle](/citadel/dsa/linked-list-cycle).
