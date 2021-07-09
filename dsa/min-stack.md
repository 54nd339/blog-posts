---
title: Min Stack - Carry the Minimum Alongside Each Element
description: A stack whose minimum is available in O(1) — scanning on every query is O(n), a parallel min-stack makes it constant, and an encoded-delta trick does it with one stack.
date: 2021-07-09
draft: false
slug: /dsa/min-stack
tags:
  - Competitive Programming
  - Stack
  - Design
---

The minimum has a *history* — pop the current minimum and the previous one must resurface. One scalar cannot hold that; the trick is to precompute the running minimum at push time.

## Description

Design a stack supporting `push(val)`, `pop()`, `top()`, and `getMin()`, with every operation in $O(1)$.

**Example**

```
MinStack ops:  push(-2), push(0), push(-3), getMin() -> -3,
               pop(), top() -> 0, getMin() -> -2
```

**Constraints**

- $-2^{31} \le \text{val} < 2^{31}$
- `pop`, `top`, `getMin` are only called on a non-empty stack.
- At most $3 \times 10^4$ calls.

## Prerequisites

- [Stack](/citadel/dsa/stack) operations.
- The idea of augmenting each entry with an aggregate-so-far.

## Approach 1: Brute Force

### Intuition

Keep a plain list; compute the minimum on demand by scanning.

### Algorithm

1. `push`/`pop`/`top` operate on a list directly.
2. `getMin`: pop everything while tracking the minimum, then push it all back (or just `min(list)`).

```python
class MinStack:
    def __init__(self):
        self.data = []

    def push(self, val: int) -> None:
        self.data.append(val)

    def pop(self) -> None:
        self.data.pop()

    def top(self) -> int:
        return self.data[-1]

    def getMin(self) -> int:
        return min(self.data)
```

### Complexity

- **Time:** $O(1)$ for `push`/`pop`/`top`; $O(n)$ for `getMin`.
- **Space:** $O(n)$.

## Approach 2: Two Stacks

### Intuition

Keep a second stack whose top is always the minimum of everything currently in the main stack.

### Algorithm

1. On `push(val)`: append `val` to the value stack; append `min(val, mins[-1] if mins else val)` to the min stack.
2. On `pop`: pop both stacks together.
3. `top`: value stack's top. `getMin`: min stack's top.

```python
class MinStack:
    def __init__(self):
        self.data = []
        self.mins = []

    def push(self, val: int) -> None:
        self.data.append(val)
        self.mins.append(val if not self.mins else min(val, self.mins[-1]))

    def pop(self) -> None:
        self.data.pop()
        self.mins.pop()

    def top(self) -> int:
        return self.data[-1]

    def getMin(self) -> int:
        return self.mins[-1]
```

### Complexity

- **Time:** $O(1)$ for all operations.
- **Space:** $O(n)$ — two stacks of equal height.

## Approach 3: One Stack

### Intuition

Store, for each element, its *difference from the minimum at the time it was pushed*, plus a single `min` variable. A negative stored value signals "this element became the new minimum", and it encodes how to restore the previous one.

### Algorithm

1. Keep a stack and a scalar `min`.
2. On `push(val)`: if the stack is empty, push `0` and set `min = val`. Else push `val - min`; if `val < min`, set `min = val`.
3. On `pop`: let `diff = stack.pop()`; if `diff < 0`, restore `min = min - diff`.
4. `top`: if `stack[-1] >= 0`, return `min + stack[-1]`; else return `min`.
5. `getMin`: return `min`.

```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.min = 0

    def push(self, val: int) -> None:
        if not self.stack:
            self.stack.append(0)
            self.min = val
        else:
            self.stack.append(val - self.min)
            if val < self.min:
                self.min = val

    def pop(self) -> None:
        diff = self.stack.pop()
        if diff < 0:
            self.min -= diff

    def top(self) -> int:
        diff = self.stack[-1]
        return self.min + diff if diff > 0 else self.min

    def getMin(self) -> int:
        return self.min
```

### Complexity

- **Time:** $O(1)$ for all operations.
- **Space:** $O(n)$ — one stack, no parallel structure.

## Common Pitfalls

- **Approach 2: not popping both stacks together.** If `pop` touches only the value stack, the two desync and `getMin` reports a stale minimum.
- **Approach 3: sign conventions.** `top` must return `min` (not `min + diff`) when `diff <= 0`, because a non-positive delta means this element *is* the minimum it was pushed against.
- **A space micro-optimisation of Approach 2** — pushing to the min stack only when `val <= current_min`, popping only on equality — requires `<=`, not `<`, or two equal minimums are recorded once and popping one loses it while an equal value remains.

## The keystone

When an aggregate (min, max, sum, gcd) must be $O(1)$ on a stack, store the aggregate-so-far *with* each element so popping restores the previous aggregate automatically. The same "augment each node with a running summary" idea appears in monotonic stacks and segment trees.
