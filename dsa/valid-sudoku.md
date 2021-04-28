---
title: Valid Sudoku - One Pass, Three Kinds of Seen-Set
description: Checking a 9x9 board for rule violations without solving it — the only real question is indexing the three constraints, and the 3x3 box index is just integer division.
date: 2021-04-28
draft: false
slug: /dsa/valid-sudoku
tags:
  - Competitive Programming
  - Arrays & Hashing
  - Matrix
---

The board is a fixed 81 cells, so every correct approach is $O(1)$ in the input-size sense. What the approaches trade is clarity and constant factors, and the sub-problem worth learning is turning a `(row, col)` cell into the index of its `3 x 3` box.

## Description

Given a partially filled `9 x 9` Sudoku board, decide whether it is valid: no digit `1`–`9` repeats within any row, any column, or any `3 x 3` box. Empty cells are `'.'` and are ignored. The board does not need to be solvable.

**Example**

```
Input:  board =
[["5","3",".",".","7",".",".",".","."]
,["6",".",".","1","9","5",".",".","."]
,[".","9","8",".",".",".",".","6","."]
,["8",".",".",".","6",".",".",".","3"]
,["4",".",".","8",".","3",".",".","1"]
,["7",".",".",".","2",".",".",".","6"]
,[".","6",".",".",".",".","2","8","."]
,[".",".",".","4","1","9",".",".","5"]
,[".",".",".",".","8",".",".","7","9"]]
Output: true
```

**Constraints**

- `board.length == board[i].length == 9`
- `board[i][j]` is a digit `1`–`9` or `'.'`.

## Prerequisites

- Hash sets for duplicate detection.
- The block-index formula `(r // 3) * 3 + c // 3`, which flattens a 2-D block coordinate into a bucket `0`–`8`.
- Bitmasks (Approach 3): one integer as a set of up to 9 flags.

## Approach 1: Brute Force

### Intuition

Validate each of the 27 units — 9 rows, 9 columns, 9 boxes — independently with a set.

### Algorithm

1. For each row, collect its non-`.` digits into a set; if a duplicate appears, return `false`.
2. Do the same for each column.
3. For each box, iterate its 9 cells using `row = (box // 3) * 3 + i`, `col = (box % 3) * 3 + j`; check for duplicates.
4. Return `true` if all 27 checks pass.

```python
def isValidSudoku(board: list[list[str]]) -> bool:
    def ok(cells: list[str]) -> bool:
        vals = [c for c in cells if c != "."]
        return len(vals) == len(set(vals))

    for r in range(9):
        if not ok(board[r]):
            return False
    for c in range(9):
        if not ok([board[r][c] for r in range(9)]):
            return False
    for box in range(9):
        cells = [
            board[(box // 3) * 3 + i][(box % 3) * 3 + j]
            for i in range(3) for j in range(3)
        ]
        if not ok(cells):
            return False
    return True
```

### Complexity

- **Time:** $O(9^2)$ — a constant, three passes over the board.
- **Space:** $O(9)$ for a unit's set.

## Approach 2: Hash Set (One Pass)

### Intuition

Keep 9 row sets, 9 column sets, and 9 box sets. Walk every cell once and check all three constraints for it at the same time.

### Algorithm

1. Create `rows`, `cols`, `boxes` as maps of index to set.
2. For each filled cell `(r, c)` with digit `d`, compute `b = (r // 3) * 3 + c // 3`.
3. If `d` is in `rows[r]`, `cols[c]`, or `boxes[b]`, return `false`.
4. Otherwise add `d` to all three.
5. Return `true` after the full board.

```python
from collections import defaultdict

def isValidSudoku(board: list[list[str]]) -> bool:
    rows = defaultdict(set)
    cols = defaultdict(set)
    boxes = defaultdict(set)
    for r in range(9):
        for c in range(9):
            d = board[r][c]
            if d == ".":
                continue
            b = (r // 3) * 3 + c // 3
            if d in rows[r] or d in cols[c] or d in boxes[b]:
                return False
            rows[r].add(d)
            cols[c].add(d)
            boxes[b].add(d)
    return True
```

### Complexity

- **Time:** $O(9^2)$ — 81 cells, three $O(1)$ set checks each.
- **Space:** $O(9^2)$ — the sets hold at most 81 entries, three times over.

## Approach 3: Bitmask

### Intuition

Each "seen digits" set is at most 9 bits, so store it as one integer. Bit `d - 1` set means digit `d` has appeared.

### Algorithm

1. Create three lists of 9 integers, all `0` — one per row, column, box.
2. For each filled cell, let `bit = 1 << (int(d) - 1)`.
3. If `bit` is already set in the row, column, or box integer (`x & bit`), return `false`.
4. Otherwise OR `bit` into all three.
5. Return `true`.

```python
def isValidSudoku(board: list[list[str]]) -> bool:
    rows = [0] * 9
    cols = [0] * 9
    boxes = [0] * 9
    for r in range(9):
        for c in range(9):
            d = board[r][c]
            if d == ".":
                continue
            bit = 1 << (int(d) - 1)
            b = (r // 3) * 3 + c // 3
            if rows[r] & bit or cols[c] & bit or boxes[b] & bit:
                return False
            rows[r] |= bit
            cols[c] |= bit
            boxes[b] |= bit
    return True
```

### Complexity

- **Time:** $O(9^2)$.
- **Space:** $O(9)$ — 27 integers.

## Common Pitfalls

- **The box index.** It must be `(r // 3) * 3 + c // 3`. Dropping the `* 3` (`r // 3 + c // 3`) collapses distinct boxes onto the same key — `(0, 5)` and `(5, 0)` both map to `1` — so real duplicates go unreported.
- **Forgetting to skip `'.'`.** Empty cells are not digits; adding them to a set makes every column with two blanks "invalid".
- **`int(d) - 1` for the bit index** — a digit character must be converted; `ord(d)` without the `- ord('1')` offset shifts every bit.

## The keystone

When several constraints apply to the same element, give each its own hash set (or bitmask) and check all of them in a single pass. The box-index arithmetic `(r // 3) * 3 + c // 3` is the same flatten-a-2D-coordinate move used in [Search a 2D Matrix](/citadel/dsa/search-a-2d-matrix) and grid-as-array traversals.
