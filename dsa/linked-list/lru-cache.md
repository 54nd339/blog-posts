---
title: LRU Cache - Hash Map Meets Doubly Linked List
description: A fixed-capacity cache that evicts the least recently used key — a list is O(n) per operation, but a hash map to doubly-linked-list nodes gives O(1) get and put.
date: 2021-09-19
draft: false
slug: /dsa/lru-cache
tags:
  - Competitive Programming
  - Linked List
  - Design
---

Two requirements pull in different directions: $O(1)$ lookup by key (a hash map) and $O(1)$ "move this to most-recently-used" and "evict the oldest" (an ordered structure with cheap removal from the middle — a doubly linked list). The design is to run both, with the map's values pointing at list nodes.

## Description

Implement `LRUCache(capacity)` with:

- `get(key)` — return the value if present, else `-1`; a hit counts as a use.
- `put(key, value)` — insert or update; if over capacity, evict the least recently used key.

Both operations must be $O(1)$ average.

**Example**

```
LRUCache(2); put(1,1); put(2,2); get(1) -> 1;
put(3,3)  # evicts key 2
get(2) -> -1; put(4,4)  # evicts key 1
get(1) -> -1; get(3) -> 3; get(4) -> 4
```

**Constraints**

- $1 \le \text{capacity} \le 3000$
- $0 \le \text{key}, \text{value} \le 10^5$
- Up to $2 \times 10^5$ calls.

## Prerequisites

- Hash maps.
- Doubly linked lists with sentinel head/tail nodes for $O(1)$ splice.

## Approach 1: Brute Force

### Intuition

Keep `(key, value)` pairs in a list ordered by recency. Every operation scans.

### Algorithm

1. `get`: linear-search for the key; on hit, remove and re-append the pair (now most recent), return the value.
2. `put`: linear-search; if present, remove it; if at capacity, pop index `0` (least recent); append the new pair.

```python
class LRUCache:
    def __init__(self, capacity: int):
        self.cap = capacity
        self.items: list[list] = []

    def get(self, key: int) -> int:
        for i, (k, v) in enumerate(self.items):
            if k == key:
                self.items.append(self.items.pop(i))
                return v
        return -1

    def put(self, key: int, value: int) -> None:
        for i, (k, _) in enumerate(self.items):
            if k == key:
                self.items.pop(i)
                break
        if len(self.items) >= self.cap:
            self.items.pop(0)
        self.items.append([key, value])
```

### Complexity

- **Time:** $O(n)$ per `get` and `put`.
- **Space:** $O(n)$.

## Approach 2: Doubly Linked List + Hash Map

### Intuition

A doubly linked list orders keys by recency: most recent near the tail, least recent near the head. A hash map from key to its node gives $O(1)$ access; because the list is doubly linked, unlinking any node is $O(1)$.

### Algorithm

1. Sentinels `head` and `tail`; `head.next` is the LRU node, `tail.prev` is the MRU node. Map `key → node`.
2. Helpers: `remove(node)` unlinks it; `insert(node)` places it just before `tail`.
3. `get(key)`: if absent, return `-1`. Else `remove` its node, `insert` it (mark MRU), return its value.
4. `put(key, value)`: if the key exists, `remove` its old node. Create a node, store in the map, `insert` it. If `len(map) > cap`, `remove` `head.next`, delete its key from the map.

```python
class Node:
    def __init__(self, key=0, val=0):
        self.key, self.val = key, val
        self.prev = self.next = None

class LRUCache:
    def __init__(self, capacity: int):
        self.cap = capacity
        self.map: dict[int, Node] = {}
        self.head, self.tail = Node(), Node()
        self.head.next, self.tail.prev = self.tail, self.head

    def _remove(self, node: Node) -> None:
        node.prev.next, node.next.prev = node.next, node.prev

    def _insert(self, node: Node) -> None:
        node.prev, node.next = self.tail.prev, self.tail
        self.tail.prev.next = node
        self.tail.prev = node

    def get(self, key: int) -> int:
        if key not in self.map:
            return -1
        node = self.map[key]
        self._remove(node)
        self._insert(node)
        return node.val

    def put(self, key: int, value: int) -> None:
        if key in self.map:
            self._remove(self.map[key])
        node = Node(key, value)
        self.map[key] = node
        self._insert(node)
        if len(self.map) > self.cap:
            lru = self.head.next
            self._remove(lru)
            del self.map[lru.key]
```

### Complexity

- **Time:** $O(1)$ for `get` and `put`.
- **Space:** $O(\text{capacity})$.

## Approach 3: Built-In Ordered Map

### Intuition

Python's `dict` preserves insertion order and offers $O(1)$ `move_to_end` and `popitem`, so it *is* an ordered hash map. (`collections.OrderedDict` makes the intent explicit.)

### Algorithm

1. Store `key → value` in an `OrderedDict`.
2. `get`: if present, `move_to_end(key)` and return the value, else `-1`.
3. `put`: if present, delete first; set `key`; if over capacity, `popitem(last=False)` to drop the oldest.

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.cap = capacity
        self.cache: OrderedDict[int, int] = OrderedDict()

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.cap:
            self.cache.popitem(last=False)
```

### Complexity

- **Time:** $O(1)$ for `get` and `put`.
- **Space:** $O(\text{capacity})$.

## Common Pitfalls

- **Sentinel nodes.** With dummy `head` and `tail`, `_remove` and `_insert` never touch `None`; without them, every operation needs empty-list and single-element branches.
- **Not deleting the evicted key from the map.** Unlinking the node is not enough — its map entry must go too, or `len(map)` stays wrong and stale keys "exist".
- **`put` on an existing key over capacity.** Update must not trigger an eviction of the key you just refreshed; remove the old node *before* inserting the new one so the count is right.
- **Approach 3: forgetting `move_to_end` on an update.** A `put` to an existing key is a use and must refresh recency.

## The keystone

When you need two different $O(1)$ access patterns on the same data — by key and by order — run two structures that share pointers: a hash map for lookup, a linked list for ordering. This exact hash-map-plus-linked-list pairing also builds LFU caches and [Design Twitter](/citadel/dsa/design-twitter)-style feeds.
