---
title: Burnside's Lemma and Polya Enumeration - Counting Up To Symmetry
description: Counting distinct configurations under a symmetry group by averaging the fixed points of each symmetry, the necklace and bracelet formulas it produces, and Polya's cycle-index refinement.
date: 2024-06-15
draft: false
slug: /cp/burnside-polya
tags:
  - Competitive Programming
  - Combinatorics
  - Group Theory
---

"How many distinct necklaces of $n$ beads in $k$ colours, where rotations count as the same?" Naively counting $k^n$ and dividing by $n$ is wrong (some necklaces have extra symmetry). Burnside's lemma gives the right answer: **average, over all symmetries, the number of configurations that symmetry leaves unchanged.**

## The problem

A finite group $G$ acts on a set $X$ of configurations (colourings, arrangements). Count the **orbits** — equivalence classes under $G$, i.e. configurations considered the same if some symmetry maps one to the other.

Example: necklaces of $4$ beads, $2$ colours, rotations only ($G = \mathbb{Z}_4$). Answer: $6$ (`0000, 0001, 0011, 0101, 0111, 1111` as representatives).

## Burnside's lemma

$$\#\text{orbits} = \frac{1}{|G|} \sum_{g \in G} |\text{Fix}(g)|,$$

where $\text{Fix}(g) = \{x \in X : g \cdot x = x\}$ is the set of configurations $g$ fixes.

**Why:** count pairs $(g, x)$ with $g \cdot x = x$ two ways. By $g$: $\sum_g |\text{Fix}(g)|$. By $x$: $\sum_x |\text{Stab}(x)|$, and $|\text{Stab}(x)| = |G| / |\text{Orbit}(x)|$ (orbit-stabiliser), so $\sum_x |G|/|\text{Orbit}(x)| = |G| \cdot \#\text{orbits}$. Equate.

The practical content: **you need $|\text{Fix}(g)|$ for each symmetry $g$**, which is usually easy — a colouring is fixed by $g$ iff it is constant on each cycle of $g$'s permutation of the positions, so $|\text{Fix}(g)| = k^{c(g)}$ where $c(g)$ is the number of cycles.

## How it works

Necklaces, $n = 4$, $k = 2$, rotations $\{r^0, r^1, r^2, r^3\}$:

| $g$ | permutation of positions | cycles $c(g)$ | $|\text{Fix}(g)| = 2^{c(g)}$ |
| --- | --- | --- | --- |
| $r^0$ | identity | 4 | 16 |
| $r^1$ | $(0\,1\,2\,3)$ | 1 | 2 |
| $r^2$ | $(0\,2)(1\,3)$ | 2 | 4 |
| $r^3$ | $(0\,3\,2\,1)$ | 1 | 2 |

$\#\text{orbits} = \frac{16 + 2 + 4 + 2}{4} = 6$.

## Standard closed forms

- **Necklaces** ($n$ beads, $k$ colours, rotations = $\mathbb{Z}_n$): $\dfrac{1}{n}\sum_{d \mid n} \varphi(n/d)\, k^{d}$. (A rotation by $j$ has $\gcd(n, j)$ cycles; group the $j$ by $d = \gcd(n, j)$, and $\varphi(n/d)$ of them share each $d$.)
- **Bracelets** (add reflections = dihedral $D_n$): average the necklace sum with the reflection contributions — for odd $n$, $n$ reflections each with $k^{(n+1)/2}$ fixed; for even $n$, $n/2$ through opposite vertices ($k^{n/2 + 1}$) and $n/2$ through edge midpoints ($k^{n/2}$).
- **Cube face colourings** ($k$ colours, rotation group of order $24$): $\dfrac{k^6 + 3k^4 + 12k^3 + 8k^2}{24}$.

## Polya enumeration theorem

A refinement: to count orbits **weighted by how many beads of each colour** (e.g. "necklaces with exactly $a$ red and $b$ blue"), substitute $x_i + y_i + \dots$ powers into the **cycle index**

$$Z(G) = \frac{1}{|G|}\sum_{g \in G} \prod_{\ell} a_{\ell}^{c_\ell(g)},$$

where $c_\ell(g)$ is the number of $\ell$-cycles of $g$. Setting each $a_\ell = x^\ell + y^\ell + \dots$ and reading off the coefficient of $x^a y^b \dots$ gives the count with that colour distribution. Plain Burnside is Polya with all $a_\ell = k$.

## Algorithm

```python
def count_necklaces(n: int, k: int) -> int:
    from math import gcd
    def phi(m):
        r, p = m, 2
        while p * p <= m:
            if m % p == 0:
                while m % p == 0: m //= p
                r -= r // p
            p += 1
        if m > 1: r -= r // m
        return r
    total = sum(phi(n // d) * k ** d for d in range(1, n + 1) if n % d == 0)
    return total // n

def burnside(group_perms: list[list[int]], k: int) -> int:
    """group_perms: each a permutation of positions [0..n)."""
    def cycles(p):
        seen = [False] * len(p)
        c = 0
        for i in range(len(p)):
            if not seen[i]:
                c += 1
                j = i
                while not seen[j]:
                    seen[j] = True
                    j = p[j]
        return c
    return sum(k ** cycles(p) for p in group_perms) // len(group_perms)
```

## Common pitfalls

- **Dividing $k^n$ by $|G|$.** Only correct when the action is free (no configuration has a non-trivial stabiliser) — usually false. Use the full Burnside sum.
- **Reflections.** "Rotations only" is $\mathbb{Z}_n$; "flips allowed" is $D_n$ with $2n$ elements — and the reflection fixed-point counts differ between odd and even $n$.
- **The sum must be divisible by $|G|$.** It always is (that is the lemma); if your code produces a non-integer, a fixed-point count is wrong.
- **Fixed points of $g$.** A colouring is fixed iff constant on each cycle of $g$, so $|\text{Fix}(g)| = k^{c(g)}$ — but only when colours are unconstrained. With "exactly $a$ of colour $1$" you need Polya's cycle index.
- **Large $n$ with the naive sum.** $\sum_{d \mid n}$ has only $O(d(n))$ terms; iterate divisors, not all $j \in [0, n)$.

## The keystone

The number of distinct configurations under a symmetry group is the average number of configurations fixed by each symmetry, $\frac{1}{|G|}\sum_g |\text{Fix}(g)|$, and $|\text{Fix}(g)| = k^{(\text{cycles of }g)}$. For cyclic symmetry this collapses to $\frac{1}{n}\sum_{d \mid n}\varphi(n/d)k^d$; Polya's cycle index handles fixed colour counts.
