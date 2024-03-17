---
title: Sqrt Tree - O(1) Range Queries With O(n log log n) Build
description: A layered structure that answers any associative range query in constant time without needing idempotence, by recursing the sqrt-decomposition idea and precomputing block prefixes, suffixes, and a between-blocks table.
date: 2024-03-17
draft: false
slug: /cp/sqrt-tree
tags:
  - Competitive Programming
  - Data Structures
  - Range Queries
---

A [sparse table](/citadel/cp/sparse-table) gives $O(1)$ range queries but only for **idempotent** operations like $\min$. A sqrt tree gives $O(1)$ range queries for **any** associative operation — sum, product, matrix product, "leftmost/rightmost with property" — at the cost of a slightly more elaborate build. It is niche (most problems are fine with a segment tree's $O(\log n)$), but it is the answer when you truly need constant-time queries on a non-idempotent monoid.

## The problem

Static array $a[0 \dots n-1]$, associative operation $\oplus$ (no inverse, no idempotence required). Answer many "$a[l] \oplus \dots \oplus a[r]$" in $O(1)$ each.

Example: $\oplus$ = matrix multiplication, $a$ a sequence of $2\times2$ transforms; query = the composed transform over a segment, in constant time.

## The idea

Split the array into $\sqrt n$ blocks of size $\sqrt n$. Precompute three things:

1. **Prefixes** within each block: `pref[b][i]` = $\oplus$ of that block from its start up to $i$.
2. **Suffixes** within each block: `suf[b][i]` = $\oplus$ from $i$ to the block's end.
3. **Between-blocks table** `between[i][j]` = $\oplus$ of whole blocks $i$ through $j$.

Then a query $[l, r]$:

- **Spanning two or more blocks:** the answer is `suf[block(l)][l] ⊕ between[block(l)+1][block(r)-1] ⊕ pref[block(r)][r]` — a suffix of the first block, the whole blocks in the middle from the table, and a prefix of the last block. Three precomputed pieces, at most two combines.
- **Same block:** none of the three pieces above spans it, so this case is handled by **recursion** — build a sqrt tree inside every block and delegate the query to it. The block size shrinks as $n \to \sqrt n \to n^{1/4} \to \dots$, reaching size $O(1)$ after $O(\log \log n)$ levels. That recursion depth is where the $O(n \log \log n)$ build cost comes from.

## The layers

At the top level, block size is $\sqrt n$. Level $k$ has block size $n^{1/2^{k}}$ (rounded to a power of two in practice). Each level stores its own prefixes, suffixes, and between-table over its own block layout. A query descends to the finest level where $l$ and $r$ still lie in different blocks of that level — there, the three-piece formula applies directly.

## Complexity

- **Build:** $O(n \log \log n)$ time and space.
- **Query:** $O(1)$ — a fixed number of table lookups and combines, independent of range length.
- **Update:** $O(\sqrt n)$ for a point update (rebuild the affected block chain at each level); a variant supports it, but if you need updates a [segment tree](/citadel/data-structures/segment-tree) is usually the better trade.

## When to use it (rarely)

- Non-idempotent op **and** a genuine need for $O(1)$ queries — e.g. $10^7$ queries where an $O(\log n)$ factor is the difference between AC and TLE.
- Composing transforms (affine maps, $2\times2$ matrices, hash-combine) over static segments.

For almost everything else: idempotent op → sparse table; need updates or just $O(\log n)$ is fine → segment tree; weird query, $O(\sqrt n)$ acceptable → plain [sqrt decomposition](/citadel/cp/sqrt-decomposition).

## Common pitfalls

- **Reaching for it by default.** The constant factor and code volume rarely pay off. Confirm that $O(\log n)$ per query is actually too slow first.
- **Operation must be associative and consistently ordered.** For non-commutative $\oplus$ (matrix product), keep the left-to-right order in every prefix, suffix, and between combine.
- **Between-table size.** It is $O(\sqrt n \times \sqrt n) = O(n)$ per level — fine, but summed naively across levels people sometimes allocate too much; each level's table is over *its* block count.
- **Same-block base case.** The recursion must bottom out at block size $\le 2$ (or a small constant) with a direct combine, or same-block queries are not $O(1)$.
- **Updates.** The plain sqrt tree is static; do not assume point updates without the update-supporting variant.

## The keystone

A sqrt tree recurses the sqrt-decomposition split $O(\log\log n)$ times, and at each level precomputes block prefixes, suffixes, and a whole-block between-table — so any range splits into three precomputed pieces and answers in $O(1)$, for **any** associative operation. It is the constant-time range query when [sparse table](/citadel/cp/sparse-table) can't apply because the operation has no idempotence.
