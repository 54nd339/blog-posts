---
title: N-Queens - One Queen Per Row, Three Sets Say Where
description: Placing n non-attacking queens on an n by n board — recurse row by row and track occupied columns and both diagonal families so each placement check is constant time.
date: 2021-11-30
draft: false
slug: /dsa/n-queens
tags:
  - Competitive Programming
  - Backtracking
  - Matrix
---

Put exactly one queen in each row; the only question is which column. A column is legal for the current row when no earlier queen shares its column or either diagonal. Track those three things as you descend and the check is $O(1)$.

## Description

Place `n` queens on an `n x n` chessboard so that no two attack each other (no shared row, column, or diagonal). Return all distinct board configurations, each as a list of strings using `Q` and `.`.

**Example**

```
Input:  n = 4
Output: [[".Q..","...Q","Q...","..Q."],
         ["..Q.","Q...","...Q",".Q.."]]

Input:  n = 1
Output: [["Q"]]
```

**Constraints**

- $1 \le n \le 9$

## Prerequisites

- Row-by-row recursion (one decision per row).
- The diagonal identities: cells on a `↘` diagonal share `r + c`; cells on a `↙` diagonal share `r - c`.

## Approach 1: Backtracking with a column scan

### Intuition

Recurse on the row. For each candidate column, scan all previously placed queens to check the column and both diagonals directly. Simple, no auxiliary sets, but each check is $O(n)$.

### Algorithm

1. `board` is a list of chosen columns, one per placed row.
2. `dfs(row)`: if `row == n`, render and record the board; return.
3. For `col` in `range(n)`: if `safe(row, col)`, set `board[row] = col`, `dfs(row + 1)`, undo.
4. `safe(row, col)`: for each earlier `r`, reject if `board[r] == col` or `abs(board[r] - col) == row - r`.

```python
def solveNQueens(n: int) -> list[list[str]]:
    res = []
    board = [-1] * n

    def safe(row, col):
        for r in range(row):
            if board[r] == col or abs(board[r] - col) == row - r:
                return False
        return True

    def render():
        return ["".join("Q" if board[r] == c else "." for c in range(n)) for r in range(n)]

    def dfs(row):
        if row == n:
            res.append(render())
            return
        for col in range(n):
            if safe(row, col):
                board[row] = col
                dfs(row + 1)
                board[row] = -1

    dfs(0)
    return res
```

### Complexity

- **Time:** $O(n!)$ leaves times $O(n)$ per safety scan and $O(n^2)$ per rendered board.
- **Space:** $O(n)$ recursion and `board`, excluding output.

## Approach 2: Backtracking with hash sets

### Intuition

Maintain three sets: occupied columns, occupied `↘` diagonals keyed by `r + c`, and occupied `↙` diagonals keyed by `r - c`. A placement is legal iff none of its three keys is in its set. Every check and update is $O(1)$.

### Algorithm

1. `cols`, `pos_diag` (`r + c`), `neg_diag` (`r - c`) as sets.
2. `dfs(row)`: at `row == n`, record.
3. For `col` in `range(n)`: skip if `col in cols` or `row + col in pos_diag` or `row - col in neg_diag`. Otherwise add all three keys, place the queen, recurse, then remove all three and clear the cell.

```python
def solveNQueens(n: int) -> list[list[str]]:
    res = []
    board = [["."] * n for _ in range(n)]
    cols, pos_diag, neg_diag = set(), set(), set()

    def dfs(row):
        if row == n:
            res.append(["".join(r) for r in board])
            return
        for col in range(n):
            if col in cols or (row + col) in pos_diag or (row - col) in neg_diag:
                continue
            cols.add(col)
            pos_diag.add(row + col)
            neg_diag.add(row - col)
            board[row][col] = "Q"
            dfs(row + 1)
            cols.remove(col)
            pos_diag.remove(row + col)
            neg_diag.remove(row - col)
            board[row][col] = "."

    dfs(0)
    return res
```

### Complexity

- **Time:** $O(n!)$ — the branching collapses fast once three constraints filter columns; rendering each solution is $O(n^2)$.
- **Space:** $O(n)$ for the three sets and recursion.

