---
title: Sparse Table - O(1) Idempotent Range Queries
description: Precomputing every power-of-two interval so a static range minimum (or gcd, or or) is two table lookups, why idempotence lets the two overlap, and the disjoint variant for non-idempotent operations.
date: 2024-02-22
draft: false
slug: /cp/sparse-table
tags:
  - Competitive Programming
  - Data Structures
  - Range Queries
---

If the array never changes and the operation is **idempotent** — $\min$, $\max$, $\gcd$, bitwise AND/OR — you can answer any range query in $O(1)$ after an $O(n \log n)$ precompute. The trick is to store the answer for every interval whose length is a power of two, then cover an arbitrary range with two of them that are allowed to overlap.

## The problem

Given a static array $a[0 \dots n-1]$ and an idempotent associative operation $\oplus$, answer many queries "$a[l] \oplus a[l+1] \oplus \dots \oplus a[r]$" — the classic being **range minimum query** (RMQ).

Example: $a = [5, 2, 4, 7, 6, 3, 1, 2]$. RMQ$(1, 5) = \min(2, 4, 7, 6, 3) = 2$.

## The idea

Let $\text{sp}[k][i]$ be $\oplus$ over $a[i \dots i + 2^k - 1]$. Build it by doubling:

$$\text{sp}[0][i] = a[i], \qquad \text{sp}[k][i] = \text{sp}[k-1][i] \;\oplus\; \text{sp}[k-1][i + 2^{k-1}].$$

To query $[l, r]$ of length $\ell = r - l + 1$, let $k = \lfloor \log_2 \ell \rfloor$. The two intervals $[l,\ l + 2^k - 1]$ and $[r - 2^k + 1,\ r]$ together cover $[l, r]$ and **overlap in the middle**. Because $\oplus$ is idempotent ($x \oplus x = x$), counting the overlap twice does no harm:

$$\text{query}(l, r) = \text{sp}[k][l] \;\oplus\; \text{sp}[k][r - 2^k + 1].$$

## How it works

$a = [5, 2, 4, 7, 6, 3, 1, 2]$, RMQ$(1, 5)$: length $5$, $k = \lfloor \log_2 5 \rfloor = 2$, $2^k = 4$. Left block $\text{sp}[2][1] = \min(a[1..4]) = \min(2,4,7,6) = 2$. Right block $\text{sp}[2][5 - 4 + 1] = \text{sp}[2][2] = \min(a[2..5]) = \min(4,7,6,3) = 3$. Answer $\min(2, 3) = 2$. The two blocks overlapped on $a[2..4]$; harmless for $\min$.

## Algorithm

```python
def build_sparse(a: list[int]):
    n = len(a)
    LOG = n.bit_length()
    sp = [a[:]]
    for k in range(1, LOG):
        prev = sp[k - 1]
        half = 1 << (k - 1)
        sp.append([min(prev[i], prev[i + half]) for i in range(n - (1 << k) + 1)])
    return sp

def query_min(sp, l: int, r: int) -> int:
    k = (r - l + 1).bit_length() - 1
    return min(sp[k][l], sp[k][r - (1 << k) + 1])
```

Precompute a `log` table (`log[1] = 0`, `log[i] = log[i//2] + 1`) if you want to avoid `bit_length` in a hot loop.

## Complexity

- **Build:** $O(n \log n)$ time and space.
- **Query:** $O(1)$ for idempotent $\oplus$; two lookups and one combine.

## The disjoint sparse table (non-idempotent ops)

For sums, products, or "matrix product" where double-counting the overlap *does* matter, the **disjoint sparse table** splits differently: at level $k$ it precomputes, for each block of size $2^{k+1}$, the prefix and suffix aggregates from the block's midpoint. A query finds the highest bit where $l$ and $r$ differ, which pins a level whose midpoint lies between them, and combines one suffix with one prefix — still $O(1)$ per query, $O(n \log n)$ build, no idempotence needed.

## Common pitfalls

- **Using it with updates.** Sparse table is static. One element change forces an $O(n \log n)$ rebuild — use a [Fenwick](/citadel/cp/fenwick-tree) or [segment tree](/citadel/data-structures/segment-tree) instead.
- **Non-idempotent op with the overlapping query.** RMQ tolerates the overlap; range *sum* would add the middle twice. Use a prefix-sum array (sum has an inverse) or the disjoint sparse table.
- **`k` off by one.** $k = \lfloor \log_2(r - l + 1) \rfloor$; `(len).bit_length() - 1`. Getting $k$ one too big reads out of the array.
- **Row lengths.** Level $k$ has only $n - 2^k + 1$ valid starts; sizing every row to $n$ wastes memory and invites out-of-range reads.
- **RMQ via LCA confusion.** RMQ reduces to [LCA on a Cartesian tree](/citadel/cp/lowest-common-ancestor) and vice versa; sparse table is the direct route when you just need the min.

## The keystone

Store $\oplus$ over every power-of-two interval, then cover any range with two overlapping blocks of size $2^{\lfloor \log_2 \ell \rfloor}$ — $O(1)$ per query because idempotence makes the overlap free. It is the fastest answer for *static* min/max/gcd; add updates and you drop back to a logarithmic structure.
