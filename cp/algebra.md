---
title: Algebra for Competitive Programming - The Number Theory Toolkit
description: A map of the algebra section — modular arithmetic, primes, sieves, GCD identities, and the transforms — showing which identity each contest problem is really testing and how the tools connect.
date: 2023-11-18
draft: false
slug: /cp/algebra
tags:
  - Competitive Programming
  - Algebra
  - Number Theory
---

Most "math" problems in a contest are not asking you to invent anything. They are asking whether you know one identity: that $a^{p-1} \equiv 1 \pmod p$, that $\gcd(a, b) = \gcd(b, a \bmod b)$, that the number of divisors is multiplicative. This section is a catalogue of those identities and the routines that turn each one into code.

The through-line is **modular arithmetic**. Contest answers are almost always demanded "modulo $10^9 + 7$" because the true answer overflows every integer type. So every tool here has a "mod $m$" version, and the whole section is really about computing in the ring $\mathbb{Z}/m\mathbb{Z}$ quickly.

## The spine: fast exponentiation

[Binary exponentiation](/citadel/cp/binary-exponentiation) computes $a^n$ in $O(\log n)$ multiplications by squaring. It is the single most reused routine in the section:

- $a^n \bmod m$ for large $n$ — the direct use.
- **Modular inverse** via Fermat: $a^{-1} \equiv a^{m-2} \pmod m$ when $m$ is prime ([modular inverse](/citadel/cp/modular-inverse)).
- Anything with a linear recurrence — [Fibonacci numbers](/citadel/cp/fibonacci-numbers), path counts — by exponentiating a matrix instead of a scalar.

## GCD and the Euclidean family

- [Euclidean algorithm](/citadel/cp/euclidean-gcd) — $\gcd(a, b)$ in $O(\log \min(a, b))$.
- [Extended Euclidean](/citadel/cp/extended-euclidean) — also returns $x, y$ with $ax + by = \gcd(a, b)$. This is the other route to a modular inverse, and the only one that works when $m$ is not prime.
- [Linear Diophantine equations](/citadel/cp/linear-diophantine-equations) — $ax + by = c$ has a solution iff $\gcd(a, b) \mid c$; extended Euclid builds it.
- [Linear congruence](/citadel/cp/linear-congruence-equation) and the [Chinese Remainder Theorem](/citadel/cp/chinese-remainder-theorem) — solving $ax \equiv b \pmod n$, then stitching solutions modulo coprime moduli into one.

## Primes and multiplicative functions

- [Sieve of Eratosthenes](/citadel/cp/sieve-of-eratosthenes) — every prime below $n$ in $O(n \log \log n)$; the [linear sieve](/citadel/cp/linear-sieve) also hands you smallest-prime-factor for instant factorization.
- [Primality tests](/citadel/cp/primality-tests) and [integer factorization](/citadel/cp/integer-factorization) — Miller-Rabin and Pollard's rho for numbers far past sieve range.
- [Euler's totient](/citadel/cp/eulers-totient-function) and [divisor count / sum](/citadel/cp/divisors-count-and-sum) — multiplicative functions you evaluate straight from a factorization.

## The transforms and the exotic tools

The "Miscellaneous" cluster is where the section stops being elementary: [FFT](/citadel/algorithms/FastFourierTransform) for $O(n \log n)$ polynomial multiplication, [operations on polynomials](/citadel/cp/operations-on-polynomials) built on it, [big integers](/citadel/cp/arbitrary-precision-arithmetic) when even $10^{18}$ is too small, and [continued fractions](/citadel/cp/continued-fractions) for best rational approximations.

## Recognising which tool a problem wants

- "modulo $10^9 + 7$", "count the ways" → you will need [binary exponentiation](/citadel/cp/binary-exponentiation) and probably [binomial coefficients](/citadel/cp/binomial-coefficients).
- "divide modulo a prime" → [modular inverse](/citadel/cp/modular-inverse).
- "smallest $x$ with $x \equiv r_i \pmod{m_i}$" → [CRT](/citadel/cp/chinese-remainder-theorem).
- "how many integers up to $n$ are coprime to $n$" → [totient](/citadel/cp/eulers-totient-function).
- "multiply two degree-$10^5$ polynomials", "count pairs with a given sum" → [FFT](/citadel/algorithms/FastFourierTransform).

## Where this goes next

The [combinatorics](/citadel/cp/combinatorics) section is built almost entirely on this one: binomial coefficients modulo a prime need the inverse, Catalan numbers need the inverse, inclusion-exclusion needs fast modular sums. Get the exponentiation-and-inverse pair solid here and that section becomes bookkeeping.