## Approach 3: Backtracking with boolean arrays

### Intuition

The sets become fixed-size boolean arrays: `cols[n]`, `pos_diag[2n - 1]` indexed by `r + c`, `neg_diag[2n - 1]` indexed by `r - c + (n - 1)` to shift the range into `[0, 2n - 2]`. Array indexing is a touch faster than set hashing.

### Algorithm

Identical control flow to Approach 2, with `cols[col]`, `pos_diag[row + col]`, `neg_diag[row - col + n - 1]` as the occupancy flags, set to `True` before recursing and `False` after.

```python
def solveNQueens(n: int) -> list[list[str]]:
    res = []
    board = [["."] * n for _ in range(n)]
    cols = [False] * n
    pos_diag = [False] * (2 * n - 1)
    neg_diag = [False] * (2 * n - 1)

    def dfs(row):
        if row == n:
            res.append(["".join(r) for r in board])
            return
        for col in range(n):
            pd, nd = row + col, row - col + n - 1
            if cols[col] or pos_diag[pd] or neg_diag[nd]:
                continue
            cols[col] = pos_diag[pd] = neg_diag[nd] = True
            board[row][col] = "Q"
            dfs(row + 1)
            cols[col] = pos_diag[pd] = neg_diag[nd] = False
            board[row][col] = "."

    dfs(0)
    return res
```

### Complexity

- **Time:** $O(n!)$.
- **Space:** $O(n)$.

## Approach 4: Backtracking with bitmasks

### Intuition

Represent the three occupancy sets as the bits of three integers. The set of columns still open for this row is `~(cols | pos | neg)` masked to `n` bits; iterate its set bits. On descent, shift the diagonal masks — `pos` shifts left, `neg` shifts right — because a `↘` diagonal in the next row is one column over.

### Algorithm

1. `dfs(row, cols, pos, neg)`: at `row == n`, record the reconstructed board.
2. `available = ((1 << n) - 1) & ~(cols | pos | neg)`.
3. While `available`: take `bit = available & -available`; recurse with `cols | bit`, `(pos | bit) << 1`, `(neg | bit) >> 1`; clear `bit` from `available`.

```python
def solveNQueens(n: int) -> list[list[str]]:
    res = []
    placements = []

    def dfs(row, cols, pos, neg):
        if row == n:
            res.append(["".join("Q" if c == placements[r] else "." for c in range(n))
                        for r in range(n)])
            return
        available = ((1 << n) - 1) & ~(cols | pos | neg)
        while available:
            bit = available & -available
            available ^= bit
            col = bit.bit_length() - 1
            placements.append(col)
            dfs(row + 1, cols | bit, (pos | bit) << 1, (neg | bit) >> 1)
            placements.pop()

    dfs(0, 0, 0, 0)
    return res
```

### Complexity

- **Time:** $O(n!)$, with the smallest constant factor of the four.
- **Space:** $O(n)$ recursion.

## Common Pitfalls

- **Wrong diagonal key.** `↘` diagonals share `r + c`; `↙` diagonals share `r - c`. Swapping them, or using `r - c` for both, lets attacking queens through.
- **Negative index for `neg_diag`.** `r - c` ranges over `-(n-1) .. n-1`; add `n - 1` before indexing an array.
- **Not undoing all three marks.** Column and both diagonals must be cleared on the way back, along with the board cell.
- **Bitmask diagonal shift direction.** `pos` shifts left going down a row, `neg` shifts right; reversing them corrupts the constraint.
- **Rendering cost ignored.** Building the string board is $O(n^2)$ per solution — fine here, but do it only at a leaf.

## The keystone

N-Queens is the archetypal constraint-satisfaction backtrack: one variable per row, a small domain, and three constant-time consistency checks that kill most of the `n!` tree before it grows. The progression from column-scan to sets to bitmask is the same optimisation each time — replace an $O(n)$ feasibility check with an $O(1)$ one. That closes the [Backtracking](/citadel/dsa/backtracking) section; the constraint-propagation idea reappears in [Graphs](/citadel/dsa/graphs) as topological ordering.
