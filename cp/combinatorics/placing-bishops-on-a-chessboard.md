---
title: Placing Bishops on a Chessboard - Diagonal DP
description: Counting ways to place K non-attacking bishops on an n by n board by rotating 45 degrees so bishops become rooks, then a DP over diagonals that adds cells in increasing-length order.
date: 2024-07-05
draft: false
slug: /cp/placing-bishops-on-a-chessboard
tags:
  - Competitive Programming
  - Combinatorics
  - Dynamic Programming
---

Count the ways to place $K$ bishops on an $n \times n$ board so that no two attack each other. Bishops move diagonally, and the two diagonal directions are independent — rotating the board $45°$ turns bishops into rooks on a jagged board, and then it is a clean diagonal-by-diagonal DP.

## The problem

Place $K$ identical bishops on distinct cells of an $n \times n$ board with no two on a common diagonal (either direction). Output the count.

Small values: $n = 2$, all $K$: $K=0 \to 1$, $K=1 \to 4$, $K=2 \to 4$, $K=3 \to 0$. ($n=2$: the two "/" diagonals of length 1 and the two of length 2... the answer for $K=2$ on a $2\times2$ is $4$.)

## The idea

A bishop attacks along the two diagonal families:

- **anti-diagonals** ($i + j = \text{const}$),
- **main diagonals** ($i - j = \text{const}$).

Colour the board like a chessboard. A bishop on a black cell can never share a diagonal with one on a white cell — the two colours are completely independent problems. Within one colour, rotate coordinates by $45°$: cell $(i, j)$ maps to $(i + j,\ i - j)$, and the two diagonal constraints become "same row" and "same column" — bishops become **rooks** on a diamond-shaped board.

So: for each colour, solve "place some rooks on this staircase board, no two in a row or column", split by how many go on that colour, and combine.

## The DP

Order the diagonals of one direction (say the "$i + j = d$" diagonals of one colour) by **increasing length**: $\ell_1 \le \ell_2 \le \dots$. Process them one at a time.

`dp[d][k]` = number of ways to place $k$ non-attacking bishops using the first $d$ diagonals.

Transition: for diagonal $d$ of length $\ell_d$, either place no bishop on it (`dp[d-1][k]`), or place one. If we place one, it must avoid the $k - 1$ diagonals of the *other* direction already occupied — but because we sorted by length, every previously placed bishop sits on a shorter diagonal, and a shorter diagonal of the perpendicular family crosses this one in a cell that... the clean statement: the number of free cells on diagonal $d$ when $k - 1$ bishops are already placed is $\ell_d - (k - 1)$. Hence

$$dp[d][k] = dp[d-1][k] + \big(\ell_d - (k - 1)\big) \cdot dp[d-1][k-1].$$

Do this once for each colour (each has its own list of diagonal lengths), then combine: total for $K$ bishops $= \sum_{a + b = K} \text{black}[a] \cdot \text{white}[b]$.

## Diagonal lengths

For an $n \times n$ board, the diagonals in one direction have lengths $1, 2, \dots, n-1, n, n-1, \dots, 2, 1$. Split by colour: sorting each colour's lengths ascending is what makes "$\ell_d - (k-1)$ free cells" correct.

## Algorithm

```python
def bishops(n: int, K: int) -> int:
    if K > 2 * n - 1:
        return 0

    # diagonal lengths in one direction, grouped by colour of the diagonal
    lengths = [n - abs(n - 1 - d) for d in range(2 * n - 1)]   # 1,2,...,n,...,2,1
    colour = [[], []]
    for d, L in enumerate(lengths):
        colour[d % 2].append(L)
    for c in colour:
        c.sort()

    def solve(diags: list[int]) -> list[int]:
        m = len(diags)
        dp = [0] * (m + 1)
        dp[0] = 1
        for L in diags:
            for k in range(min(len(dp) - 1, L), 0, -1):
                dp[k] += (L - (k - 1)) * dp[k - 1]
        return dp                                     # dp[k] = ways to place k on this colour

    b = solve(colour[0])
    w = solve(colour[1])
    total = 0
    for a in range(len(b)):
        if 0 <= K - a < len(w):
            total += b[a] * w[K - a]
    return total
```

## Complexity

- **Time:** $O(n \cdot K)$ — a DP over $O(n)$ diagonals and $O(K)$ bishop counts, done twice, plus an $O(K)$ combine.
- **Space:** $O(K)$.

## Common pitfalls

- **Not separating colours.** Black and white diagonals never interact; treating all $2n - 1$ diagonals of one direction as one list double-blocks and undercounts.
- **Sorting by length.** The "$\ell_d - (k-1)$ free cells" formula only holds if diagonals are processed shortest first, so every earlier bishop lies on a strictly shorter perpendicular diagonal.
- **The other direction.** You DP over diagonals of *one* direction; the "$k - 1$ used" accounts for the perpendicular family automatically. Do not run a 2-D DP over both.
- **$K$ range.** At most $2n - 1$ non-attacking bishops fit ($2n - 2$ on an $n \times n$ for $n \ge 2$, actually — the two longest opposite-direction diagonals share endpoints); return $0$ past the max.
- **Modulus.** Counts grow fast; reduce `dp[k]` mod the required prime.

## The keystone

Bishops on different-coloured squares never interact, and a $45°$ rotation turns each colour's bishops into rooks. Processing that colour's diagonals shortest-first, `dp[d][k] = dp[d-1][k] + (ℓ_d - (k-1))·dp[d-1][k-1]` counts non-attacking placements; convolve the two colours' results for the final count.
