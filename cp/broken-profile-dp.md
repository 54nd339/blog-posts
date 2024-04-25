---
title: Broken Profile DP - Tiling and the Parquet Problem
description: Filling a grid column by column while carrying a bitmask of which cells on the frontier stick out, the transition that fills one cell at a time, and why it counts domino tilings in O(nm 2^m).
date: 2024-04-25
draft: false
slug: /cp/broken-profile-dp
tags:
  - Competitive Programming
  - Dynamic Programming
  - Bitmask
---

"How many ways to tile an $n \times m$ grid with $1 \times 2$ dominoes?" and its many cousins are solved by sweeping the grid cell by cell and keeping a bitmask of the jagged boundary between filled and unfilled — the **broken profile**. The mask has one bit per row (or column), so the DP is $O(nm\,2^m)$ with $m$ the small dimension.

## The problem

Count (or optimise over) ways to fill an $n \times m$ grid with pieces — dominoes, L-trominoes, "each cell independently coloured subject to a local constraint" — where a piece placed at a cell can poke into not-yet-processed cells. Assume $m \le \sim 16$ so $2^m$ is manageable; orient the grid so $m$ is the smaller side.

Example: $2 \times 3$ grid, domino tilings — there are $3$.

## The idea

Process cells in column-major order: $(0,0), (1,0), \dots, (m{-}1,0), (0,1), \dots$. Maintain a mask of $m$ bits describing, for the current frontier, which cells are **already occupied by a piece placed earlier that pokes forward**.

`dp[i][mask]` = number of ways to have filled all cells before position $i$ (in the linear order), with `mask` recording which of the next $m$ cells (the current column's cells from row $r$ onward, wrapping) are pre-filled.

At cell $(r, c)$ with incoming `mask`:

- **Bit $r$ of mask is set** → this cell is already filled by a piece from the left/above; move to the next cell, clearing bit $r$, not placing anything.
- **Bit $r$ clear** → we must cover $(r, c)$ now. Options:
  - place a **vertical** domino covering $(r, c)$ and $(r{+}1, c)$ — requires $r + 1 < m$ and bit $r{+}1$ clear; set bit $r{+}1$ in the outgoing mask.
  - place a **horizontal** domino covering $(r, c)$ and $(r, c{+}1)$ — sets the bit for $(r, c{+}1)$ in the outgoing mask (that cell is one full column ahead, which after the bit-rotation is bit $r$).

Each transition advances by one cell and produces one or two successor masks. After all $nm$ cells, the answer is `dp` at the end with `mask == 0`.

## How it works

$2 \times 3$ grid, $m = 2$ (process 2-cell columns). Start `dp[(0,0)][00] = 1`.

- $(0,0)$, mask $00$: bit 0 clear. Vertical → covers $(0,0),(1,0)$, next cell $(0,1)$ with mask... the vertical fills this column entirely. Or horizontal → covers $(0,0),(0,1)$, mark $(0,1)$.
- Continue cell by cell. The bookkeeping resolves to exactly the $3$ tilings: (three vertical), (one vertical + two horizontal on top/bottom), and the mirror.

The mechanical version below is easier to trust than the hand trace.

## Algorithm

The convention that stays sane in practice is a **row-by-row profile**: the state between rows is a mask of which columns the row above pushed a vertical domino into. For each incoming mask, recursively enumerate every valid way to fill the current row, producing the mask it pushes to the next row.

```python
def domino_tilings(n: int, m: int) -> int:
    # n rows, m columns; caller ensures m = min(dimensions)

    def fill_row(col, cur, down_next, prev_pushed, results):
        # cur: columns already covered in this row; down_next: verticals pushed to next row
        if col == m:
            results.append(down_next)
            return
        if (prev_pushed >> col) & 1 or (cur >> col) & 1:   # this cell already covered
            fill_row(col + 1, cur, down_next, prev_pushed, results)
            return
        # vertical domino: covers (row, col), pushes (row+1, col)
        fill_row(col + 1, cur | (1 << col), down_next | (1 << col), prev_pushed, results)
        # horizontal domino: covers (row, col) and (row, col+1)
        if (col + 1 < m
                and not (prev_pushed >> (col + 1)) & 1
                and not (cur >> (col + 1)) & 1):
            fill_row(col + 2, cur | (3 << col), down_next, prev_pushed, results)

    dp = {0: 1}
    for _ in range(n):
        ndp = {}
        for pushed, ways in dp.items():
            results = []
            fill_row(0, 0, 0, pushed, results)
            for nd in results:
                ndp[nd] = ndp.get(nd, 0) + ways
        dp = ndp
    return dp.get(0, 0)                          # frontier must be empty after the last row
```

A finer-grained variant advances **one cell at a time** instead of a whole row, with `dp[pos][mask]` over $nm$ positions; it has the same $2^m$ state but simpler transitions (place nothing / vertical / horizontal at the first empty frontier cell). The row version above is easier to get right because the "which bit means which cell" question only arises within a single row.

## Complexity

- **Time:** $O(n \cdot 2^m \cdot \text{(row-fills per mask)})$. The number of ways to fill one row is a Fibonacci-like count, well below $2^m$, so in practice this runs comfortably for $m \le 16$–$18$.
- **Space:** $O(2^m)$ for the DP layer.

## Common pitfalls

- **Wrong small dimension.** Always make $m = \min(n, m)$; the mask width is the exponent.
- **Bit-position convention.** "One cell ahead in column-major order" is *not* the same bit as "one row down". Pick the row-mask formulation if the rotation logic keeps biting you.
- **Final mask nonzero.** The answer counts only states with an empty frontier at the end — a leftover set bit means a domino pokes outside the grid.
- **Overcounting horizontal placements.** A horizontal domino is placed once, from its left cell; do not also place it from its right cell.
- **Modulo.** Tiling counts explode; take everything mod the required prime as you go.

## The keystone

Sweep the grid keeping a bitmask of which frontier cells are already covered by pieces reaching forward; fill one cell (or one row) at a time, branching over the pieces that can cover the first empty cell. With the small dimension $m \le 16$, this is $O(nm\,2^m)$ — the standard tool for domino/tromino tilings and grid-colouring counts with local constraints.
