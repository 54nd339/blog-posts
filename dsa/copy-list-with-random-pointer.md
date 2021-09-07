---
title: Copy List with Random Pointer - Cloning a Tangled Graph
description: Deep-copying a list where each node also points at an arbitrary node — a hash map from original to copy handles the forward references, and an interleaving trick removes the map entirely.
date: 2021-09-07
draft: false
slug: /dsa/copy-list-with-random-pointer
tags:
  - Competitive Programming
  - Linked List
  - Hash Table
---

The `random` pointer can point anywhere, including to a node you have not created yet. The fix is to decouple "make all the copies" from "wire up the pointers", with a map from each original node to its clone bridging the two phases — or, cleverly, by threading the clones through the original list.

## Description

Each node has `val`, `next`, and `random` (pointing to any node in the list or `None`). Return a deep copy: a new list of new nodes whose `next` and `random` mirror the original's structure, with no new node pointing at an original.

**Example**

```
Input:  head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
Output: [[7,null],[13,0],[11,4],[10,2],[1,0]]
        (each pair is [val, index that random points to])
```

**Constraints**

- `0 <= n <= 1000`
- $-10^4 \le \text{Node.val} \le 10^4$
- `random` is `None` or points to some node in the list.

## Prerequisites

- Hash maps keyed by object identity.
- The idea that cloning a pointer structure is a graph copy: create nodes first, connect edges second.

## Approach 1: Recursion + Hash Map

### Intuition

Copy a node by recursively copying whatever it points to, memoising so each original maps to exactly one clone.

### Algorithm

1. Keep a map `old → new`.
2. `clone(node)`: if `node` is `None`, return `None`. If `node` is in the map, return its clone. Otherwise create the clone, store it, set `clone.next = clone(node.next)` and `clone.random = clone(node.random)`, and return it.

```python
def copyRandomList(head):
    seen = {None: None}

    def clone(node):
        if node in seen:
            return seen[node]
        copy = Node(node.val)
        seen[node] = copy
        copy.next = clone(node.next)
        copy.random = clone(node.random)
        return copy

    return clone(head)
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$ — map plus recursion.

## Approach 2: Hash Map (Two Pass)

### Intuition

First pass: create a bare clone for every node and record `old → new`. Second pass: with all clones existing, set each clone's `next` and `random` by looking up the originals' targets.

### Algorithm

1. Map `{None: None}`. Pass 1: for each `node`, `mp[node] = Node(node.val)`.
2. Pass 2: for each `node`, `mp[node].next = mp[node.next]`, `mp[node].random = mp[node.random]`.
3. Return `mp[head]`.

```python
def copyRandomList(head):
    mp = {None: None}
    node = head
    while node:
        mp[node] = Node(node.val)
        node = node.next
    node = head
    while node:
        mp[node].next = mp[node.next]
        mp[node].random = mp[node.random]
        node = node.next
    return mp[head]
```

### Complexity

- **Time:** $O(n)$ — two passes.
- **Space:** $O(n)$ — the map.

## Approach 3: Hash Map (One Pass)

### Intuition

Use a default map that lazily creates an empty clone on first access, so `next` and `random` can be wired in the same pass that creates nodes.

### Algorithm

1. `mp` returns a fresh `Node(0)` on unknown keys; `mp[None] = None`.
2. For each `node`: `mp[node].val = node.val`; `mp[node].next = mp[node.next]`; `mp[node].random = mp[node.random]`.
3. Return `mp[head]`.

```python
from collections import defaultdict

def copyRandomList(head):
    mp = defaultdict(lambda: Node(0))
    mp[None] = None
    node = head
    while node:
        mp[node].val = node.val
        mp[node].next = mp[node.next]
        mp[node].random = mp[node.random]
        node = node.next
    return mp[head]
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(n)$.

## Approach 4: Space Optimized (interleave)

### Intuition

Thread each clone right after its original: `A → A' → B → B' → ...`. Now `A'.random` is simply `A.random.next`, because `A.random`'s clone sits directly after it. Then unweave the two lists.

### Algorithm

1. Pass 1: for each original `A`, insert `A' = Node(A.val)` between `A` and `A.next`.
2. Pass 2: for each original `A`, if `A.random`, set `A.next.random = A.random.next`.
3. Pass 3: detach — for each original `A`, `A.next = A.next.next` and `A'.next = A'.next.next` (guarding `None`).
4. Return the first clone.

```python
def copyRandomList(head):
    if not head:
        return None
    node = head
    while node:
        copy = Node(node.val, node.next)
        node.next = copy
        node = copy.next
    node = head
    while node:
        if node.random:
            node.next.random = node.random.next
        node = node.next.next
    dummy = Node(0)
    copy_tail = dummy
    node = head
    while node:
        copy_tail.next = node.next
        copy_tail = copy_tail.next
        node.next = node.next.next
        node = node.next
    return dummy.next
```

### Complexity

- **Time:** $O(n)$ — three passes.
- **Space:** $O(1)$ extra.

## Approach 5: Space Optimized (stash in random)

### Intuition

A variant: temporarily store each clone in the *original's* `random` field, use those stashed references to set the clones' `random` pointers, then restore.

### Algorithm

1. Pass 1: for each `A`, create `A'`; stash the original target in `A'.next` (`A'.next = A.random`), then set `A.random = A'` so each original points at its clone.
2. Pass 2: for each `A`, let `A' = A.random` and `target = A'.next` (an *original* node); set `A'.random = target.random` (which is `target`'s clone).
3. Pass 3: for each `A`, restore `A.random = A'.next` (the stashed original target), then relink `A'.next` to the next clone (`A.next.random`) and chain the clones onto a dummy.

```python
def copyRandomList(head):
    if not head:
        return None
    cur = head
    while cur:
        copy = Node(cur.val)
        copy.next = cur.random      # stash original's random target
        cur.random = copy           # original points at its clone
        cur = cur.next
    cur = head
    while cur:
        copy = cur.random
        target = copy.next          # an original node (or None)
        copy.random = target.random if target else None
        cur = cur.next
    dummy = Node(0)
    tail = dummy
    cur = head
    while cur:
        copy = cur.random
        cur.random = copy.next      # restore original's random
        copy.next = cur.next.random if cur.next else None
        tail.next = copy
        tail = copy
        cur = cur.next
    return dummy.next
```

### Complexity

- **Time:** $O(n)$.
- **Space:** $O(1)$ extra.

## Common Pitfalls

- **Mapping `None` to something other than `None`.** Seeding the map with `{None: None}` lets `clone.next`/`clone.random` handle the null target with no branch.
- **Wiring pointers before all clones exist (single-pass without a lazy map).** `random` can point forward; you need every clone to exist, or a default-dict that fabricates it on demand.
- **Interleave: not restoring the original list.** The problem expects the input list unmodified; passes 2 and 3 must fully detach and repair.

## The keystone

Copying any pointer structure with forward or cyclic references is a two-phase graph clone: allocate all nodes, then connect all edges, with an identity map linking old to new. The interleaving trick is a neat way to *be* that map in $O(1)$ space when nodes have a spare `next` slot to borrow.
