---
title: Daily Temperatures - A Stack of Days Still Waiting for Warmth
description: For each day, how many days until it gets warmer — scanning forward from every day is quadratic, but a stack of unresolved indices gives each day an O(1) amortised resolution.
date: 2021-07-15
draft: false
slug: /dsa/daily-temperatures
tags:
  - Competitive Programming
  - Stack
  - Monotonic Stack
---

The brute force asks, for each day, "when is the next warmer day?" and scans forward. The monotonic stack flips it: each new day *resolves* every earlier day it is warmer than.

## Description

Given `temperatures`, return an array `answer` where `answer[i]` is the number of days to wait after day `i` for a warmer temperature, or `0` if none exists.

**Example**

```
Input:  temperatures = [73,74,75,71,69,72,76,73]
Output: [1,1,4,2,1,1,0,0]

Input:  temperatures = [30,40,50,60]
Output: [1,1,1,0]
```

**Constraints**

- $1 \le \text{temperatures.length} \le 10^5$
- $30 \le \text{temperatures}[i] \le 100$

## Prerequisites

- Monotonic [stack](/citadel/dsa/stack) for "next greater element".
- The amortised argument: each index pushed and popped once.

## Approach 1: Brute Force

### Intuition

For each day, walk forward until a warmer day.

### Algorithm

1. For each `i`, scan `j = i + 1, i + 2, ...`; the first `j` with `temperatures[j] > temperatures[i]` gives `answer[i] = j - i`.
2. If none, `answer[i] = 0`.

```python
def dailyTemperatures(temperatures: list[int]) -> list[int]:
    n = len(temperatures)
    answer = [0] * n
    for i in range(n):
        for j in range(i + 1, n):
            if temperatures[j] > temperatures[i]:
                answer[i] = j - i
                break
    return answer
```

### Complexity

- **Time:** $O(n^2)$ — a decreasing array makes every scan run to the end.
- **Space:** $O(1)$ beyond the output.

## Approach 2: Stack

### Intuition

Keep a stack of `(temperature, index)` for days whose answer is still unknown, decreasing by temperature. A new day warmer than the top resolves it.

### Algorithm

1. `answer = [0] * n`, empty stack.
2. For each `(i, temp)`:
   - While the stack is non-empty and `temp > stack[-1][0]`: pop `(t, idx)`; set `answer[idx] = i - idx`.
   - Push `(temp, i)`.
3. Return `answer`.

```python
def dailyTemperatures(temperatures: list[int]) -> list[int]:
    n = len(temperatures)
    answer = [0] * n
    stack: list[tuple[int, int]] = []
    for i, temp in enumerate(temperatures):
        while stack and temp > stack[-1][0]:
            t, idx = stack.pop()
            answer[idx] = i - idx
        stack.append((temp, i))
    return answer
```

### Complexity

- **Time:** $O(n)$ — each index pushed once, popped at most once.
- **Space:** $O(n)$ for the stack.

## Approach 3: Dynamic Programming

### Intuition

Fill `answer` right to left. From day `i`, jump forward using already-computed answers: if day `i + 1` is not warmer, hop to `i + 1 + answer[i + 1]` (the next day *that* day pointed to), and so on.

### Algorithm

1. `answer = [0] * n`.
2. For `i` from `n - 2` down to `0`, set `j = i + 1`:
   - While `j < n` and `temperatures[j] <= temperatures[i]`: if `answer[j] == 0`, set `j = n` (dead end); else `j += answer[j]`.
   - If `j < n`, `answer[i] = j - i`.

```python
def dailyTemperatures(temperatures: list[int]) -> list[int]:
    n = len(temperatures)
    answer = [0] * n
    for i in range(n - 2, -1, -1):
        j = i + 1
        while j < n and temperatures[j] <= temperatures[i]:
            if answer[j] == 0:
                j = n
            else:
                j += answer[j]
        if j < n:
            answer[i] = j - i
    return answer
```

### Complexity

- **Time:** $O(n)$ amortised — each hop lands on a strictly warmer day, so the chain from any `i` is short in total.
- **Space:** $O(1)$ beyond the output.

## Common Pitfalls

- **`>=` instead of `>`.** "Warmer" is strict; an equal temperature does not resolve the wait.
- **Storing only temperatures on the stack (Approach 2).** You need the index to compute `i - idx`.
- **Approach 3: treating `answer[j] == 0` as "keep going".** A zero there means day `j` has no warmer day ahead, so neither does `i` through `j` — stop.

## The keystone

A monotonic stack computes "nearest greater/smaller element" for every position in linear time: pushing an element that dominates the top means the top is permanently answered. This is the same machine as [Sliding Window Maximum](/citadel/dsa/sliding-window-maximum)'s deque and the core of [Car Fleet](/citadel/dsa/car-fleet) and [Largest Rectangle in Histogram](/citadel/dsa/largest-rectangle-in-histogram).
