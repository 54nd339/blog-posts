---
title: Time Based Key-Value Store - Binary Search on Sorted Timestamps
description: A store where get(key, t) returns the value from the largest timestamp not exceeding t — because timestamps are inserted in increasing order, each key's history is already sorted for binary search.
date: 2021-08-04
draft: false
slug: /dsa/time-based-key-value-store
tags:
  - Competitive Programming
  - Binary Search
  - Design
---

`set` always arrives with a strictly increasing timestamp for a given key, so each key's list of `(timestamp, value)` pairs is sorted by construction. `get` is then a "largest timestamp `<= t`" query — a binary search.

## Description

Design `TimeMap` with:

- `set(key, value, timestamp)` — store `value` under `key` at time `timestamp`.
- `get(key, timestamp)` — return the value set for `key` at the largest stored time `<= timestamp`, or `""` if none.

**Example**

```
set("foo", "bar", 1)
get("foo", 1)  -> "bar"
get("foo", 3)  -> "bar"      # latest at time <= 3 is time 1
set("foo", "bar2", 4)
get("foo", 4)  -> "bar2"
get("foo", 5)  -> "bar2"
```

**Constraints**

- $1 \le \text{key.length}, \text{value.length} \le 100$
- $1 \le \text{timestamp} \le 10^7$
- `set` is called with strictly increasing `timestamp` values per key.
- At most $2 \times 10^5$ calls.

## Prerequisites

- Hash map of key → list.
- [Binary search](/citadel/dsa/binary-search) for "rightmost element `<= x`" (an upper bound minus one).

## Approach 1: Brute Force

### Intuition

Store a list per key; on `get`, scan it for the largest timestamp `<= t`.

### Algorithm

1. `store`: `key -> list of (timestamp, value)`.
2. `set`: append `(timestamp, value)`.
3. `get`: iterate the list, tracking the value at the largest timestamp `<= t`.

```python
class TimeMap:
    def __init__(self):
        self.store: dict[str, list[tuple[int, str]]] = {}

    def set(self, key: str, value: str, timestamp: int) -> None:
        self.store.setdefault(key, []).append((timestamp, value))

    def get(self, key: str, timestamp: int) -> str:
        best = ""
        for ts, val in self.store.get(key, []):
            if ts <= timestamp:
                best = val
            else:
                break
        return best
```

### Complexity

- **Time:** $O(1)$ for `set`; $O(n)$ for `get`.
- **Space:** $O(m \cdot n)$ over `m` keys.

## Approach 2: Binary Search (Sorted Map)

### Intuition

Keep each key's history in an ordered structure with a "floor" query. Python has no built-in tree map, but `SortedDict` from `sortedcontainers` provides it; here it is emulated by keeping a sorted list of timestamps alongside.

### Algorithm

1. Per key, maintain a list of timestamps and a parallel list of values, both kept sorted by timestamp (they already arrive sorted).
2. `get`: `bisect_right` the timestamps for `t`, take the value one index back.

```python
from bisect import bisect_right

class TimeMap:
    def __init__(self):
        self.times: dict[str, list[int]] = {}
        self.values: dict[str, list[str]] = {}

    def set(self, key: str, value: str, timestamp: int) -> None:
        self.times.setdefault(key, []).append(timestamp)
        self.values.setdefault(key, []).append(value)

    def get(self, key: str, timestamp: int) -> str:
        if key not in self.times:
            return ""
        i = bisect_right(self.times[key], timestamp)
        return self.values[key][i - 1] if i > 0 else ""
```

### Complexity

- **Time:** $O(1)$ amortised for `set`; $O(\log n)$ for `get`.
- **Space:** $O(m \cdot n)$.

## Approach 3: Binary Search (Array)

### Intuition

Same as Approach 2 but with a single list of `[timestamp, value]` pairs per key and a hand-written binary search for the rightmost timestamp `<= t`.

### Algorithm

1. Per key, a list of `[timestamp, value]`; `set` appends.
2. `get`: binary-search for the largest index with `timestamp <= t`; return its value or `""`.

```python
class TimeMap:
    def __init__(self):
        self.store: dict[str, list[list]] = {}

    def set(self, key: str, value: str, timestamp: int) -> None:
        self.store.setdefault(key, []).append([timestamp, value])

    def get(self, key: str, timestamp: int) -> str:
        arr = self.store.get(key, [])
        lo, hi = 0, len(arr) - 1
        res = ""
        while lo <= hi:
            mid = (lo + hi) // 2
            if arr[mid][0] <= timestamp:
                res = arr[mid][1]
                lo = mid + 1
            else:
                hi = mid - 1
        return res
```

### Complexity

- **Time:** $O(1)$ for `set`; $O(\log n)$ for `get`.
- **Space:** $O(m \cdot n)$.

## Common Pitfalls

- **Using `bisect_left` for `get`.** You want the rightmost timestamp `<= t`, which is `bisect_right(times, t) - 1`. `bisect_left` returns the wrong index when `t` exactly matches a stored timestamp.
- **Returning the pair instead of the value.** `get` returns the value string, not `(timestamp, value)`.
- **Assuming timestamps need sorting.** They arrive strictly increasing per key, so `append` keeps the list sorted; an explicit sort per `set` would make `set` $O(n \log n)$ for nothing.

## The keystone

When one axis of your data is monotonically ordered by construction (insertion time, sequence number, version), a "latest at or before `t`" lookup is a binary search — no tree needed. This "sorted-by-arrival, bisect on query" pattern is common in versioned stores and event logs.
