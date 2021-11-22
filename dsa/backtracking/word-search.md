---
title: Word Search - DFS With a Trail You Erase
description: Whether a word can be traced through adjacent grid cells without reuse — DFS from each matching start, marking cells on the current path and unmarking on the way back.
date: 2021-11-22
draft: false
slug: /dsa/word-search
tags:
  - Competitive Programming
  - Backtracking
  - Matrix
---

From each cell that matches the first letter, walk the grid depth-first spelling the word. Mark a cell while it is on the current path so you cannot step on it twice; unmark it when you back out so other paths can use it.

## Description

Given an `m x n` grid of characters `board` and a string `word`, return `true` if `word` can be formed from letters of sequentially adjacent (horizontally or vertically) cells, using each cell at most once.

**Example**

```
Input:  board = [["A","B","C","E"],
                 ["S","F","C","S"],
                 ["A","D","E","E"]], word = "ABCCED"
Output: true

Input:  board = same, word = "ABCB"
Output: false
```

**Constraints**

- $1 \le m, n \le 6$; $1 \le \text{word.length} \le 15$
- Letters are uppercase and lowercase English.

## Prerequisites

- Grid DFS with four-directional moves.
- Backtracking: mark state before recursing, restore after.

## Approach 1: Backtracking (hash set of path cells)

### Intuition

Track the current path in a set of `(r, c)`. A cell is usable if it is in bounds, matches the next letter, and is not already on the path.

### Algorithm

1. For each cell, call `dfs(r, c, 0)`.
2. `dfs(r, c, k)`: if `k == len(word)`, return `True`. Reject out-of-bounds, `board[r][c] != word[k]`, or `(r, c)` in `path`.
3. Add `(r, c)` to `path`; recurse into the 4 neighbours with `k + 1`; remove `(r, c)`.
4. Return whether any start succeeded.

```python
def exist(board: list[list[str]], word: str) -> bool:
    rows, cols = len(board), len(board[0])
    path = set()

    def dfs(r, c, k):
        if k == len(word):
            return True
        if (r < 0 or c < 0 or r >= rows or c >= cols
                or board[r][c] != word[k] or (r, c) in path):
            return False
        path.add((r, c))
        found = (dfs(r + 1, c, k + 1) or dfs(r - 1, c, k + 1) or
                 dfs(r, c + 1, k + 1) or dfs(r, c - 1, k + 1))
        path.remove((r, c))
        return found

    return any(dfs(r, c, 0) for r in range(rows) for c in range(cols))
```

### Complexity

- **Time:** $O(m \cdot n \cdot 4^L)$ where `L` is the word length.
- **Space:** $O(L)$ for the path set and recursion.

## Approach 2: Backtracking (visited matrix)

### Intuition

Replace the set with a boolean grid of the same shape — a marginally faster membership test.

### Algorithm

Identical to Approach 1, using `visited[r][c]` instead of `(r, c) in path`, set to `True` before recursing and back to `False` after.

```python
def exist(board: list[list[str]], word: str) -> bool:
    rows, cols = len(board), len(board[0])
    visited = [[False] * cols for _ in range(rows)]

    def dfs(r, c, k):
        if k == len(word):
            return True
        if (r < 0 or c < 0 or r >= rows or c >= cols
                or visited[r][c] or board[r][c] != word[k]):
            return False
        visited[r][c] = True
        found = (dfs(r + 1, c, k + 1) or dfs(r - 1, c, k + 1) or
                 dfs(r, c + 1, k + 1) or dfs(r, c - 1, k + 1))
        visited[r][c] = False
        return found

    return any(dfs(r, c, 0) for r in range(rows) for c in range(cols))
```

### Complexity

- **Time:** $O(m \cdot n \cdot 4^L)$.
- **Space:** $O(m \cdot n)$ for the visited grid.

## Approach 3: Backtracking (in-place marking)

### Intuition

Avoid any extra structure: overwrite the current cell with a sentinel like `'#'` during recursion, restore it after.

### Algorithm

Same DFS; on entry set `tmp = board[r][c]`, `board[r][c] = '#'`; on exit `board[r][c] = tmp`. The mismatch check `board[r][c] != word[k]` also rejects `'#'` automatically.

```python
def exist(board: list[list[str]], word: str) -> bool:
    rows, cols = len(board), len(board[0])

    def dfs(r, c, k):
        if k == len(word):
            return True
        if (r < 0 or c < 0 or r >= rows or c >= cols
                or board[r][c] != word[k]):
            return False
        tmp = board[r][c]
        board[r][c] = "#"
        found = (dfs(r + 1, c, k + 1) or dfs(r - 1, c, k + 1) or
                 dfs(r, c + 1, k + 1) or dfs(r, c - 1, k + 1))
        board[r][c] = tmp
        return found

    return any(dfs(r, c, 0) for r in range(rows) for c in range(cols))
```

### Complexity

- **Time:** $O(m \cdot n \cdot 4^L)$.
- **Space:** $O(L)$ recursion, no auxiliary grid.

## Common Pitfalls

- **Not restoring the cell.** Whether you use a set, a matrix, or in-place `'#'`, the mark must be undone after the four recursions, or later start positions see a corrupted board / stale path.
- **Checking `k == len(word)` after the bounds/match check.** Check "word complete" *first*; otherwise a valid final step can be rejected for being at an edge.
- **Revisiting via a different direction.** All four moves must consult the same mark; a common bug is marking only before some of the recursive calls.
- **Trying every cell as a start.** Necessary — the word can begin anywhere.

## The keystone

Path search on a grid without cell reuse is DFS plus a *reversible* mark. The "set state, recurse, unset state" rhythm is the definition of backtracking; the three variants here only differ in where the mark lives.
