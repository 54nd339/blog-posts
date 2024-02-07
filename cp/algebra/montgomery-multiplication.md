---
title: Montgomery Multiplication - Fast Modular Products
description: Replacing the division in a mod b with shifts and multiplies by moving numbers into Montgomery form, why that speeds up long exponentiation loops, and why Python contestants rarely need it.
date: 2024-02-07
draft: false
slug: /cp/montgomery-multiplication
tags:
  - Competitive Programming
  - Algebra
  - Modular Arithmetic
---

Every step of a [binary exponentiation](/citadel/cp/binary-exponentiation) loop does `x = x * y % n`. The `% n` is an integer division, and division is several times slower than multiplication on real hardware. Montgomery multiplication removes it: work in a transformed representation where reduction modulo $n$ costs only shifts, multiplications, and one conditional subtraction.

## The problem

Compute many products $a \cdot b \bmod n$ for a fixed odd modulus $n$ (as in modular exponentiation, [Miller-Rabin](/citadel/cp/primality-tests), [Pollard's rho](/citadel/cp/integer-factorization)) without a hardware `div`/`mod` per product.

## The idea

Pick $r = 2^k > n$ with $\gcd(r, n) = 1$ (automatic since $n$ is odd). Map each residue $x$ to its **Montgomery form** $\bar x = x r \bmod n$. The core routine is **REDC**, which given $T < rn$ returns $T r^{-1} \bmod n$:

1. $q \leftarrow (T \bmod r) \cdot n' \bmod r$, where $n' = -n^{-1} \bmod r$ (precomputed once).
2. $t \leftarrow (T + q n) / r$ — this division is exact because $T + qn \equiv 0 \pmod r$, and dividing by $r = 2^k$ is a bit shift.
3. If $t \ge n$, subtract $n$. Return $t$.

Then $\overline{a} \cdot \overline{b}$ fed through REDC gives $(a r)(b r) r^{-1} = (ab) r \bmod n = \overline{ab}$ — a product in Montgomery form, computed with a mask, a multiply, an add, a shift, and maybe one subtraction. No division. Convert in with $\bar x = \text{REDC}(x \cdot (r^2 \bmod n))$ and out with $x = \text{REDC}(\bar x)$; the conversions cost one REDC each, amortised away over a long exponentiation.

## How it works

Take $n = 97$, $r = 2^7 = 128$. Precompute $n' = -97^{-1} \bmod 128 = 95$ and $r^2 \bmod n = 128^2 \bmod 97 = 88$.

Multiply $a = 12$ by $b = 34$ modulo $97$ (true answer $408 \bmod 97 = 20$):

1. **Into Montgomery form.** $\bar a = \text{REDC}(12 \cdot 88) = 81$ and $\bar b = \text{REDC}(34 \cdot 88) = 84$. (Indeed $12 \cdot 128 \bmod 97 = 81$ and $34 \cdot 128 \bmod 97 = 84$.)
2. **Multiply.** $\text{REDC}(\bar a \cdot \bar b) = \text{REDC}(81 \cdot 84) = \text{REDC}(6804) = 38$ — the product $20$ in Montgomery form ($20 \cdot 128 \bmod 97 = 38$).
3. **Out of Montgomery form.** $\text{REDC}(38) = 20$.

Every REDC is a mask (`& 127`), a multiply, an add, a shift (`>> 7`), and at most one subtraction — no division anywhere.

## Reference implementation

```python
class Montgomery:
    def __init__(self, n: int, k: int = 64):
        assert n % 2 == 1
        self.n = n
        self.k = k
        self.r = 1 << k
        self.mask = self.r - 1
        self.n_inv = (-pow(n, -1, self.r)) & self.mask     # -n^{-1} mod r
        self.r2 = (self.r * self.r) % n                    # r^2 mod n

    def redc(self, t: int) -> int:
        q = ((t & self.mask) * self.n_inv) & self.mask
        u = (t + q * self.n) >> self.k
        return u - self.n if u >= self.n else u

    def to_mont(self, x: int) -> int:
        return self.redc(x % self.n * self.r2)

    def from_mont(self, x_bar: int) -> int:
        return self.redc(x_bar)

    def mul(self, a_bar: int, b_bar: int) -> int:
        return self.redc(a_bar * b_bar)

    def pow(self, base: int, exp: int) -> int:
        result = self.to_mont(1)
        b = self.to_mont(base)
        while exp:
            if exp & 1:
                result = self.mul(result, b)
            b = self.mul(b, b)
            exp >>= 1
        return self.from_mont(result)
```

## Complexity

- **Per multiply:** one $k$-bit multiplication plus $O(1)$ shifts/adds — the same asymptotic cost as a schoolbook multiply, but with the `div` removed, typically a 2–4x constant-factor win in C/C++ inner loops.
- **Setup:** one modular inverse modulo $r$ (Newton's iteration, $O(\log k)$ steps) and one $r^2 \bmod n$.

## Why Python contestants rarely need it

Python's `int` is arbitrary precision and `%` is already a well-optimised C routine; `pow(a, b, n)` uses an efficient built-in. Montgomery pays off when you are hand-writing the exponentiation loop in a compiled language and the modulus is a fixed 64-bit value — Miller-Rabin and Pollard's rho on $2^{63}$-scale numbers in C++ are the classic cases. In Python, prefer the built-ins; keep this as the explanation of what those `mulmod` tricks in others' C++ templates are doing.

## Common pitfalls

- **Even modulus.** $r = 2^k$ must be coprime to $n$, so $n$ has to be odd. For even $n$, split off the power of two and CRT-recombine.
- **Forgetting to convert.** Inputs must be pushed to Montgomery form before `mul`, and the final result pulled back with one `redc`. A comparison like `x_bar == 0` still works (form of $0$ is $0$), but `x_bar == 1` does not — $1$ maps to $r \bmod n$.
- **Sign of `n_inv`.** The constant is $-n^{-1} \bmod r$, not $n^{-1}$. Off by a sign and `redc` returns garbage.
- **`k` too small.** Need $2^k > n$; for a 64-bit modulus use $k = 64$ and 128-bit intermediates in C++.

## The keystone

Montgomery form trades one division per modular multiply for a fixed conversion cost, by choosing the reduction modulus to be a power of two so that "reduce mod $r$" is a mask and "divide by $r$" is a shift. It is a constant-factor optimisation for compiled-language inner loops on a fixed modulus — invisible and unnecessary in Python, but worth recognising in every serious C++ number-theory template.
