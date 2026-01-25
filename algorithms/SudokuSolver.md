---
title: Cracking the Code - Solving Sudoku with Backtracking
description: Dive into the classic Sudoku puzzle and learn how a backtracking algorithm can systematically find solutions by filling empty cells and validating constraints for rows, columns, and 3x3 subgrids.
date: 2023-02-23
draft: false
slug: /pensieve/algorithms/sudoku-solver
tags:
  - DSA
  - Backtracking
---

Hey everyone, and welcome back to the blog! Many of us have spent a pleasant hour (or perhaps a few frustrating ones!) trying to solve a Sudoku puzzle. That familiar 9x9 grid, partially filled with numbers, challenging us to complete it such that every row, every column, and every one of the nine 3x3 subgrids contains all digits from 1 to 9 exactly once.

The Sudoku puzzle is a fantastic example of a **constraint satisfaction problem**, and it serves as a perfect illustration of how **backtracking algorithms** can be used to navigate complex search spaces and find solutions. Today, let's explore how we can teach a computer to solve Sudoku using this powerful algorithmic technique!

## The Problem: The 9x9 Grid Challenge
Let's formally define the Sudoku Solver problem:

Given a partially filled $9 \times 9$ grid of numbers (where empty cells might be represented by 0 or a blank), the goal is to fill in the empty cells such that the completed grid satisfies the following Sudoku rules:
1.  Each **row** must contain all the digits from 1 to 9 exactly once.
2.  Each **column** must contain all the digits from 1 to 9 exactly once.
3.  Each of the nine **$3 \times 3$ subgrids** (also called "boxes" or "regions") of the an N×N chessboard must contain all of the digits from 1 to 9 exactly once.

The problem is to find a valid, complete solution to the given Sudoku puzzle.

## The Strategy: Backtracking to Fill the Grid

A backtracking algorithm is an ideal approach for solving Sudoku puzzles. It systematically tries to fill in the empty cells one by one, and if it hits a point where a current placement violates Sudoku rules or leads to a dead end, it "backtracks" (undoes the last placement) and tries a different number.

The general algorithm works as follows:

