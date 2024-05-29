---
title: Gaussian Elimination - Linear Systems, Rank, and GF(2)
description: Row-reducing A x = b to read off a solution, the number of free variables, or inconsistency, with partial pivoting over the reals, a modular inverse pivot mod p, and the bitset XOR form over GF(2).
date: 2024-05-29
draft: false
slug: /cp/gauss-linear-system
tags:
  - Competitive Programming
  - Linear Algebra
  - Gaussian Elimination
---

Gaussian elimination row-reduces an augmented matrix to a form where the solution — or the fact that there isn't one, or that there are infinitely many — can be read off directly. It is $O(n^3)$ and the same procedure regardless of the arithmetic; only the "divide by the pivot" step changes between reals, integers mod a prime, and bits.

## The problem

Given $A \in F^{n \times m}$ and $\mathbf{b} \in F^{n}$, decide whether $A\mathbf{x} = \mathbf{b}$ has a solution; if so, produce one and report the dimension of the solution space (number of free variables = $m - \text{rank}(A)$).

Example: $x + y = 3,\ x - y = 1$ → $x = 2,\ y = 1$, unique. $x + y = 3,\ 2x + 2y = 7$ → inconsistent. $x + y = 3$ alone → one free variable.

## The idea

Maintain a "current row" and "current column". For each column:

1. **Find a pivot** — a row at or below the current row with a non-zero entry in this column. Over $\mathbb{R}$, pick the one with **largest absolute value** (partial pivoting) for numerical stability. If none, the column has no pivot — it is a free variable; move to the next column.
2. **Swap** that row up to the current row.
3. **Normalise** — divide the pivot row by the pivot value (over $\mathbb{R}$: real division; mod $p$: multiply by the [modular inverse](/citadel/cp/modular-inverse); GF(2): the pivot is already $1$).
4. **Eliminate** — subtract a multiple of the pivot row from every other row so their entry in this column becomes $0$.
5. Advance the current row.

After reduction: if any row is all zeros in $A$ but non-zero in $\mathbf{b}$, **no solution**. Otherwise set free variables to $0$ (or a parameter) and back-substitute.

## Algorithm (over the reals, partial pivoting)

```python
def gauss(a: list[list[float]], b: list[float]):
    n = len(a)
    m = len(a[0])
    aug = [row[:] + [b[i]] for i, row in enumerate(a)]
    where = [-1] * m                              # pivot column -> row
    row = 0
    for col in range(m):
        if row >= n:
            break
        piv = max(range(row, n), key=lambda r: abs(aug[r][col]))
        if abs(aug[piv][col]) < 1e-12:
            continue
        aug[row], aug[piv] = aug[piv], aug[row]
        where[col] = row
        inv = 1.0 / aug[row][col]
        aug[row] = [x * inv for x in aug[row]]
        for r in range(n):
            if r != row and abs(aug[r][col]) > 1e-12:
                f = aug[r][col]
                aug[r] = [x - f * y for x, y in zip(aug[r], aug[row])]
        row += 1
    for r in range(row, n):                       # zero rows: check consistency
        if abs(aug[r][m]) > 1e-9:
            return None, 0                        # no solution
    x = [0.0] * m
    for col in range(m):
        if where[col] != -1:
            x[col] = aug[where[col]][m]
    free = m - row
    return x, free                                # one solution; `free` free variables
```

### Modulo a prime

Replace `1.0 / aug[row][col]` with `pow(aug[row][col], MOD - 2, MOD)` and every `x - f*y` with `(x - f*y) % MOD`. No pivoting for stability is needed — any non-zero pivot works — but you still must pivot to *find* a non-zero entry.

### Over GF(2)

Pack each row (coefficients + RHS bit) into a Python integer used as a bitset. Elimination is `aug[r] ^= aug[row]` when bit `col` of `aug[r]` is set. This is the **linear (XOR) basis**:

```python
def xor_basis(nums: list[int], BITS: int = 60) -> list[int]:
    basis = [0] * BITS
    for x in nums:
        for bit in range(BITS - 1, -1, -1):
            if not (x >> bit) & 1:
                continue
            if basis[bit] == 0:
                basis[bit] = x
                break
            x ^= basis[bit]
    return [b for b in basis if b]                # size = dimension of the span

def max_xor(nums: list[int], BITS: int = 60) -> int:
    basis = [0] * BITS
    for x in nums:
        for bit in range(BITS - 1, -1, -1):
            if (x >> bit) & 1:
                if basis[bit]:
                    x ^= basis[bit]
                else:
                    basis[bit] = x
                    break
    best = 0
    for bit in range(BITS - 1, -1, -1):
        if basis[bit] and (best ^ basis[bit]) > best:
            best ^= basis[bit]
    return best
```

## Complexity

- **Time:** $O(n \cdot m \cdot \min(n, m))$, i.e. $O(n^3)$ for a square system. GF(2) with bitsets is $O(n^2 m / w)$ with word size $w$ (Python's big-int gives a large effective $w$).
- **Space:** $O(nm)$.

## Common pitfalls

- **No pivoting over the reals.** Skipping partial pivoting makes near-zero pivots blow up round-off; a system that is fine mathematically returns garbage.
- **Epsilon choice.** Comparing a float to exactly $0$ is wrong; use a tolerance ($10^{-9}$ish), and be aware it can misclassify a barely-consistent system.
- **Rank vs solvability.** A solution exists iff $\text{rank}(A) = \text{rank}([A \mid \mathbf{b}])$. The "zero row with non-zero RHS" check is exactly that test.
- **Free variables.** With $m - \text{rank}$ free variables there are infinitely many solutions (or $2^{\text{free}}$ over GF(2)); the code returns one — parametrise if the problem wants all or a count.
- **Modulo a non-prime.** The inverse-pivot step needs a *field*. For composite moduli, use fraction-free elimination or CRT over prime factors.

## The keystone

Gaussian elimination row-reduces $[A \mid \mathbf{b}]$ in $O(n^3)$; the pivot columns give a particular solution, the pivot-free columns are free variables, and a zero row with non-zero RHS means no solution. The only thing that changes with the arithmetic is the pivot division — a reciprocal, a [modular inverse](/citadel/cp/modular-inverse), or nothing at all over GF(2), where elimination becomes the XOR-basis trick.
