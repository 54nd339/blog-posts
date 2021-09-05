---
title: Remove Nth Node From End of List - A Gap of N
description: Deleting the n-th node from the end in one pass — hold two pointers exactly n apart, and when the front reaches the end the back sits on the node before the target.
date: 2021-09-05
draft: false
slug: /dsa/remove-nth-node-from-end-of-list
tags:
  - Competitive Programming
  - Linked List
  - Two Pointers
---

"From the end" in a singly linked list normally needs the length. The one-pass trick: start a second pointer `n` nodes behind the first, then move both together — when the front falls off the end, the back is `n` from the end, i.e. just before the node to delete.

## Description

Given the head of a linked list, remove the `n`-th node from the end and return the head.

**Example**

```
Input:  head = [1, 2, 3, 4, 5], n = 2
Output: [1, 2, 3, 5]

Input:  head = [1], n = 1
Output: []
```

**Constraints**

- The list has `1` to `30` nodes.
- $1 \le n \le \text{list length}$
- $0 \le \text{Node.val} \le 100$

## Prerequisites

- The dummy-head idiom for safe head deletion.
- Two pointers held at a fixed gap.

## Approach 1: Brute Force

### Intuition

Store all nodes, index the one to remove as `len - n`, and splice around it.

### Algorithm

1. Collect nodes into an array.
2. `idx = len(nodes) - n`. If `idx == 0`, return `head.next`.
3. Otherwise `nodes[idx - 1].next = nodes[idx].next`; return `head`.

```python
def removeNthFromEnd(head, n):
    nodes = []
    node = head
    while node:
        nodes.append(node)
        node = node.next
    idx = len(nodes) - n
    if idx == 0:
        return head.next
    nodes[idx - 1].next = nodes[idx].next
    return head
```

### Complexity

- **Time:** $O(L)$.
- **Space:** $O(L)$.

## Approach 2: Iteration (Two Pass)

### Intuition

First pass counts the length `L`; second pass walks `L - n` steps to the node before the target.

### Algorithm

1. Count `L`.
2. If `L - n == 0`, return `head.next`.
3. Walk a pointer `L - n - 1` steps from `head`; set its `next` to skip one node.
4. Return `head`.

```python
def removeNthFromEnd(head, n):
    L = 0
    node = head
    while node:
        L += 1
        node = node.next
    if L - n == 0:
        return head.next
    node = head
    for _ in range(L - n - 1):
        node = node.next
    node.next = node.next.next
    return head
```

### Complexity

- **Time:** $O(L)$ — two passes.
- **Space:** $O(1)$.

## Approach 3: Recursion

### Intuition

Recurse to the end; count nodes on the way back up. When the counter hits `n`, the current call's node is the target — return its `next` to splice it out.

### Algorithm

1. `rec(node)`: if `node` is `None`, return `(None, 0)`.
2. `(new_next, count) = rec(node.next)`; `node.next = new_next`; `count += 1`.
3. If `count == n`, return `(node.next, count)` (drop `node`); else return `(node, count)`.
4. Call `rec(head)` and return its node.

```python
def removeNthFromEnd(head, n):
    def rec(node):
        if not node:
            return None, 0
        new_next, count = rec(node.next)
        node.next = new_next
        count += 1
        if count == n:
            return node.next, count
        return node, count
    return rec(head)[0]
```

### Complexity

- **Time:** $O(L)$.
- **Space:** $O(L)$ — call stack.

## Approach 4: Two Pointers

### Intuition

Put `left` at a dummy before `head` and `right` at `head`. Advance `right` `n` steps. Now move both until `right` is `None`: `left` is exactly on the node before the target.

### Algorithm

1. `dummy = ListNode(0, head)`; `left = dummy`; `right = head`.
2. Advance `right` `n` times.
3. While `right`: `left = left.next`, `right = right.next`.
4. `left.next = left.next.next`; return `dummy.next`.

```python
def removeNthFromEnd(head, n):
    dummy = ListNode(0, head)
    left, right = dummy, head
    for _ in range(n):
        right = right.next
    while right:
        left = left.next
        right = right.next
    left.next = left.next.next
    return dummy.next
```

### Complexity

- **Time:** $O(L)$ — a single pass.
- **Space:** $O(1)$.

## Common Pitfalls

- **Deleting the head without a dummy.** When `n` equals the list length, the target *is* the head; the dummy node makes `left.next = left.next.next` handle it uniformly.
- **Advancing `right` `n + 1` instead of `n` steps.** With `left` at the dummy, `right` should move `n` steps so the final gap leaves `left` on the predecessor. Test with `n = 1` (delete tail) and `n = length` (delete head).
- **Off-by-one in the two-pass walk.** You want the node *before* the target, so `L - n - 1` steps from `head`.

## The keystone

A fixed gap between two pointers converts "the k-th from the end" into "when the leader ends, the follower is where you want it" — one pass, no length count. The same fast/slow-with-an-offset idea powers [Linked List Cycle](/citadel/dsa/linked-list-cycle) and middle-finding in [Reorder List](/citadel/dsa/reorder-list).
