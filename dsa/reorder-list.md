---
title: Reorder List - Split, Reverse, Zip
description: Interleaving a list as first, last, second, second-last — an array of node references does it in O(n) space, but finding the middle, reversing the back half, and merging does it in O(1).
date: 2021-08-30
draft: false
slug: /dsa/reorder-list
tags:
  - Competitive Programming
  - Linked List
  - Two Pointers
---

The target order `L0, Ln, L1, Ln-1, L2, Ln-2, ...` is exactly what you get by taking the front half in order and the back half in reverse, one node at a time. So: find the middle, reverse the second half, then zip the two halves together.

## Description

Given the head of a singly linked list `L0 → L1 → ... → Ln-1 → Ln`, reorder it in place to `L0 → Ln → L1 → Ln-1 → L2 → ...`. Only node pointers may be changed, not values.

**Example**

```
Input:  head = [1, 2, 3, 4]
Output: [1, 4, 2, 3]

Input:  head = [1, 2, 3, 4, 5]
Output: [1, 5, 2, 4, 3]
```

**Constraints**

- The list has `1` to $5 \times 10^4$ nodes.
- $1 \le \text{Node.val} \le 1000$

## Prerequisites

- [Reverse a Linked List](/citadel/dsa/reverse-linked-list).
- Fast/slow pointers to find the middle.

## Approach 1: Brute Force

### Intuition

Put every node reference in an array; then two pointers from the ends rebuild the links in the reordered sequence.

### Algorithm

1. Traverse the list, appending each node to `nodes`.
2. `i = 0`, `j = len(nodes) - 1`. While `i < j`: link `nodes[i] → nodes[j]`, then `nodes[j] → nodes[i + 1]`; `i += 1`, `j -= 1`.
3. Set `nodes[i].next = None` to terminate.

```python
def reorderList(head) -> None:
    if not head:
        return
    nodes = []
    node = head
    while node:
        nodes.append(node)
        node = node.next
    i, j = 0, len(nodes) - 1
    while i < j:
        nodes[i].next = nodes[j]
        i += 1
        if i == j:
            break
        nodes[j].next = nodes[i]
        j -= 1
    nodes[i].next = None
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ for the array of references.

## Approach 2: Recursion

### Intuition

Recurse to the tail while a front pointer stays at the head. On the way back up, splice the current tail node in right after the front node, then advance the front.

### Algorithm

1. `rec(front, cur)`: recurse on `cur.next` until `cur` is `None`.
2. On unwind, if `front` has met or passed `cur`, set `cur.next = None` and signal stop.
3. Otherwise save `front.next`, set `front.next = cur`, `cur.next = saved`, and continue with the new front.

```python
def reorderList(head) -> None:
    stop = False

    def rec(front, cur):
        nonlocal stop
        if not cur:
            return front
        front = rec(front, cur.next)
        if stop:
            return front
        if front is cur or front.next is cur:
            cur.next = None
            stop = True
            return front
        nxt = front.next
        front.next = cur
        cur.next = nxt
        return nxt

    rec(head, head.next)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ — recursion depth.

## Approach 3: Reverse And Merge

### Intuition

Find the middle with slow/fast pointers, reverse the second half in place, then merge the two halves node by node.

### Algorithm

1. `slow`, `fast` from `head`; advance `slow` by 1 and `fast` by 2 until `fast` (or `fast.next`) is `None`. `slow` is the start of the second half.
2. Reverse from `slow.next` onward; cut `slow.next = None`. Call the reversed head `second`.
3. `first = head`. While `second`: splice one node from `first` then one from `second`, advancing both.

```python
def reorderList(head) -> None:
    slow, fast = head, head.next
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next

    second = slow.next
    slow.next = None
    prev = None
    while second:
        nxt = second.next
        second.next = prev
        prev = second
        second = nxt
    second = prev

    first = head
    while second:
        f_next, s_next = first.next, second.next
        first.next = second
        second.next = f_next
        first, second = f_next, s_next
```

### Complexity

- **Time:** $O(n)$ — find middle, reverse, merge, each linear.
- **Space:** $O(1)$.

## Common Pitfalls

- **Not cutting the first half's tail.** After splitting, `slow.next` (or the first half's last node) must be set to `None`, or the merge walks into stale links and can loop.
- **Middle for even vs odd length.** Starting `fast = head.next` puts `slow` at the end of the first half, so the first half is the same length or one longer than the second — which is what the interleave wants.
- **Merge termination.** Drive the merge by `while second`, since the second (reversed) half is never longer than the first.

## The keystone

"Split, transform one part, recombine" is a recurring linked-list shape: middle-finding + in-place reversal + a merge. Each piece is a standalone skill ([Reverse a Linked List](/citadel/dsa/reverse-linked-list), [Linked List Cycle](/citadel/dsa/linked-list-cycle)'s pointer trick, [Merge Two Sorted Lists](/citadel/dsa/merge-two-sorted-lists)) composed here.
