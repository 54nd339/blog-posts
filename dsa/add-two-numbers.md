---
title: Add Two Numbers - Elementary Addition, Digit by Digit
description: Summing two numbers stored as reversed digit lists — walk both lists in lockstep with a carry, and keep going while either list has digits or a carry remains.
date: 2021-09-08
draft: false
slug: /dsa/add-two-numbers
tags:
  - Competitive Programming
  - Linked List
  - Math
---

The digits are stored least-significant first, which is exactly the order you add by hand: units, then tens, then hundreds, carrying as you go. The loop condition is the whole subtlety — it must survive one list ending before the other, and a final carry.

## Description

Given two non-empty linked lists representing non-negative integers with digits stored in reverse order (one digit per node), add them and return the sum as a linked list in the same format.

**Example**

```
Input:  l1 = [2,4,3], l2 = [5,6,4]      # 342 + 465
Output: [7,0,8]                          # 807

Input:  l1 = [9,9,9,9,9,9,9], l2 = [9,9,9,9]
Output: [8,9,9,9,0,0,0,1]
```

**Constraints**

- Each list has `1` to `100` nodes; each `Node.val` is `0`–`9`.
- No leading zeros except the number `0` itself.

## Prerequisites

- The dummy-head idiom.
- Carry propagation in base-10 addition.

## Approach 1: Recursion

### Intuition

Add the two current digits plus an incoming carry; the result node's `next` is the recursive sum of the rest with the new carry.

### Algorithm

1. `add(l1, l2, carry)`: if `l1`, `l2`, and `carry` are all empty/zero, return `None`.
2. `total = (l1.val if l1 else 0) + (l2.val if l2 else 0) + carry`.
3. Create a node with `total % 10`; its `next` is `add(l1.next, l2.next, total // 10)`.

```python
def addTwoNumbers(l1, l2):
    def add(a, b, carry):
        if not a and not b and not carry:
            return None
        total = (a.val if a else 0) + (b.val if b else 0) + carry
        node = ListNode(total % 10)
        node.next = add(a.next if a else None, b.next if b else None, total // 10)
        return node
    return add(l1, l2, 0)
```

### Complexity

- **Time:** $O(\max(m, n))$.
- **Space:** $O(\max(m, n))$ — recursion plus output.

## Approach 2: Iteration

### Intuition

Walk both lists with a `tail` pointer on a dummy node and a running `carry`. Loop while either list has a node or the carry is non-zero.

### Algorithm

1. `dummy = ListNode()`, `tail = dummy`, `carry = 0`.
2. While `l1` or `l2` or `carry`:
   - `total = (l1.val if l1 else 0) + (l2.val if l2 else 0) + carry`.
   - `carry, digit = divmod(total, 10)`.
   - `tail.next = ListNode(digit)`; `tail = tail.next`.
   - Advance `l1` and `l2` if present.
3. Return `dummy.next`.

```python
def addTwoNumbers(l1, l2):
    dummy = ListNode()
    tail = dummy
    carry = 0
    while l1 or l2 or carry:
        total = (l1.val if l1 else 0) + (l2.val if l2 else 0) + carry
        carry, digit = divmod(total, 10)
        tail.next = ListNode(digit)
        tail = tail.next
        l1 = l1.next if l1 else None
        l2 = l2.next if l2 else None
    return dummy.next
```

### Complexity

- **Time:** $O(\max(m, n))$.
- **Space:** $O(1)$ extra beyond the output.

## Common Pitfalls

- **Stopping when both lists end.** `999 + 1` needs one more node for the final carry; the loop condition must include `or carry`.
- **Assuming equal lengths.** Treat a missing node as `0`.
- **Forgetting the carry can only ever be `0` or `1`** — two digits plus a carry is at most `19` — so `total // 10` is enough; no loop needed per digit.

## The keystone

Digit-list arithmetic is just the grade-school algorithm with `divmod` doing the carry. The dummy head plus a "while inputs remain or carry remains" loop is the template for any pairwise list-building traversal, including [Merge Two Sorted Lists](/citadel/dsa/merge-two-sorted-lists).
