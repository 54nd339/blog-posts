---
title: Detect Squares - Fix a Diagonal, Multiply the Corner Counts
description: A data structure counting axis-aligned squares through a query point — store point multiplicities in a map, then for each point sharing no coordinate, treat it as the opposite corner and multiply the two remaining corner counts.
date: 2022-07-06
draft: false
slug: /dsa/detect-squares
tags:
  - Competitive Programming
  - Math & Geometry
  - Hash Table
---

For an axis-aligned square, once you fix the query point and a diagonally opposite point with `abs(dx) == abs(dy) != 0`, the other two corners are determined. Count how many times each of those exists and multiply; sum over all candidate diagonal points.

## Description

Implement `DetectSquares`:

- `add(point)` — add a point `[x, y]` to the structure (duplicates allowed).
- `count(point)` — given a query point, return the number of ways to pick three points already added that, with the query point, form an axis-aligned square with positive area.

**Example**

```
add([3,10]); add([11,2]); add([3,2])
count([11,10]) -> 1
count([14,8])  -> 0
```

**Constraints**

- $0 \le x, y \le 1000$
- At most 3000 calls total to `add` and `count`.

## Prerequisites

- A `Counter` keyed by `(x, y)` for point multiplicities.
- The observation that a diagonal corner with equal `|dx|` and `|dy|` fixes the square.

## Approach 1: Point-count map, iterate diagonal candidates

### Intuition

Keep `counts[(x, y)]`. For `count(qx, qy)`, iterate every stored point `(px, py)` with `abs(px - qx) == abs(py - qy)` and `px != qx` (non-degenerate diagonal). The other corners are `(qx, py)` and `(px, qy)`. Add `counts[(px, py)] * counts[(qx, py)] * counts[(px, qy)]`.

### Algorithm

1. `add`: `counts[tuple(point)] += 1`; also keep a list `pts` of added points (or iterate `counts` keys).
2. `count(qx, qy)`: `total = 0`. For each key `(px, py)` in `counts`: if `abs(px - qx) != abs(py - qy)` or `px == qx`, skip. Else `total += counts[(px, py)] * counts.get((qx, py), 0) * counts.get((px, qy), 0)`.
3. Return `total`.

```python
from collections import Counter

class DetectSquares:
    def __init__(self):
        self.counts = Counter()

    def add(self, point: list[int]) -> None:
        self.counts[(point[0], point[1])] += 1

    def count(self, point: list[int]) -> int:
        qx, qy = point
        total = 0
        for (px, py), c in self.counts.items():
            if px == qx or abs(px - qx) != abs(py - qy):
                continue
            total += c * self.counts.get((qx, py), 0) * self.counts.get((px, qy), 0)
        return total
```

### Complexity

- **Time:** `add` $O(1)$; `count` $O(k)$ where `k` is the number of distinct points.
- **Space:** $O(k)$.

## Approach 2: Column index for fewer candidates

### Intuition

Group points by `x`. For `count(qx, qy)`, only points in column `qx` can be a vertical edge partner; for each such `(qx, py)` with `py != qy`, the side length is `abs(py - qy)`, giving two possible squares (left and right). Multiply the three needed corner counts for each side.

### Algorithm

1. `by_col[x]` is a `Counter` of `y` values; also keep the flat `counts`.
2. `count(qx, qy)`: for each `py` in `by_col[qx]` with `py != qy`: `side = abs(py - qy)`. For `nx` in `(qx - side, qx + side)`: `total += by_col[qx][py] * counts[(nx, qy)] * counts[(nx, py)]`.
3. Return `total`.

```python
from collections import Counter, defaultdict

class DetectSquares:
    def __init__(self):
        self.counts = Counter()
        self.by_col = defaultdict(Counter)

    def add(self, point: list[int]) -> None:
        x, y = point
        self.counts[(x, y)] += 1
        self.by_col[x][y] += 1

    def count(self, point: list[int]) -> int:
        qx, qy = point
        total = 0
        for py, c in self.by_col[qx].items():
            if py == qy:
                continue
            side = abs(py - qy)
            for nx in (qx - side, qx + side):
                total += c * self.counts[(nx, qy)] * self.counts[(nx, py)]
        return total
```

### Complexity

- **Time:** `add` $O(1)$; `count` $O(h)$ where `h` is the number of points in column `qx`.
- **Space:** $O(k)$.

## Common Pitfalls

- **Counting degenerate squares.** Require `px != qx` (equivalently `side != 0`); a zero-area "square" would otherwise be counted.
- **Using `>` or `<` instead of `abs(dx) == abs(dy)`.** The diagonal must have equal horizontal and vertical spans for an axis-aligned square.
- **Forgetting multiplicity.** `add` allows duplicates; multiply by every corner's stored count, not by 1.
- **Missing one of the two squares.** For a fixed vertical partner in column `qx`, there is a square on each side (`qx - side` and `qx + side`).

## The keystone

Geometric counting problems often reduce to "fix one or two defining points, then the rest are forced — look them up and multiply". A coordinate-keyed hash map makes each lookup $O(1)$, turning an $O(k^3)$ search over quadruples into $O(k)$ per query.
