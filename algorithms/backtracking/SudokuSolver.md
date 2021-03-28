---
title: Solving Sudoku with Backtracking
description: Fill a 9-by-9 grid so every row, column, and 3-by-3 box holds the digits 1 to 9 once each. Backtracking fills empty cells one at a time, trying digits 1 to 9, and rewinds the moment a cell has no legal digit.
date: 2021-03-28
draft: false
slug: /algorithms/SudokuSolver
tags:
  - Algorithms
  - Backtracking
---

A Sudoku grid is solved when every row, every column, and every $3 \times 3$ box contains 1 through 9 exactly once. It's a **constraint satisfaction problem**, and backtracking solves it the same way a person does under pressure: pick an empty cell, try a digit that doesn't obviously break a rule, move on, and if you get stuck, undo and try the next digit.

## The idea

1. Find an empty cell. If there are none, the grid is solved.
2. Try each digit 1–9 in that cell.
3. If the digit already appears in the cell's row, column, or box, skip it.
4. Otherwise write it and recurse. If the recursion reports success, done.
5. If it fails, erase the digit and try the next. If all nine fail, return failure so the previous cell reconsiders.

Unlike N-queens, you usually want *a* solution, not all of them, so the recursion returns a boolean and stops at the first complete grid.

## The code

```python
def solve(grid):                    # grid: 9x9 list of lists, 0 = empty; solved in place
    def valid(r, c, d):
        if any(grid[r][j] == d for j in range(9)):        # row
            return False
        if any(grid[i][c] == d for i in range(9)):        # column
            return False
        br, bc = 3 * (r // 3), 3 * (c // 3)
        return all(grid[br + i][bc + j] != d               # 3x3 box
                   for i in range(3) for j in range(3))

    def backtrack():
        for r in range(9):
            for c in range(9):
                if grid[r][c] == 0:
                    for d in range(1, 10):
                        if valid(r, c, d):
                            grid[r][c] = d
                            if backtrack():
                                return True
                            grid[r][c] = 0        # undo
                    return False                  # no digit works here
        return True                               # no empty cell left

    return backtrack()


puzzle = [
    [5,3,0, 0,7,0, 0,0,0], [6,0,0, 1,9,5, 0,0,0], [0,9,8, 0,0,0, 0,6,0],
    [8,0,0, 0,6,0, 0,0,3], [4,0,0, 8,0,3, 0,0,1], [7,0,0, 0,2,0, 0,0,6],
    [0,6,0, 0,0,0, 2,8,0], [0,0,0, 4,1,9, 0,0,5], [0,0,0, 0,8,0, 0,7,9],
]
assert solve(puzzle) and puzzle[0] == [5, 3, 4, 6, 7, 8, 9, 1, 2]
```

Scanning for the first empty cell on every call is simple but wasteful; a real solver picks the empty cell with the fewest legal digits (the "most constrained" cell), which prunes far harder.

## Cost

With $k$ empty cells and 9 digits to try each, the worst case is $O(9^k)$ — but the row/column/box checks eliminate most branches immediately, and a well-posed puzzle solves near-instantly. Extra space is just the $O(k)$ recursion stack; the grid is edited in place.

Generalised from $9 \times 9$ to $n^2 \times n^2$, Sudoku is NP-complete — there's no known way to avoid the exponential worst case.

## Sudoku is graph coloring

Make each of the 81 cells a vertex, join two vertices whenever they share a row, column, or box, and a solution is a proper 9-[coloring](/citadel/algorithms/GraphColoring) of that graph where the givens are pre-coloured. The backtracking here is exactly the graph-coloring backtracking with a fixed structure — and the same four moves (try, check, recurse, undo) drive [N-queens](/citadel/algorithms/NQueens) and the [Hamiltonian cycle](/citadel/algorithms/HamiltonianCycle).
