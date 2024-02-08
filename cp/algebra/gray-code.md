---
title: Gray Code - Counting Where Only One Bit Changes
description: The binary reflected Gray code, its one-line formula n xor (n shifted right), the inverse that recovers the index, and why "adjacent codes differ in a single bit" is the property contest problems want.
date: 2024-02-08
draft: false
slug: /cp/gray-code
tags:
  - Competitive Programming
  - Algebra
  - Number Systems
---

A Gray code is an ordering of all $2^k$ binary strings of length $k$ in which consecutive strings differ in exactly one bit. The standard one — the binary reflected Gray code — has a one-line formula in each direction, which is why it turns up in problems about hypercube walks, subset enumeration with cheap updates, and rotary encoders.

## The problem

Produce a sequence $g(0), g(1), \dots, g(2^k - 1)$ of distinct $k$-bit values such that $g(i)$ and $g(i+1)$ differ in a single bit (and, for the reflected code, so do $g(2^k - 1)$ and $g(0)$ — it is cyclic). Also: given a code word, recover its index $i$.

Example, $k = 3$: $000, 001, 011, 010, 110, 111, 101, 100$. Each step flips one bit.

## The idea

**Forward.** $g(n) = n \oplus (n \gg 1)$. XOR-ing $n$ with itself shifted right by one turns each bit into the parity of it and its left neighbour. Incrementing $n$ by $1$ changes a run of low bits in ordinary binary, but after this transform exactly one output bit flips — the one at the position of the lowest $0$ bit of $n$ (equivalently, the carry position).

**Inverse.** Recover $n$ from $g$ by a prefix-XOR of the bits from the top down:

$$n = g \oplus (g \gg 1) \oplus (g \gg 2) \oplus \dots$$

which in code is a loop `g ^= g >> shift` doubling the shift, or the branchless `n = g; while g: g >>= 1; n ^= g`.

## How it works

$k = 3$:

| $n$ | binary | $n \gg 1$ | $g(n) = n \oplus (n\gg1)$ |
| --- | --- | --- | --- |
| 0 | 000 | 000 | 000 |
| 1 | 001 | 000 | 001 |
| 2 | 010 | 001 | 011 |
| 3 | 011 | 001 | 010 |
| 4 | 100 | 010 | 110 |
| 5 | 101 | 010 | 111 |
| 6 | 110 | 011 | 101 |
| 7 | 111 | 011 | 100 |

Every adjacent pair in the last column differs in one bit, and so do rows $7$ and $0$.

## Algorithm

```python
def gray(n: int) -> int:
    return n ^ (n >> 1)

def inverse_gray(g: int) -> int:
    n = 0
    while g:
        n ^= g
        g >>= 1
    return n

def gray_sequence(k: int) -> list[int]:
    return [gray(n) for n in range(1 << k)]
```

## Where it is useful

- **Enumerating subsets with $O(1)$ transitions.** Walk $g(0), g(1), \dots$; between consecutive masks exactly one element is added or removed, so an incrementally maintained aggregate (sum, XOR, a DP value) updates in $O(1)$ instead of $O(k)$. The bit that changes at step $n$ is $\text{lowbit}(n{+}1)$'s position, i.e. `(n & -(n+1))` style — or just XOR consecutive codes.
- **Hypercube / bit-flip graphs.** A Gray code is a Hamiltonian path (cycle) on the $k$-dimensional hypercube. Problems that ask for an order of states where each step toggles one boolean are asking for a Gray code.
- **Minimising switching.** Rotary and linear position encoders use Gray code so a misread at a boundary is off by at most one.

## Common pitfalls

- **Assuming any single-bit-change order is "the" Gray code.** There are many; the *reflected* one is the code the formula produces. If a problem pins a specific order, check it is this one.
- **Inverse with a fixed shift.** `g ^ (g >> 1)` is the forward map, not the inverse. The inverse needs the full prefix-XOR (`while g: g >>= 1; n ^= g`).
- **Width.** `gray(n)` for `n` in `[0, 2^k)` stays within `k` bits, but the top code $g(2^{k-1}) = 11\ldots10$ variants — just be sure you allocated `k` bits, not `k-1`.
- **Non-power-of-two ranges.** The clean cyclic property is for a full $2^k$. A Gray code over $m$ values that is still cyclic exists only when $m$ is even.

## The keystone

The binary reflected Gray code is just $g(n) = n \oplus (n \gg 1)$, with the index recovered by a downward prefix-XOR. Its defining property — consecutive values differ in one bit — is exactly what lets subset-enumeration DPs update an aggregate in constant time per step and what makes it a Hamiltonian tour of the hypercube.
