---
title: Linked List Cycle - Floyd's Tortoise and Hare
description: Detecting whether a linked list loops — a visited-set is O(n) space, but a slow pointer and a fast pointer moving at different speeds must collide inside any cycle, in O(1) space.
date: 2021-09-14
draft: false
slug: /dsa/linked-list-cycle
tags:
  - Competitive Programming
  - Linked List
  - Two Pointers
---

If the list has a cycle, a pointer moving two steps per tick gains one step per tick on a pointer moving one step, so it eventually laps the slow pointer and they land on the same node. If there is no cycle, the fast pointer runs off the end.

## Description

Given the head of a linked list, return `true` if it contains a cycle. A cycle exists if some node can be revisited by following `next` pointers.

**Example**

```
Input:  head = [3,2,0,-4], tail connects to index 1
Output: true

Input:  head = [1], no cycle
Output: false
```

**Constraints**

- The list has `0` to $10^4$ nodes.
- $-10^5 \le \text{Node.val} \le 10^5$

## Prerequisites

- Fast/slow (tortoise and hare) pointers.
- The observation that two pointers with a speed difference of 1 must meet in a loop.

## Approach 1: Hash Set

### Intuition

Walk the list, remembering every node visited. Revisiting one means a cycle.

### Algorithm

1. Empty set `seen`.
2. For each node from `head`: if it is in `seen`, return `true`; add it; advance.
3. Reaching `None` means return `false`.

```python
def hasCycle(head) -> bool:
    seen = set()
    node = head
    while node:
        if node in seen:
            return True
        seen.add(node)
        node = node.next
    return False
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ — the set.

## Approach 2: Fast and Slow Pointers

### Intuition

Move `slow` by one and `fast` by two each step. In a cycle, `fast` closes the gap to `slow` by one node per step and must eventually coincide. Without a cycle, `fast` hits `None`.

### Algorithm

1. `slow = fast = head`.
2. While `fast` and `fast.next`:
   - `slow = slow.next`; `fast = fast.next.next`.
   - If `slow is fast`, return `true`.
3. Return `false`.

```python
def hasCycle(head) -> bool:
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:
            return True
    return False
```

### Complexity

- **Time:** $O(n)$ — once `slow` enters the cycle, `fast` catches it within one cycle length.
- **Space:** $O(1)$.

## Common Pitfalls

- **Loop guard.** You must check both `fast` and `fast.next` before `fast.next.next`, or a non-cyclic even-length list throws `AttributeError`.
- **Comparing values instead of identity.** Use `slow is fast` (node identity); `slow.val == fast.val` gives false positives on lists with repeated values.
- **Starting `fast` at `head.next`.** It works but changes the initial condition; starting both at `head` and checking *after* moving is the cleaner invariant.

## The keystone

Floyd's cycle detection turns a "have I been here?" question into a collision between two speeds — $O(1)$ space, no bookkeeping. Extending it (reset one pointer to the head, then advance both by one) finds the cycle's *entry* node, which is exactly how [Find the Duplicate Number](/citadel/dsa/find-the-duplicate-number) works.
