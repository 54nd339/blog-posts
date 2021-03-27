---
title: The N-Queens Puzzle - Backtracking Over a Chessboard
description: Place N chess queens on an N-by-N board so none attacks another. Trying every arrangement is hopeless; backtracking places one queen per row, abandons a partial board the moment two queens conflict, and finds every solution.
date: 2021-03-27
draft: false
slug: /algorithms/NQueens
tags:
  - Algorithms
  - Backtracking
---

A queen attacks along its row, its column, and both diagonals. The **N-queens problem** asks for all ways to place $N$ queens on an $N \times N$ board with no two attacking each other. For $N = 8$ there are $\binom{64}{8} \approx 4.4$ billion ways to drop 8 queens on 64 squares, and only 92 are solutions. You can't enumerate the first set. Backtracking never builds a board that's already broken.

## The idea

Every solution has exactly one queen per row (two in a row would attack along it). So represent a partial solution as "the column of the queen in each filled row so far", and fill rows top to bottom:

1. In the current row, try each column.
2. If a queen there would attack any queen already placed (same column, or same diagonal), skip it.
3. Otherwise place it and recurse on the next row.
4. When a row has no safe column, return — the caller will try its next column. This is the backtrack.
5. When all $N$ rows are filled, record the board.

The pruning is the whole point: the instant a partial placement conflicts, every completion of it is discarded without being examined.

For $N = 4$: row 0 tries column 0. Row 1 can't use column 0 (same column) or column 1 (diagonal), so it takes column 2. Row 2 now has no safe column — 0, 1, 3 all clash with a queen above — so it backtracks, row 1 tries column 3, row 2 takes column 1, row 3 has nothing safe, and the whole `col 0` branch fails. Starting row 0 at column 1 gives the solution $(1, 3, 0, 2)$. There are exactly two solutions for $N = 4$, and 92 for $N = 8$.

## The code

```python
def solve_n_queens(n):
    solutions = []
    cols = []                       # cols[r] = column of the queen in row r

    def safe(row, col):
        for r, c in enumerate(cols):
            if c == col or abs(c - col) == row - r:   # same column or same diagonal
                return False
        return True

    def place(row):
        if row == n:
            solutions.append(cols[:])
            return
        for col in range(n):
            if safe(row, col):
                cols.append(col)
                place(row + 1)
                cols.pop()          # backtrack

    place(0)
    return solutions


assert len(solve_n_queens(8)) == 92
assert solve_n_queens(2) == [] and solve_n_queens(3) == []
```

The diagonal test is the neat part: two queens at `(r, c)` and `(row, col)` share a diagonal exactly when the row gap equals the column gap, `abs(c - col) == row - r`.

## Cost

Because one column is fixed per row and `safe` rejects repeated columns, the search explores at most $N!$ leaf placements — far fewer in practice, since a diagonal conflict high up prunes an entire subtree. There's no simple closed form for the true count; it's exponential but heavily pruned. Space is $O(N)$ for the `cols` list plus $O(N)$ recursion depth — the single-array representation avoids an $O(N^2)$ board.

## The pattern

N-queens is the textbook shape of backtracking: a partial solution, a cheap consistency check, a recursive extend, and an undo. The same four moves solve [Sudoku](/citadel/algorithms/SudokuSolver), [graph coloring](/citadel/algorithms/GraphColoring), and the [Hamiltonian cycle](/citadel/algorithms/HamiltonianCycle) — all constraint-satisfaction problems where the win is refusing to explore any branch that's already lost.
