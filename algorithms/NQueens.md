---
title: The N-Queens Puzzle - A Royal Challenge Solved with Backtracking
description: Explore the classic N-Queens problem – placing N chess queens on an N×N board so none threaten each other. We dive into how a backtracking algorithm systematically finds all possible solutions.
date: 2023-02-21
draft: false
slug: /pensieve/algorithms/n-queens
tags:
  - DSA
  - Backtracking
---

Hey everyone, and welcome back to the blog! Chess is a game of strategy, and one of its most powerful pieces, the Queen, can move any number of squares horizontally, vertically, or diagonally. Now, imagine you have an N×N chessboard and N queens. The challenge? Place all N queens on the board such that no two queens threaten each other. This is the famous **N-Queens problem**.

It's a classic combinatorial problem and a fantastic example of how **backtracking algorithms** can be used to explore solutions to constraint satisfaction problems. It's not just a fun puzzle; it teaches us a powerful algorithmic technique applicable to many real-world scenarios where we need to find configurations that meet a specific set of rules.

## The Problem: A Royal Standoff
Let's formally define the N-Queens problem:

The task is to place $N$ chess queens on an $N \times N$ chessboard such that no two queens can attack each other. This means that a solution requires that no two queens share the same:
* **Row**
* **Column**
* **Diagonal** (in any of the four diagonal directions)

The problem is typically to find *all possible arrangements* of $N$ queens that satisfy these conditions, or sometimes just one such arrangement, or the total count of arrangements.

The N-Queens problem is named after the game of chess and is a well-known example used to illustrate the power and mechanics of backtracking.

## The Strategy: Backtracking to Place Queens Safely

A backtracking algorithm is a general algorithmic technique for finding all (or some) solutions to computational problems, notably constraint satisfaction problems. It incrementally builds candidates for the solutions and abandons a candidate ("backtracks") as soon as it determines that the candidate cannot possibly be completed to a valid solution.

For the N-Queens problem, the basic idea is:
1.  **Start with an empty chessboard.**
2.  Try to **place queens one by one, typically row by row** (or column by column). For each row, iterate through all possible columns to place a queen.
3.  **Check for Safety:** Before placing a queen in a cell `(row, col)`, check if this position is "safe" – i.e., if placing a queen there would conflict with any queens already placed in previous rows (in the same column or on the same diagonals).
4.  **If Safe, Place and Recur:** If the position is safe, place the queen there and recursively try to place queens in the subsequent rows.
5.  **If Not Safe or Recursive Call Fails (Backtrack):** If the current position is not safe, or if placing a queen here and attempting to solve for subsequent rows doesn't lead to a complete solution, then this path is not viable. "Backtrack" by removing the queen from the current cell and try placing it in the next available column in the current row.
6.  **Solution Found:** If queens have been successfully placed in all $N$ rows, a solution has been found. Print it or count it. Then, to find other solutions, backtrack to explore other possibilities.
7.  If all columns in the current row have been tried without success, backtrack to the previous row and try a different column placement for the queen in that row.

### Pseudocode for N-Queens

Here's a conceptual pseudocode for solving the N-Queens problem, aiming to print all solutions:

```pseudocode
// n: The size of the N×N chessboard (and the number of queens)

function N_Queens_Solver(n):
    // board[row][col] = 1 if a queen is placed, 0 otherwise.
    // Initialize an n x n board with all 0s.
    board = create_empty_board(n)
   
    // Call the recursive utility function to solve, starting from row 0.
    solve_N_Queens_Util(board, 0, n)

// Recursive utility function to place queens
// board: The current state of the chessboard
// row: The current row we are trying to place a queen in
// n: The size of the board
function solve_N_Queens_Util(board, current_row, n):
    // Base Case: If all queens are placed (i.e., we've successfully placed a queen in row n-1,
    // and now current_row is n), then print the board configuration.
    if current_row == n:
        print_board_solution(board)
        return // Found a solution, return to find more if backtracking continues

    // Try placing a queen in all columns of the current_row one by one
    for col_to_try = 0 to n-1:
        // Check if placing a queen at board[current_row][col_to_try] is safe
        if is_Safe_To_Place(board, current_row, col_to_try, n):
            // Place the queen
            board[current_row][col_to_try] = 1

            // Recur to place the rest of the queens in subsequent rows
            solve_N_Queens_Util(board, current_row + 1, n)
           
            // Backtrack: If placing queen in board[current_row][col_to_try]
            // doesn't lead to a solution (or after a solution is found and we want more),
            // remove the queen (unmark the cell) and try other columns.
            board[current_row][col_to_try] = 0
   
    // If no column in current_row leads to a solution, this function call returns,
    // causing backtracking in the previous call.

// Helper function to check if a queen can be placed at board[row][col]
// Assumes queens are placed row by row, so only need to check columns and diagonals
// in rows above the current 'check_row'.
function is_Safe_To_Place(board, check_row, check_col, n):
    // Check this column on upper rows
    for i = 0 to check_row - 1:
        if board[i][check_col] == 1:
            return false

    // Check upper-left diagonal
    // Loop while row i and column j are within board bounds
    for i = check_row, j = check_col; i >= 0 and j >= 0; i = i - 1, j = j - 1:
        if board[i][j] == 1:
            return false

    // Check upper-right diagonal
    // Loop while row i is within board bounds and column j is within board bounds
    for i = check_row, j = check_col; i >= 0 and j < n; i = i - 1, j = j + 1:
        if board[i][j] == 1:
            return false
           
    // If no conflicts, it's safe
    return true
```

