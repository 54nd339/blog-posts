---
title: Merge Two Sorted Lists - The Dummy Head Trick
description: Splicing two sorted linked lists into one — a recursion that picks the smaller head each call, and an iterative merge whose dummy node removes every "is this the first node?" special case.
date: 2021-08-28
draft: false
slug: /dsa/merge-two-sorted-lists
tags:
  - Competitive Programming
  - Linked List
  - Recursion
---

Merging is the classic two-pointer sweep, but on linked nodes you splice pointers instead of copying values. A *dummy head* node means you never special-case attaching the very first result node.

## Description

Given the heads of two sorted linked lists `list1` and `list2`, splice them into one sorted list by reusing the existing nodes, and return its head.

**Example**

```
Input:  list1 = [1, 2, 4], list2 = [1, 3, 4]
Output: [1, 1, 2, 3, 4, 4]

Input:  list1 = [], list2 = [0]
Output: [0]
```

**Constraints**

- Each list has `0` to `50` nodes.
- $-100 \le \text{Node.val} \le 100$
- Both lists are sorted non-decreasing.

## Prerequisites

- The two-pointer merge step from merge sort.
- The dummy/sentinel node idiom for building a list.

## Approach 1: Recursion

### Intuition

The merged list's head is the smaller of the two current heads; the rest is the merge of what remains.

### Algorithm

1. If `list1` is `None`, return `list2`; if `list2` is `None`, return `list1`.
2. If `list1.val <= list2.val`, set `list1.next = mergeTwoLists(list1.next, list2)` and return `list1`.
3. Otherwise set `list2.next = mergeTwoLists(list1, list2.next)` and return `list2`.

```python
def mergeTwoLists(list1, list2):
    if not list1:
        return list2
    if not list2:
        return list1
    if list1.val <= list2.val:
        list1.next = mergeTwoLists(list1.next, list2)
        return list1
    list2.next = mergeTwoLists(list1, list2.next)
    return list2
```

### Complexity

- **Time:** $O(n + m)$.
- **Space:** $O(n + m)$ — recursion depth.

## Approach 2: Iteration

### Intuition

Walk both lists with a `tail` pointer that starts on a dummy node. Repeatedly attach the smaller head and advance. When one list runs out, attach the other whole.

### Algorithm

1. `dummy = ListNode()`, `tail = dummy`.
2. While both `list1` and `list2` are non-empty: attach the smaller-valued node to `tail.next`, advance that list, advance `tail`.
3. Attach whichever list still has nodes.
4. Return `dummy.next`.

```python
def mergeTwoLists(list1, list2):
    dummy = ListNode()
    tail = dummy
    while list1 and list2:
        if list1.val <= list2.val:
            tail.next = list1
            list1 = list1.next
        else:
            tail.next = list2
            list2 = list2.next
        tail = tail.next
    tail.next = list1 or list2
    return dummy.next
```

### Complexity

- **Time:** $O(n + m)$.
- **Space:** $O(1)$.

## Common Pitfalls

- **Building without a dummy node.** You then need a branch to set the head on the first attachment; the dummy absorbs that case.
- **Forgetting `tail.next = list1 or list2` at the end.** When one list empties, the other's remaining (already sorted) nodes must be linked in one step.
- **Using `<` instead of `<=`.** Both give a valid merge, but `<=` keeps the merge *stable* (equal values keep `list1` before `list2`).

## The keystone

The dummy head is a small idiom with a big payoff: any time you build a linked list, start from a sentinel and return `dummy.next`, and the "first element" edge case disappears. This merge is also the `conquer` step of [Merge K Sorted Lists](/citadel/dsa/merge-k-sorted-lists).
