---
title: Fibonacci Numbers - Fast Doubling and Matrix Powers
description: From the linear recurrence to the O(log n) methods — matrix exponentiation and the fast-doubling identities — plus the Pisano period that makes Fibonacci modulo m eventually cyclic.
date: 2023-12-17
draft: false
slug: /cp/fibonacci-numbers
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

$F_0 = 0$, $F_1 = 1$, $F_n = F_{n-1} + F_{n-2}$. The sequence is a warm-up for a general idea: any linear recurrence can be evaluated at index $n$ in $O(\log n)$ time, either by raising a matrix to the $n$-th power or by a pair of "doubling" identities. Fibonacci is where both tricks are easiest to see.

## The problem

Compute $F_n \bmod m$ for $n$ up to, say, $10^{18}$. The naive recursion is exponential; the obvious loop is $O(n)$ and still too slow at that scale. We want $O(\log n)$.

Values to anchor on: $F_{10} = 55$, $F_{20} = 6765$, $F_{50} = 12{,}586{,}269{,}025$.

## Approach 1: Bottom-up loop

### Intuition

The recurrence only looks back two terms, so carry two rolling variables and step forward.

### Algorithm

1. If $n \le 1$, return $n$.
2. Set $(a, b) = (0, 1)$.
3. Repeat $n - 1$ times: $(a, b) \leftarrow (b,\; (a + b) \bmod m)$.
4. Return $b$.

```python
def fib_linear(n: int, m: int) -> int:
    a, b = 0, 1
    for _ in range(n):
        a, b = b, (a + b) % m
    return a
```

### Complexity

- **Time:** $O(n)$ additions.
- **Space:** $O(1)$.

Fine for $n \le 10^7$ or so; the baseline everything else beats.

## Approach 2: Matrix exponentiation

### Intuition

One recurrence step is a linear map:

$$\begin{pmatrix} F_{n+1} \\ F_n \end{pmatrix} = \begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix} \begin{pmatrix} F_n \\ F_{n-1} \end{pmatrix}.$$

Applying it $n$ times means raising that $2 \times 2$ matrix to the $n$-th power, and [binary exponentiation](/citadel/cp/binary-exponentiation) does that in $O(\log n)$ matrix multiplies. The identity $\begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}^{n} = \begin{pmatrix} F_{n+1} & F_n \\ F_n & F_{n-1} \end{pmatrix}$ reads $F_n$ straight off the corner.

### Algorithm

1. Define $2 \times 2$ matrix multiply modulo $m$.
2. Binary-exponentiate $\begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}$ to the power $n$.
3. Return the top-right entry.

```python
def mat_mul(A, B, m):
    return [[(A[0][0]*B[0][0] + A[0][1]*B[1][0]) % m,
             (A[0][0]*B[0][1] + A[0][1]*B[1][1]) % m],
            [(A[1][0]*B[0][0] + A[1][1]*B[1][0]) % m,
             (A[1][0]*B[0][1] + A[1][1]*B[1][1]) % m]]

def fib_matrix(n: int, m: int) -> int:
    result = [[1, 0], [0, 1]]        # identity
    base = [[1, 1], [1, 0]]
    while n > 0:
        if n & 1:
            result = mat_mul(result, base, m)
        base = mat_mul(base, base, m)
        n >>= 1
    return result[0][1]
```

### Complexity

- **Time:** $O(\log n)$ matrix multiplies, each $O(k^3) = O(8)$ for $k = 2$, so $O(\log n)$.
- **Space:** $O(1)$.

The template generalises: any order-$k$ linear recurrence becomes a $k \times k$ companion matrix, evaluated in $O(k^3 \log n)$.

## Approach 3: Fast doubling

### Intuition

Two identities collapse index $2k$ and $2k+1$ into functions of index $k$:

$$F_{2k} = F_k\,(2 F_{k+1} - F_k), \qquad F_{2k+1} = F_{k+1}^2 + F_k^2.$$

Recurse on $\lfloor n/2 \rfloor$, then combine. Same $O(\log n)$ as the matrix method but with fewer multiplications and no matrix bookkeeping — the fastest in practice.

### Algorithm

1. Recurse on $n \gg 1$ to get $(F_k,\; F_{k+1})$.
2. Compute $c = F_k(2F_{k+1} - F_k)$ and $d = F_k^2 + F_{k+1}^2$, all modulo $m$.
3. If the low bit of $n$ is $0$, return $(c, d)$; else return $(d,\; c + d)$.

```python
def fib_fast(n: int, m: int) -> int:
    def rec(k):
        if k == 0:
            return (0, 1)
        a, b = rec(k >> 1)                       # F(k>>1), F((k>>1)+1)
        c = (a * ((2 * b - a) % m)) % m          # F(2*(k>>1))
        d = (a * a + b * b) % m                  # F(2*(k>>1)+1)
        return (d, (c + d) % m) if k & 1 else (c, d)
    return rec(n)[0]
```

### Complexity

- **Time:** $O(\log n)$ big-integer multiplications.
- **Space:** $O(\log n)$ recursion depth.

## Fibonacci modulo m: the Pisano period

The sequence $F_n \bmod m$ is eventually periodic (in fact purely periodic), with period $\pi(m)$ called the **Pisano period**. It never exceeds $6m$, and $\pi(10) = 60$. If you need many queries $F_{n_i} \bmod m$ for one modulus, precompute one period and index into it; each query is then $O(1)$ after an $O(\pi(m))$ setup.

## Common pitfalls

- **Off-by-one in the index convention.** With $F_0 = 0$, the loop in Approach 1 runs $n$ times and returns `a`. Decide once whether your $F_1$ is the first or second $1$ and stay consistent across approaches.
- **`2 * b - a` going negative under a modulus.** Reduce it as `(2 * b - a) % m` before multiplying; Python's `%` returns a non-negative result, but in C++ you must add `m`.
- **Matrix multiply without reducing.** Reduce every entry mod $m$ inside `mat_mul`, not just at the end, or the products blow past 64 bits in fixed-width languages.
- **Using Binet's formula.** $F_n = (\varphi^n - \psi^n)/\sqrt5$ loses precision past $n \approx 70$ in floating point — never use it for exact contest answers.

## The keystone

Fibonacci is the smallest example of a linear recurrence, and it shows the two ways to jump to index $n$ in logarithmic time: exponentiate the transition matrix, or use doubling identities. The matrix method is the one that transfers — replace the $2 \times 2$ with a $k \times k$ companion matrix and you can evaluate any constant-coefficient recurrence, or count length-$n$ walks in a graph, in $O(k^3 \log n)$.
