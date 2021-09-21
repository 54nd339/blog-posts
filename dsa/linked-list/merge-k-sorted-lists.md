---
title: Merge K Sorted Lists - Pair Them Up
description: Merging k sorted linked lists — scanning all k heads each step is O(nk), but a heap of the k current heads or pairwise divide-and-conquer merging gets it to O(n log k).
date: 2021-09-21
draft: false
slug: /dsa/merge-k-sorted-lists
tags:
  - Competitive Programming
  - Linked List
  - Heap
---

Merging two lists is $O(n)$. Merging `k` naively — always picking the smallest of `k` heads — costs $O(nk)$. The `\log k` factor comes from either a heap (find the min in $O(\log k)$) or from merging in a balanced binary-tree pattern so each element is merged only $\log k$ times.

## Description

Given an array of `k` sorted linked lists, merge them into one sorted list and return its head.

**Example**

```
Input:  lists = [[1,4,5],[1,3,4],[2,6]]
Output: [1,1,2,3,4,4,5,6]

Input:  lists = []
Output: []
```

**Constraints**

- $0 \le k \le 10^4$
- Each list is sorted ascending; total nodes up to $10^4$.
- $-10^4 \le \text{Node.val} \le 10^4$

## Prerequisites

- [Merge Two Sorted Lists](/citadel/dsa/merge-two-sorted-lists) — the merge subroutine.
- Binary heaps; divide and conquer.

## Approach 1: Brute Force

### Intuition

Collect every value, sort, rebuild.

### Algorithm

1. Traverse all lists, appending every `val` to an array.
2. Sort the array.
3. Build a new list from the sorted values.

```python
def mergeKLists(lists):
    vals = []
    for node in lists:
        while node:
            vals.append(node.val)
            node = node.next
    vals.sort()
    dummy = ListNode()
    tail = dummy
    for v in vals:
        tail.next = ListNode(v)
        tail = tail.next
    return dummy.next
```

### Complexity

- **Time:** $O(n \log n)$ for `n` total nodes.
- **Space:** $O(n)$ for the value array (plus the new list).

## Approach 2: Iteration (scan all heads)

### Intuition

Repeatedly pick the smallest current head across all `k` lists.

### Algorithm

1. `dummy`, `tail = dummy`.
2. Loop: scan the `k` lists for the one with the smallest non-null head. If none, stop. Attach it, advance that list.
3. Return `dummy.next`.

```python
def mergeKLists(lists):
    dummy = ListNode()
    tail = dummy
    lists = [l for l in lists if l]
    while lists:
        i = min(range(len(lists)), key=lambda j: lists[j].val)
        tail.next = lists[i]
        tail = tail.next
        lists[i] = lists[i].next
        if not lists[i]:
            lists.pop(i)
    return dummy.next
```

### Complexity

- **Time:** $O(n k)$ — an $O(k)$ scan per node.
- **Space:** $O(1)$.

## Approach 3: Merge One By One

### Intuition

Fold the array: merge list 0 with list 1, that result with list 2, and so on.

### Algorithm

1. If `lists` is empty, return `None`.
2. `merged = lists[0]`; for `i` from `1`: `merged = merge_two(merged, lists[i])`.
3. Return `merged`.

```python
def mergeKLists(lists):
    if not lists:
        return None

    def merge_two(a, b):
        dummy = ListNode()
        tail = dummy
        while a and b:
            if a.val <= b.val:
                tail.next, a = a, a.next
            else:
                tail.next, b = b, b.next
            tail = tail.next
        tail.next = a or b
        return dummy.next

    merged = lists[0]
    for i in range(1, len(lists)):
        merged = merge_two(merged, lists[i])
    return merged
```

### Complexity

- **Time:** $O(n k)$ — the accumulating list is re-walked each merge.
- **Space:** $O(1)$.

## Approach 4: Heap

### Intuition

A min-heap of the `k` current heads yields the global minimum in $O(\log k)$; after popping one, push its successor.

### Algorithm

1. Push `(node.val, i, node)` for each non-empty list head (`i` breaks value ties since nodes are not comparable).
2. Pop the smallest, attach to the result, and if it has a `next`, push that.
3. Repeat until the heap is empty.

```python
import heapq

def mergeKLists(lists):
    heap = []
    for i, node in enumerate(lists):
        if node:
            heapq.heappush(heap, (node.val, i, node))
    dummy = ListNode()
    tail = dummy
    while heap:
        val, i, node = heapq.heappop(heap)
        tail.next = node
        tail = tail.next
        if node.next:
            heapq.heappush(heap, (node.next.val, i, node.next))
    return dummy.next
```

### Complexity

- **Time:** $O(n \log k)$.
- **Space:** $O(k)$ for the heap.

## Approach 5: Divide and Conquer (Recursion)

### Intuition

Merge halves recursively: merging `k` lists is merging the first `k/2`, merging the last `k/2`, then merging those two. Each node participates in $\log k$ merges.

### Algorithm

1. `split(l, r)`: if `l > r`, return `None`; if `l == r`, return `lists[l]`; `m = (l + r) // 2`; return `merge_two(split(l, m), split(m + 1, r))`.
2. Call `split(0, len(lists) - 1)`.

```python
def mergeKLists(lists):
    if not lists:
        return None

    def merge_two(a, b):
        dummy = ListNode()
        tail = dummy
        while a and b:
            if a.val <= b.val:
                tail.next, a = a, a.next
            else:
                tail.next, b = b, b.next
            tail = tail.next
        tail.next = a or b
        return dummy.next

    def split(l, r):
        if l > r:
            return None
        if l == r:
            return lists[l]
        m = (l + r) // 2
        return merge_two(split(l, m), split(m + 1, r))

    return split(0, len(lists) - 1)
```

### Complexity

- **Time:** $O(n \log k)$.
- **Space:** $O(\log k)$ recursion.

## Approach 6: Divide and Conquer (Iteration)

### Intuition

The same balanced merging, bottom-up: merge adjacent pairs, halving the list count each round.

### Algorithm

1. While more than one list remains: build a new array by merging `lists[i]` with `lists[i + 1]` for each even `i` (carrying an odd leftover).
2. Return the single remaining list.

```python
def mergeKLists(lists):
    if not lists:
        return None

    def merge_two(a, b):
        dummy = ListNode()
        tail = dummy
        while a and b:
            if a.val <= b.val:
                tail.next, a = a, a.next
            else:
                tail.next, b = b, b.next
            tail = tail.next
        tail.next = a or b
        return dummy.next

    while len(lists) > 1:
        merged = []
        for i in range(0, len(lists), 2):
            a = lists[i]
            b = lists[i + 1] if i + 1 < len(lists) else None
            merged.append(merge_two(a, b))
        lists = merged
    return lists[0]
```

### Complexity

- **Time:** $O(n \log k)$.
- **Space:** $O(k)$ for the working array.

## Common Pitfalls

- **Heap: comparing nodes.** `ListNode` has no ordering, so pushing `(node.val, node)` raises `TypeError` on a value tie. Add a unique tiebreaker like the list index or a running counter.
- **One-by-one merging is not $O(n \log k)$.** Each successive merge re-traverses the growing accumulator; only *balanced* pairing gives the log factor.
- **Iterative D&C with an odd count.** Guard `i + 1 < len(lists)` and merge the leftover with `None`.

## The keystone

`\log k` "merge rounds" is the same balanced-binary-tree structure as merge sort — merge in pairs so each element is touched $\log k$ times, or use a heap to pick the minimum in $\log k$. Both beat the linear scan of all `k` fronts.
