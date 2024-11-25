---
title: The 15-Puzzle - When Is a Position Solvable?
description: Deciding in O(n) whether a sliding-tile position can reach the solved state, using the invariant that every legal move preserves the parity of the tile permutation combined with the blank's row.
date: 2024-11-25
draft: false
slug: /cp/fifteen-puzzle-solvability
tags:
  - Competitive Programming
  - Miscellaneous
  - Combinatorics
---

The 15-puzzle is a $4 \times 4$ grid holding tiles $1$ to $15$ and one blank; a move slides a tile orthogonally into the blank. Famously, exactly **half** of the $16!$ arrangements can be brought to the solved state — the other half are unreachable no matter how long you slide. A single parity check tells the two apart in linear time, so you never waste a BFS on an impossible board.

## The problem

Given a board (a permutation of $0, 1, \dots, rc-1$ on an $r \times c$ grid, with $0$ the blank), decide whether repeated legal slides can reach the goal $1, 2, \dots, rc-1, 0$.

## The invariant

Read the board row by row, skip the blank, and count **inversions**: pairs of tiles $(a, b)$ appearing in that order with $a > b$. Also note the blank's row.

A single slide does one of two things:

- **Horizontal move:** the blank swaps with a left/right neighbour. In the row-major reading the two cells are adjacent, so no tile order changes — inversions are unchanged, blank row unchanged.
- **Vertical move:** the blank swaps with a tile $c$ cells away in the reading. That tile jumps over exactly $c - 1$ other tiles, so the inversion count changes by an odd amount ($c - 1$ flips, each $\pm 1$, net parity $= c - 1 \bmod 2$). The blank's row also changes by $1$.

So define the quantity

$$P = \text{(number of inversions)} + \text{(something that tracks the blank's row when } c \text{ is even)}.$$

- **Odd width $c$:** a vertical move jumps over an *even* number of tiles, so inversion parity is invariant on its own. The board is solvable **iff the number of inversions is even**.
- **Even width $c$ (the real 15-puzzle, $c = 4$):** a vertical move flips inversion parity *and* moves the blank one row. So "inversions + blank's row" has constant parity. Measuring the blank's row **from the bottom** (bottom row $= 1$), the board is solvable **iff** inversions $+$ blank-row-from-bottom is *odd*... equivalently: solvable iff (inversions even) $\iff$ (blank on an even row from the bottom).

The goal state has $0$ inversions and the blank in the bottom row, which fixes the constant.

## Algorithm

```python
def solvable(board, rows, cols):
    tiles = [x for x in board if x != 0]
    inversions = sum(
        1
        for i in range(len(tiles))
        for j in range(i + 1, len(tiles))
        if tiles[i] > tiles[j]
    )
    if cols % 2 == 1:
        return inversions % 2 == 0
    blank_row_from_bottom = rows - (board.index(0) // cols)
    if blank_row_from_bottom % 2 == 0:
        return inversions % 2 == 1
    return inversions % 2 == 0
```

`board` is the grid flattened in row-major order. Inversion counting with a [Fenwick tree](/citadel/cp/fenwick-tree) or merge sort makes it $O(n \log n)$ for large boards; the $O(n^2)$ double loop is fine up to a few thousand tiles.

## Why it is exactly half

The legal moves generate only *even* permutations of the 16 cells (each move is a transposition of blank with a tile, but returning the blank home takes an even number of them). The reachable set is a single coset of the alternating-group action, and it has index $2$ in the full symmetric group — hence exactly half of all arrangements.

## Common pitfalls

- **Counting the blank as a tile.** Inversions are over the $rc - 1$ numbered tiles only; including $0$ corrupts the parity.
- **Blank row from the top vs bottom.** For even width you must measure from the *bottom* (or adjust the constant). Getting this backwards inverts every answer.
- **Assuming the $4\times4$ rule for all sizes.** Odd-width boards ($3\times3$ eight-puzzle, $5\times5$) use the pure "inversions even" rule; the blank row is irrelevant there.
- **Non-square boards.** The rule depends on the *width* (`cols`) parity, not on whether the board is square. A $3 \times 4$ board uses the even-width rule.
- **A different goal.** If the target is not $1, 2, \dots, rc-1, 0$, compute the inversion parity of the *start relative to the target permutation*, and compare blank rows between start and target.

## The keystone

Horizontal slides change nothing in the row-major reading; a vertical slide jumps a tile over $c - 1$ others and shifts the blank one row. So for odd width the inversion parity alone is invariant (solvable iff even), and for even width "inversions + blank row from the bottom" has fixed parity — an $O(n \log n)$ check that rules out half of all boards instantly.
