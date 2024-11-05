---
title: The Josephus Problem - The Recurrence and the O(n) Loop
description: Finding which of n people arranged in a circle survives when every k-th is eliminated - the shifting recurrence J(n) = (J(n-1) + k) mod n that solves it in O(n), the O(k log n) method for small k, and the closed form when k = 2.
date: 2024-11-05
draft: false
slug: /cp/josephus-problem
tags:
  - Competitive Programming
  - Miscellaneous
  - Sequences
---

$n$ people stand in a circle, numbered $0$ to $n-1$. Starting the count at person $0$, every $k$-th person is eliminated, the circle closes up, and counting resumes from the next person. Which position survives? Simulating with a list is $O(nk)$; a one-line recurrence does it in $O(n)$.

## The problem

Return the $0$-indexed position of the survivor, given $n$ and the step $k$. Common variants: report the whole elimination order; find the survivor for enormous $n$ with small $k$; the $k = 2$ special case; counting from a different start or in the other direction (all handled by an index shift at the end).

Example: $n = 7$, $k = 3$. Eliminations: $2, 5, 1, 6, 4, 0$; survivor $3$.

## The idea: renumber after each elimination

Solve for a circle of $1$ person: the survivor is at position $0$, so $J(1) = 0$.

Now go from $n - 1$ people to $n$. In a circle of $n$, the first person eliminated is at index $(k - 1) \bmod n$. After removing them, $n - 1$ people remain, and if we **renumber** starting from the person right after the eliminated one, we have exactly the $(n-1)$-person problem — whose answer we know is $J(n-1)$.

Translate that back to the original numbering: the renumbered position $0$ corresponds to original position $k \bmod n$, so

$$J(n) = \big(J(n-1) + k\big) \bmod n.$$

Iterate from $2$ up to $n$.

## Algorithm

```python
def josephus(n, k):
    survivor = 0                       # J(1) = 0
    for size in range(2, n + 1):
        survivor = (survivor + k) % size
    return survivor                    # 0-indexed
```

For **1-indexed** output add $1$. To start counting from person $s$, return `(survivor + s) % n`.

### Small k, huge n: O(k log n)

One full pass around a circle of size $m$ eliminates $\lfloor m/k \rfloor$ people, shrinking it to $m - \lfloor m/k \rfloor$ — a factor of about $\frac{k-1}{k}$. So the size reaches $1$ in $O(k \log n)$ passes, and each pass is a constant-work index adjustment.

```python
def josephus_fast(n, k):
    if n == 1:
        return 0
    if k == 1:
        return n - 1
    if k > n:                                   # fewer than one full pass left
        return (josephus_fast(n - 1, k) + k) % n
    removed = n // k                            # people eliminated in this pass
    res = josephus_fast(n - removed, k)         # survivor index in the shrunk circle
    res -= n % k                                # shift back past the pass's start
    if res < 0:
        res += n                                # wrapped around the circle
    else:
        res += res // (k - 1)                   # re-expand past the eliminated people
    return res
```

This matches the $O(n)$ recurrence exactly; it just collapses each sweep into one step.

### k = 2: closed form

Write $n = 2^m + \ell$ with $0 \le \ell < 2^m$. The survivor (1-indexed) is $2\ell + 1$ — equivalently, take the binary representation of $n$ and rotate its leading $1$ to the end.

```python
def josephus_k2(n):
    return 2 * (n - (1 << (n.bit_length() - 1))) + 1     # 1-indexed
```

## Complexity

- **General recurrence:** $O(n)$ time, $O(1)$ space.
- **Small $k$:** $O(k \log n)$.
- **$k = 2$:** $O(1)$.
- **Full elimination order:** $O(n \log n)$ with an order-statistics tree / Fenwick tree over "alive" flags (find the $j$-th alive person, delete, repeat).

## Common pitfalls

- **Index base.** The recurrence yields a $0$-indexed answer. Mixing in a $1$-indexed mental model off-by-ones every test.
- **`% size`, not `% n`.** At step `size`, the modulus is the current circle size, which grows each iteration. Using the final $n$ throughout is wrong.
- **Counting convention.** "Every $k$-th" means $k - 1$ people are skipped and the $k$-th is removed. If your problem eliminates the person you *land on* after $k$ steps from the last removal, that is the same $k$; if it is $k$ steps from the *next* person, adjust by one.
- **$k = 1$.** People are eliminated in order $0, 1, 2, \dots$; the survivor is $n - 1$. The general loop handles this, but the small-$k$ shortcut must special-case it.
- **Huge $n$ with large $k$.** No shortcut applies; $O(n)$ is the best general algorithm, so $n \sim 10^7$ is the practical ceiling.

## The keystone

Eliminating the first victim in a circle of $n$ and renumbering from the next survivor turns the problem into the $(n-1)$-circle, shifted by $k$: $J(n) = (J(n-1) + k) \bmod n$ with $J(1) = 0$. That is an $O(n)$ loop; $k = 2$ collapses to a bit rotation of $n$, and small $k$ admits block jumps for $O(k \log n)$.
