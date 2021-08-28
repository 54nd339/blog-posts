---
title: Reverse a Linked List - The Three-Pointer Shuffle
description: Reversing a singly linked list — a recursion that rewires on the way back up, and the iterative version that walks once with a prev, curr, and saved-next pointer in O(1) space.
date: 2021-08-28
draft: false
slug: /dsa/reverse-linked-list
tags:
  - Competitive Programming
  - Linked List
  - Recursion
---

Reversing a linked list is flipping every `next` pointer to point at the previous node. The only trap is that once you overwrite `curr.next`, you have lost your way forward — so you save it first.

## Description

Given the `head` of a singly linked list, reverse the list and return the new head.

**Example**

```
Input:  head = [1, 2, 3, 4, 5]
Output: [5, 4, 3, 2, 1]

Input:  head = []
Output: []
```

**Constraints**

- The list has `0` to `5000` nodes.
- $-5000 \le \text{Node.val} \le 5000$

## Prerequisites

- Singly linked list node: `val` and a `next` reference.
- Comfort with reassigning pointers without dropping the rest of the list.

## Approach 1: Recursion

### Intuition

Recurse to the end. As each call returns, the node *after* the current one should now point back at the current one.

### Algorithm

1. If `head` is `None` or `head.next` is `None`, return `head` (it is the new head).
2. Recursively reverse `head.next`; call the result `new_head`.
3. Set `head.next.next = head` (the next node now points back), then `head.next = None`.
4. Return `new_head`.

```python
def reverseList(head):
    if head is None or head.next is None:
        return head
    new_head = reverseList(head.next)
    head.next.next = head
    head.next = None
    return new_head
```

### Complexity

- **Time:** $O(n)$ — one call per node.
- **Space:** $O(n)$ — recursion depth.

## Approach 2: Iteration

### Intuition

Carry a `prev` pointer. For each node, remember its `next`, redirect it to `prev`, then slide both pointers forward.

### Algorithm

1. `prev = None`, `curr = head`.
2. While `curr`:
   - `nxt = curr.next`.
   - `curr.next = prev`.
   - `prev = curr`; `curr = nxt`.
3. Return `prev`.

```python
def reverseList(head):
    prev = None
    curr = head
    while curr:
        nxt = curr.next
        curr.next = prev
        prev = curr
        curr = nxt
    return prev
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Overwriting `curr.next` before saving it.** Without `nxt = curr.next` first, `curr.next = prev` destroys the only link to the rest of the list.
- **Returning `head` instead of `prev`.** After the loop, `curr` is `None` and `prev` is the last node processed — the new head. The old `head` is now the tail.
- **Recursion: forgetting `head.next = None`.** The original head must become the tail; leaving its `next` intact creates a two-node cycle at the end.

## The keystone

The `prev` / `curr` / `nxt` shuffle is the fundamental linked-list operation — it appears as a subroutine inside [Reorder List](/citadel/dsa/reorder-list), [Reverse Nodes in K-Group](/citadel/dsa/reverse-nodes-in-k-group), and palindrome checks. Learn it cold so it is a reflex, not a puzzle.
