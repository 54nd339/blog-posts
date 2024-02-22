---
title: Minimum Stack and Minimum Queue - O(1) Running Extremes
description: Keeping the minimum of a stack in O(1) by pairing each element with a running min, extending it to a queue with two such stacks, and the monotone-deque form that drives sliding-window minimum.
date: 2024-02-22
draft: false
slug: /cp/min-stack-min-queue
tags:
  - Competitive Programming
  - Data Structures
  - Stacks and Queues
---

A stack or queue plus "what is the minimum element currently inside?" — answered in $O(1)$ per operation. The stack version is a one-line trick; the queue version builds on it, and its monotone-deque cousin is the standard tool for sliding-window minimum and a family of DP speed-ups.

## The problem

Support `push`, `pop` (stack) or `push`/`pop` at opposite ends (queue), plus `get_min` returning the smallest element currently held — all in $O(1)$ amortised.

Example (queue): push $3, 1, 4, 1, 5$; `get_min` = $1$. Pop $3$; `get_min` still $1$. Pop $1$; `get_min` still $1$. Pop $4$; `get_min` still $1$.

## Minimum stack

Store, alongside each value, the minimum of the stack *up to and including* it. On `push(x)`, record `min(x, current_min)`. On `pop`, both come off together. `get_min` reads the top's recorded minimum.

```python
class MinStack:
    def __init__(self):
        self.st = []                    # (value, min_so_far)

    def push(self, x: int) -> None:
        m = x if not self.st else min(x, self.st[-1][1])
        self.st.append((x, m))

    def pop(self) -> int:
        return self.st.pop()[0]

    def get_min(self) -> int:
        return self.st[-1][1]
```

Every operation is $O(1)$ worst case; extra space is one integer per element.

## Minimum queue, version 1: two min-stacks

A queue is two stacks — `in` for pushes, `out` for pops. When `out` is empty, dump `in` into it (reversing order). Each element is moved at most once, so `pop` is $O(1)$ amortised. The queue minimum is the smaller of the two stacks' minima.

```python
class MinQueue:
    def __init__(self):
        self.ins = MinStack()
        self.outs = MinStack()

    def push(self, x: int) -> None:
        self.ins.push(x)

    def pop(self) -> int:
        if not self.outs.st:
            while self.ins.st:
                self.outs.push(self.ins.pop())
        return self.outs.pop()

    def get_min(self) -> int:
        if not self.ins.st:
            return self.outs.get_min()
        if not self.outs.st:
            return self.ins.get_min()
        return min(self.ins.get_min(), self.outs.get_min())
```

## Minimum queue, version 2: monotone deque

Keep a deque of values (or indices) in **non-decreasing** order. On `push(x)`, pop from the back everything $> x$ (they can never be the minimum while $x$ is in the queue), then append $x$. The front is always the current minimum. On `pop(v)` of the queue's front element, if it equals the deque front, pop that too.

```python
from collections import deque

class MinQueueDeque:
    def __init__(self):
        self.q = deque()            # actual queue values
        self.mono = deque()         # non-decreasing

    def push(self, x: int) -> None:
        self.q.append(x)
        while self.mono and self.mono[-1] > x:
            self.mono.pop()
        self.mono.append(x)

    def pop(self) -> int:
        x = self.q.popleft()
        if self.mono and self.mono[0] == x:
            self.mono.popleft()
        return x

    def get_min(self) -> int:
        return self.mono[0]
```

Each element is appended and removed from `mono` once, so all operations are $O(1)$ amortised.

## Sliding-window minimum

The direct application: for every window of width $k$ in an array, report the minimum. Push indices as the window advances, pop the front when it falls out of the window, read `mono[0]`. $O(n)$ total.

```python
def sliding_min(a: list[int], k: int) -> list[int]:
    dq, out = deque(), []
    for i, x in enumerate(a):
        while dq and a[dq[-1]] >= x:
            dq.pop()
        dq.append(i)
        if dq[0] <= i - k:
            dq.popleft()
        if i >= k - 1:
            out.append(a[dq[0]])
    return out
```

## Complexity

- **Min stack:** $O(1)$ worst case per operation.
- **Min queue (both versions):** $O(1)$ amortised per operation.
- **Space:** $O(n)$.

## Common pitfalls

- **Deque holding stale indices.** For sliding window, store *indices*, not values, so you can tell when the front leaves the window.
- **`>` vs `>=` when evicting the back.** Use `>=` to also drop equal elements if you want the deque to hold distinct positions of the min; either works for the min value itself, but be consistent.
- **`get_min` on an empty queue.** Version 1 must check which stack is non-empty; guard against both empty.
- **Assuming worst-case $O(1)$ for the queue.** A single `pop` can trigger an $O(n)$ transfer; it is only amortised $O(1)$. Fine for total complexity, not for a hard per-op bound.
- **Max instead of min.** Flip the comparator, or negate values — don't half-convert.

## The keystone

Pair each stacked element with the running minimum and `get_min` is $O(1)$; glue two such stacks into a queue, or keep a non-decreasing deque whose front is the answer. That monotone deque, walked across an array, is sliding-window minimum in $O(n)$ and the mechanism behind several $O(n)$ DP optimisations.
