---
title: Reverse Nodes in K-Group - Reverse in Blocks, Stitch the Seams
description: Reversing every consecutive block of k nodes, leaving a trailing remainder untouched — recursion reverses one block and delegates the rest, iteration walks with a group-boundary pointer.
date: 2021-09-22
draft: false
slug: /dsa/reverse-nodes-in-k-group
tags:
  - Competitive Programming
  - Linked List
  - Recursion
---

This is [Reverse a Linked List](/citadel/dsa/reverse-linked-list) done in fixed-size chunks. The extra work is checking a full group of `k` exists before reversing it, and reconnecting each reversed block to the previous block's new tail and the next block's new head.

## Description

Given the head of a linked list, reverse the nodes `k` at a time and return the modified list. If the number of nodes is not a multiple of `k`, leave the final `< k` nodes as they are. Only node links may change.

**Example**

```
Input:  head = [1,2,3,4,5], k = 2
Output: [2,1,4,3,5]

Input:  head = [1,2,3,4,5], k = 3
Output: [3,2,1,4,5]
```

**Constraints**

- The list has `1` to `5000` nodes; $1 \le k \le n$.
- $0 \le \text{Node.val} \le 1000$
- Follow-up: $O(1)$ extra space.

## Prerequisites

- [Reverse a Linked List](/citadel/dsa/reverse-linked-list) (iterative, three-pointer).
- The dummy-head idiom for stitching blocks.

## Approach 1: Recursion

### Intuition

Look `k` nodes ahead. If fewer than `k` remain, return the head unchanged. Otherwise reverse this group, and let recursion handle everything after it; the reversed group's tail (the original head) links to that recursive result.

### Algorithm

1. Walk `k` steps from `head`, counting. If you hit `None` before `k`, return `head`.
2. Recursively process the node after the `k`-th; call it `rest`.
3. Reverse exactly `k` nodes starting at `head`, initialising `prev = rest` so the last node of the group points into the recursively-built remainder.
4. Return the new group head (the `k`-th node).

```python
def reverseKGroup(head, k):
    node = head
    for _ in range(k):
        if not node:
            return head
        node = node.next

    prev = reverseKGroup(node, k)   # 'rest'
    curr = head
    for _ in range(k):
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt
    return prev
```

### Complexity

- **Time:** $O(n)$ — each node reversed once.
- **Space:** $O(n / k)$ — one recursion frame per group.

## Approach 2: Iteration

### Intuition

Keep a `group_prev` pointer at the node just before the current group. Find the `k`-th node of the group; if it does not exist, stop. Reverse the group, then reattach: `group_prev.next` becomes the group's new head, and `group_prev` moves to the group's new tail (the original first node).

### Algorithm

1. `dummy = ListNode(0, head)`; `group_prev = dummy`.
2. Loop:
   - From `group_prev`, walk `k` steps to `kth`. If `kth` is `None`, break.
   - `group_next = kth.next`.
   - Reverse the nodes from `group_prev.next` up to and including `kth`, using `prev = group_next` as the initial predecessor.
   - `tmp = group_prev.next` (old first node, now the group tail); `group_prev.next = kth` (new head); `group_prev = tmp`.
3. Return `dummy.next`.

```python
def reverseKGroup(head, k):
    dummy = ListNode(0, head)
    group_prev = dummy

    def kth_from(node, k):
        while node and k:
            node = node.next
            k -= 1
        return node

    while True:
        kth = kth_from(group_prev, k)
        if not kth:
            break
        group_next = kth.next
        prev, curr = group_next, group_prev.next
        while curr != group_next:
            nxt = curr.next
            curr.next = prev
            prev = curr
            curr = nxt
        tmp = group_prev.next
        group_prev.next = kth
        group_prev = tmp

    return dummy.next
```

### Complexity

- **Time:** $O(n)$ — each node visited a constant number of times.
- **Space:** $O(1)$.

## Common Pitfalls

- **Reversing a partial final group.** Check that a full `k` nodes exist *before* touching any pointers; the leftover must stay in original order.
- **Losing the seam.** The reversed group's tail must point at the *next* group's head (or the untouched remainder), and the previous group's tail must point at this group's new head. Seeding `prev` with `group_next` handles the first; updating `group_prev.next` handles the second.
- **Advancing `group_prev` to the wrong node.** After reversal, the group's tail is the node that *was* `group_prev.next`; save it before rewiring.

## The keystone

Block-wise list surgery is "reverse a sublist" plus careful seam management. The pattern — a boundary pointer before the segment, reverse the segment with the three-pointer loop, reconnect both ends — generalises to rotating, swapping, or partitioning any run of nodes.