1.  **Find an Empty Cell:** Scan the Sudoku grid to find an unassigned (empty) cell. If no empty cells are found, the puzzle is solved!
2.  **Try Digits:** For the found empty cell, try placing each digit from 1 to 9.
3.  **Check Validity:** For each digit tried, check if placing it in the current empty cell is "valid" or "safe" according to Sudoku rules (i.e., the digit doesn't already exist in the current row, current column, or the current 3x3 subgrid).
4.  **If Valid, Recur:** If placing the digit is valid:
    * Place the digit in the cell.
    * Recursively call the solver function to try and solve the rest of the puzzle from this new state.
    * If the recursive call returns `true` (meaning a solution was found for the rest of the board), then our current placement was good, so return `true`.
5.  **If Not Valid or Recursive Call Fails (Backtrack):** If placing the digit is not valid, OR if placing it and then trying to solve the rest ofthe puzzle recursively fails (the recursive call returns `false`), then this digit was not the right choice for the current cell. "Backtrack" by resetting the cell to empty (e.g., assign 0) and try the next digit (from 1 to 9) in the current cell.
6.  **No Solution from this Cell:** If all digits from 1 to 9 have been tried for the current empty cell and none lead to a solution, it means a previous placement was incorrect. Return `false` to trigger backtracking in the previous recursive call.

### Pseudocode for Sudoku Solver

Here's a conceptual pseudocode that implements this backtracking logic:

```pseudocode
// grid: The 9x9 Sudoku grid (e.g., 0 represents an empty cell)

function SudokuSolver(grid):
    if solveSudokuUtil(grid):
        print "Solution Found:"
        print_grid(grid) // Helper function to display the solved grid
    else:
        print "No solution exists for this Sudoku puzzle."

// Recursive utility function to solve the Sudoku
function solveSudokuUtil(grid):
    // Find the next empty cell (row, col)
    // If no empty cell is found, the puzzle is solved!
    empty_cell_location = find_empty_cell(grid)
    if empty_cell_location is null: // No empty cells left
        return true // Puzzle solved!

    row = empty_cell_location.row
    col = empty_cell_location.col

    // Try filling the empty cell with digits from 1 to 9
    for digit = 1 to 9:
        // Check if it's safe to place 'digit' at grid[row][col]
        if isSafeToPlace(grid, row, col, digit):
            // Make the tentative assignment
            grid[row][col] = digit

            // Recursively try to solve the rest of the puzzle
            if solveSudokuUtil(grid):
                return true // If successful, a solution is found

            // If not successful, backtrack: undo the assignment
            grid[row][col] = 0 // Reset cell to empty
           
    // If no digit (1-9) can be placed in this cell to lead to a solution,
    // trigger backtracking by returning false.
    return false

// Helper function to check if placing 'digit' at grid[row][col] is valid
function isSafeToPlace(grid, row, col, digit):
    // Check if 'digit' is not already used in the current row,
    // current column, and current 3x3 subgrid.
    return not usedInRow(grid, row, digit) and \
           not usedInCol(grid, col, digit) and \
           not usedInBox(grid, row - row % 3, col - col % 3, digit)

// Helper function to check if 'digit' is used in the specified row
function usedInRow(grid, row, digit):
    for c = 0 to 8: // Iterate through columns of the given row
        if grid[row][c] == digit:
            return true // Digit found in row
    return false

// Helper function to check if 'digit' is used in the specified column
function usedInCol(grid, col, digit):
    for r = 0 to 8: // Iterate through rows of the given col
        if grid[r][col] == digit:
            return true // Digit found in column
    return false

// Helper function to check if 'digit' is used in the 3x3 subgrid (box)
// boxStartRow, boxStartCol are the top-left coordinates of the 3x3 box
function usedInBox(grid, boxStartRow, boxStartCol, digit):
    for r_offset = 0 to 2:
        for c_offset = 0 to 2:
            if grid[boxStartRow + r_offset][boxStartCol + c_offset] == digit:
                return true // Digit found in box
    return false

// Helper to find the next empty cell (e.g., returns {row, col} or null)
function find_empty_cell(grid):
    for r = 0 to 8:
        for c = 0 to 8:
            if grid[r][c] == 0: // Assuming 0 represents an empty cell
                return {row: r, col: c}
    return null // No empty cells
```

### Explanation of the Algorithm:

The `SudokuSolver` function typically initializes the process and then calls a recursive helper function, `solveSudokuUtil`.

The `solveSudokuUtil` function works as follows:
1. **Find Empty Cell:** It first searches for an empty cell in the grid. If no empty cells are found, it means the entire grid has been successfully filled according to Sudoku rules, so a solution is found, and the function returns `true`.
2. **Try Digits:** If an empty cell `(row, col)` is found, the function iterates through digits 1 to 9, attempting to place each digit in this empty cell.
3. **Check Safety:** For each digit, it calls `isSafeToPlace` to verify if placing this digit at `(row, col)` violates any Sudoku rules (i.e., checks for conflicts in the same row, column, and the 3x3 subgrid).
4. **Place and Recur:** If the digit is safe to place:
    * The digit is tentatively placed in `grid[row][col]`.
    * A recursive call is made to `solveSudokuUtil` to try and solve the rest of the puzzle with this new placement.
    * If this recursive call returns `true` (meaning a complete solution was found down that path), then the current digit choice was correct, and `true` is propagated up the call stack.
5. **Backtrack:** If the recursive call returns `false` (meaning placing the current digit did not lead to a solution for the rest of the board), or if the `isSafeToPlace` check itself failed, then the current digit was not the right choice for this cell. The algorithm backtracks by resetting `grid[row][col]` to empty (e.g., 0) and then the loop continues to try the next digit (from 1 to 9) for the current cell.
6. **No Valid Digit Found:** If all digits from 1 to 9 have been tried for the current empty cell and none of them lead to a solution, it means a previous placement in an earlier cell was incorrect. The function returns `false`, which triggers backtracking in the previous recursive call.

The helper functions `usedInRow`, `usedInCol`, and `usedInBox` perform the checks required by `isSafeToPlace` by iterating through the respective row, column, or 3x3 subgrid to see if the proposed digit already exists.

## Proof of Correctness (Intuition)

The correctness of the backtracking algorithm for Sudoku relies on its systematic and exhaustive exploration of all valid possibilities:

* **Systematic Search:** The algorithm methodically tries to fill each empty cell with valid digits (1-9).
* **Constraint Adherence:** The `isSafeToPlace` function ensures that each digit placed adheres to the fundamental Sudoku rules (no repeats in row, column, or 3x3 box) relative to the digits already placed.
* **Backtracking:** If a path of choices leads to a state where no valid digit can be placed in an empty cell (violating Sudoku rules if any digit were placed), the algorithm correctly backtracks, undoing recent choices and exploring alternative valid choices. Since the algorithm explores all valid partial configurations and backtracks from invalid ones, if a solution exists, it will eventually find it. If it explores all possibilities and fails to find a complete valid grid, it correctly determines that no solution exists.

## Complexity Analysis

* **Time Complexity:** For a standard $9 \times 9$ Sudoku grid, let $N=9$. If there are $E$ empty cells, and for each empty cell, we try up to $N$ digits, the brute-force search space can be very large. A loose upper bound is often cited as $O(N^{E})$ or $O(N^{N^2})$ (if most cells are empty). However, due to the constraints imposed by Sudoku rules, the actual number of valid states explored is significantly less. A common way to express it for an $N \times N$ grid (where $N$ is typically $n^2$ for an $n \times n$ subgrid system, e.g., $N=9$ means $n=3$) is roughly in the order of **$O(N^{N \times N})$** in the worst-case without sophisticated pruning, though for the standard $9 \times 9$ grid, practical performance is much better. The time complexity is more accurately described as $O(m^{V})$, where $m$ is the number of choices for each cell (9) and $V$ is the number of empty cells. For a $9 \times 9$ grid, this is $O(9^k)$ where $k$ is the number of empty cells.
* **Space Complexity:** The primary space usage is for:
    * Storing the $N \times N$ Sudoku grid: O(N²).
    * The recursion call stack: In the worst case, the recursion depth can go up to the number of empty cells (or $N^2$ if we consider filling every cell). So, this is O(N²). Therefore, the overall space complexity is O(N²).

## Key Takeaways

* The **Sudoku Solver problem** aims to fill a partially completed $9 \times 9$ grid according to Sudoku rules: each digit 1-9 must appear exactly once in each row, column, and $3 \times 3$ subgrid.
* **Backtracking** is a highly effective algorithmic technique for solving Sudoku. It involves trying to place digits in empty cells one by one, checking for validity at each step, and "backtracking" (undoing a choice) if a placement leads to a dead end or violates rules.
* The `isSafe` function is critical for ensuring that each digit placement adheres to Sudoku constraints.
* While the worst-case time complexity is exponential, backtracking efficiently prunes the search space, making it practical for solving most Sudoku puzzles.

Sudoku is not just a popular brain-teaser; it's a fantastic way to understand and apply the power of recursive backtracking algorithms to solve constraint satisfaction problems!