### Explanation of the Algorithm:

1. **`N_Queens_Solver(n)` (Main Function):**
    * Creates an empty $N \times N$ chessboard (e.g., a 2D array initialized to 0s).
    * Calls a recursive helper function `solve_N_Queens_Util` to start placing queens, typically beginning with the first row (row 0).

2. **`solve_N_Queens_Util(board, current_row, n)` (Recursive Helper)**:
    * **Base Case:** If `current_row` is equal to `n`, it means we have successfully placed a queen in every row from 0 to $n-1$. A solution has been found! Print the board configuration. The function then returns (and backtracking will occur naturally to find other solutions if the loop in the previous call continues).
    * **Recursive Step:** For the `current_row`, iterate through all possible columns (`col_to_try` from 0 to $n-1$):
        * **Check Safety:** Call `is_Safe_To_Place` to determine if placing a queen at `board[current_row][col_to_try]` is valid (i.e., it doesn't conflict with any queens already placed in previous rows).
        * **Place & Recur:** If the position is safe:
            * Place a queen at `board[current_row][col_to_try]` (e.g., set the cell value to 1).
            * Make a recursive call to `solve_N_Queens_Util` for the next row (`current_row + 1`).
        * **Backtrack:** After the recursive call returns (either because a solution was found and printed, or because it couldn't place queens in subsequent rows), we **backtrack**. This means we remove the queen from `board[current_row][col_to_try]` (e.g., set the cell value back to 0). This allows the loop to continue trying to place the queen in the next column of the `current_row`, exploring other possibilities.
    * If the loop finishes without any column in `current_row` leading to a complete solution, this recursive call implicitly returns, causing backtracking in the call that invoked it (for the previous row).

3. **`is_Safe_To_Place(board, check_row, check_col, n)` (Safety Check):**
    This crucial helper function checks if placing a queen at `(check_row, check_col)` is safe. Since we are placing queens row by row, we only need to check for conflicts in rows above check_row:
    * **Column Check:** Iterate upwards from `check_row - 1` to `0` in the same `check_col`. If a queen is found, it's not safe.
    * **Upper-Left Diagonal Check:** Iterate diagonally upwards and to the left from `(check_row, check_col)`. If a queen is found, it's not safe.
    * **Upper-Right Diagonal Check:** Iterate diagonally upwards and to the right from `(check_row, check_col)`. If a queen is found, it's not safe.
    * If no conflicts are found in any of these checks, the position is safe, and the function returns `true`.

## Proof of Correctness (Intuition)

The correctness of the N-Queens algorithm using backtracking can be understood through induction and its exhaustive search nature (within the constraints):

* **Base Case:** For small N (e.g., N=1), the algorithm trivially finds the solution. For N=2 or N=3, it correctly determines no solution exists.
* **Inductive Step:** Assume the algorithm correctly explores all valid partial solutions up to row `k-1`. When trying to place a queen in row k, the `isSafe` function ensures that any placement considered doesn't conflict with queens in rows `0` to `k-1`. The recursive call then attempts to solve the problem for rows `k+1` to `N-1`. If a sub-solution is found, it's combined with the current placement. If not, backtracking ensures that all valid placements in row `k` are tried. Since the algorithm systematically explores all possible valid placements in each row and backtracks when a path is invalid or exhausted, it is guaranteed to find all valid solutions (or determine if none exist).

## Complexity Analysis

* **Time Complexity:** This is a tricky one to pin down precisely with a simple formula for all N, as the pruning done by the `isSafe` check significantly reduces the search space compared to a naive $N^N$ placements. However, in the worst-case, the algorithm explores a large portion of the search tree.
    * There are $N$ choices for the first queen (first row), then up to $N-1$ for the second (though `isSafe` prunes this heavily based on column and diagonals), and so on.
    * While not a strict $N!$ (as queens in the same column are immediately disallowed), the number of valid placements is still very large and grows rapidly. The time complexity is often cited as being roughly O(N!) or in that exponential realm because it explores permutations of queen placements across rows, heavily pruned by the safety checks. More rigorous analyses exist, but for practical purposes, it's considered exponential.
* **Space Complexity:** The primary space usage is for:
    * The $N \times N$ chessboard: O(N²).
    * The recursion call stack: In the worst case, the recursion depth can go up to $N$ (one call for each row). So, this is O(N). Therefore, the dominant factor for space is the board itself, leading to O(N²).

## Key Takeaways

* The **N-Queens problem** is a classic combinatorial puzzle that requires placing N queens on an N×N chessboard such that no two queens threaten each other.
* **Backtracking** is an effective algorithmic technique for solving this problem by systematically exploring possible queen placements, row by row, and undoing choices (backtracking) when a conflict is found or a path leads to a dead end.
* The **isSafe** function is crucial for checking if a potential queen placement conflicts with already placed queens in terms of rows, columns, and diagonals.
* While the worst-case time complexity is exponential (e.g., O(N!)), backtracking significantly prunes the search space compared to a naive brute-force of all $N^2$ choose $N$ positions.
* The space complexity is O(N²) for storing the board and O(N) for the recursion stack.

The N-Queens problem is a wonderful illustration of how recursive thinking and backtracking can be used to navigate complex search spaces and find solutions to constraint-satisfaction problems.