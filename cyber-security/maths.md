---
title: The Algebra Behind Cryptography - Groups, Fields, and Quadratic Residues
description: Why modern cryptography is built on finite algebraic structures - a review of the number theory it needs, the hierarchy of groups, rings, and fields, the finite fields GF(p) and GF(2^n) that ciphers compute in, and quadratic residues with the Legendre and Jacobi symbols.
date: 2023-09-05
draft: false
slug: /cyber-security/maths
tags:
  - Security
  - Cryptography
---

Ask why RSA is secure and the honest answer is a fact about integers: multiplying two 300-digit primes is fast, and recovering them from the product is not. Ask why AES mixes bytes the way it does and the answer is that a byte is an element of a field with 256 elements, and the mixing step is multiplication in that field. Modern cryptography runs on a small set of finite algebraic structures, and this post is a tour of them: the number theory it assumes, the hierarchy of groups, rings, and fields, the specific finite fields ciphers compute in, and the theory of quadratic residues that underpins a family of cryptosystems.

## What it builds on

Two ideas from elementary number theory carry through everything here. Both are developed in [number theory](/citadel/maths/number-theory); this is the working summary.

**Primes.** A prime is an integer greater than 1 with no divisors but 1 and itself. Their cryptographic value comes from a one-way asymmetry: given two large primes $p$ and $q$, computing $N = pq$ is trivial, while factoring a well-chosen $N$ of 2048 bits or more back into $p$ and $q$ is infeasible with known algorithms.

**Modular arithmetic.** Working modulo $n$ means working with remainders on division by $n$. We write $a \equiv b \pmod{n}$ when $n$ divides $a - b$ — equivalently, $a$ and $b$ leave the same remainder — so $17 \equiv 2 \pmod{5}$. Two operations do the heavy lifting:

- **Modular exponentiation** $a^k \bmod n$, computed efficiently by repeated squaring rather than $k$ multiplications.
- The **modular multiplicative inverse** $a^{-1}$, the value with $a \cdot a^{-1} \equiv 1 \pmod{n}$. It exists exactly when $\gcd(a, n) = 1$, and the extended Euclidean algorithm finds it.

Modular arithmetic gives a *finite* number system, which is what makes exhaustive structure possible and one-way functions plausible.

## Groups: one operation

Strip a number system down to a set and one operation obeying a few rules, and you have a **group**. A group is a set $G$ with a binary operation $\cdot$ satisfying four axioms:

1. **Closure** — for all $a, b \in G$, the result $a \cdot b$ is in $G$.
2. **Associativity** — $(a \cdot b) \cdot c = a \cdot (b \cdot c)$.
3. **Identity** — there is an $e \in G$ with $e \cdot a = a \cdot e = a$ for every $a$.
4. **Inverse** — every $a$ has a $b$ with $a \cdot b = b \cdot a = e$.

If the operation also commutes, $a \cdot b = b \cdot a$, the group is **abelian**.

Familiar examples:

- The integers $\mathbb{Z}$ under addition — identity $0$, inverse of $a$ is $-a$. Abelian.
- The non-zero rationals $\mathbb{Q}^*$ under multiplication — identity $1$, inverse of $a$ is $1/a$. Abelian.
- The integers modulo $n$, $\mathbb{Z}_n = \{0, 1, \ldots, n-1\}$, under addition mod $n$.
- The integers in $\{1, \ldots, n-1\}$ that are coprime to $n$, written $\mathbb{Z}_n^*$, under multiplication mod $n$. This one matters most in cryptography — it is the group RSA and many protocols work inside.

The structure that cryptography leans on hardest is a **cyclic group**: one where a single element $g$, the generator, produces every element as a power $g^k$. Diffie-Hellman key exchange and elliptic-curve cryptography both operate in cyclic groups, and their security rests on the **discrete logarithm problem** — given $g$ and $g^k$, recover $k$ — which is hard in a suitably chosen cyclic group. The general theory of groups, subgroups, cosets, and Lagrange's theorem is in [abstract algebra](/citadel/maths/abstract-algebra).

## Rings: two operations

Add a second operation and you get a **ring**. A ring is a set $R$ with addition and multiplication such that:

1. $R$ is an abelian group under addition (closed, associative, commutative, with identity $0$ and additive inverses).
2. $R$ is closed and associative under multiplication.
3. Multiplication distributes over addition: $a(b + c) = ab + ac$ and $(b + c)a = ba + ca$.

If multiplication commutes, the ring is **commutative**; if there is a multiplicative identity $1$, it is a **ring with unity**. The integers $\mathbb{Z}$ under ordinary addition and multiplication are a commutative ring with unity — and note that they are *not* a field, because $2$ has no integer multiplicative inverse. Polynomial rings appear in several cryptographic constructions, and the ring structure is what lets us build finite fields.

## Fields: where division works

A **field** is a commutative ring with unity in which every non-zero element has a multiplicative inverse. Equivalently:

1. $F$ is an abelian group under addition.
2. $F \setminus \{0\}$ is an abelian group under multiplication.
3. Multiplication distributes over addition.

In a field you can add, subtract, multiply, and divide (by non-zero elements), and the operations behave as they do for rationals or reals. $\mathbb{Q}$, $\mathbb{R}$, and $\mathbb{C}$ are fields.

Cryptography needs **finite fields**, also called **Galois fields**.

**GF($p$), $p$ prime.** The set $\{0, 1, \ldots, p-1\}$ with addition and multiplication mod $p$. Every non-zero element $a$ has an inverse because $p$ is prime, so $\gcd(a, p) = 1$. This is the field discrete-log protocols compute in.

**GF($p^n$), extension fields.** Here the elements are not integers but polynomials. The case cryptography uses constantly is **GF($2^n$)**, the binary fields: elements are polynomials of degree less than $n$ with coefficients in GF(2) — that is, coefficients that are 0 or 1. Addition adds coefficients mod 2, which is a bitwise XOR. Multiplication multiplies the polynomials and then reduces modulo a fixed **irreducible polynomial** of degree $n$ (one that does not factor over GF(2)), keeping the result in the field. AES computes in **GF($2^8$)**, where each element is one byte; its S-box is a multiplicative inverse in this field, and its `MixColumns` step is polynomial multiplication in it. The worked byte-level arithmetic is in [symmetric key cryptography](/citadel/cyber-security/symmetric-key).

## Quadratic residues

Within modular arithmetic, a special question arises: which numbers are perfect squares mod $n$?

**Definition.** For a positive integer $n$, an integer $a$ with $\gcd(a, n) = 1$ is a **quadratic residue modulo $n$** if the congruence $x^2 \equiv a \pmod{n}$ has a solution. If it has none, $a$ is a **quadratic non-residue**.

The clean case is an odd prime $p$. There, among $\{1, 2, \ldots, p-1\}$, exactly $(p-1)/2$ values are quadratic residues and exactly $(p-1)/2$ are non-residues — because squaring is two-to-one on the non-zero elements ($x$ and $-x$ square to the same thing).

**Worked example, $p = 7$.** Square each non-zero element:

$$1^2 \equiv 1,\quad 2^2 \equiv 4,\quad 3^2 \equiv 2,\quad 4^2 \equiv 2,\quad 5^2 \equiv 4,\quad 6^2 \equiv 1 \pmod{7}$$

The values that appear are $\{1, 2, 4\}$ — those are the quadratic residues mod 7. The other three, $\{3, 5, 6\}$, are non-residues. As predicted, there are $(7-1)/2 = 3$ of each.

**Why it matters.** Deciding whether a number is a quadratic residue mod a prime is easy. But *finding a square root* modulo a composite $N = pq$ with $p, q$ unknown is computationally equivalent to factoring $N$. That hardness is the basis of the **Goldwasser-Micali** cryptosystem (a probabilistic public-key scheme whose security is the difficulty of the quadratic residuosity problem mod $N$) and the **Rabin** cryptosystem (security equivalent to factoring). The [public key](/citadel/cyber-security/public-key) post covers the Blum-Goldwasser scheme that builds on this.

## The Legendre symbol

To reason about quadratic residuosity compactly, define the **Legendre symbol** $\left(\dfrac{a}{p}\right)$ for an integer $a$ and an odd prime $p$:

$$\left(\frac{a}{p}\right) = \begin{cases} 0 & \text{if } p \mid a \\ 1 & \text{if } a \text{ is a quadratic residue mod } p \\ -1 & \text{if } a \text{ is a quadratic non-residue mod } p \end{cases}$$

**Euler's criterion** turns this into a computation:

$$\left(\frac{a}{p}\right) \equiv a^{(p-1)/2} \pmod{p}$$

So raising $a$ to the power $(p-1)/2$ and checking whether the result is $1$ or $-1$ tells you directly whether $a$ is a residue. The reason this works: $a^{p-1} \equiv 1 \pmod{p}$ by Fermat's little theorem, so $a^{(p-1)/2}$ is a square root of $1$, hence $\pm 1$; and it is $+1$ exactly when $a$ is itself a square.

The symbol's properties:

1. If $a \equiv b \pmod{p}$, then $\left(\frac{a}{p}\right) = \left(\frac{b}{p}\right)$.
2. $\left(\frac{a^2}{p}\right) = 1$ when $p \nmid a$ — squares are residues.
3. $\left(\frac{ab}{p}\right) = \left(\frac{a}{p}\right)\left(\frac{b}{p}\right)$ — completely multiplicative.
4. $\left(\frac{1}{p}\right) = 1$.
5. $\left(\frac{-1}{p}\right) = (-1)^{(p-1)/2}$, so $-1$ is a residue when $p \equiv 1 \pmod{4}$ and a non-residue when $p \equiv 3 \pmod{4}$.
6. $\left(\frac{2}{p}\right) = (-1)^{(p^2-1)/8}$, so $2$ is a residue when $p \equiv 1$ or $7 \pmod{8}$.
7. **Law of quadratic reciprocity.** For distinct odd primes $p, q$:
   $$\left(\frac{p}{q}\right)\left(\frac{q}{p}\right) = (-1)^{\frac{p-1}{2} \cdot \frac{q-1}{2}}$$

Reciprocity is the surprising one: whether $p$ is a residue mod $q$ is tied to whether $q$ is a residue mod $p$, and the two differ only when both primes are $\equiv 3 \pmod 4$. It lets you compute a Legendre symbol by flipping and reducing, the way the Euclidean algorithm computes a gcd.

## The Jacobi symbol

The Legendre symbol needs the bottom to be an odd prime. The **Jacobi symbol** $\left(\dfrac{a}{n}\right)$ generalises it to any odd $n > 1$. If $n = p_1^{k_1} \cdots p_m^{k_m}$, then

$$\left(\frac{a}{n}\right) = \left(\frac{a}{p_1}\right)^{k_1} \cdots \left(\frac{a}{p_m}\right)^{k_m}$$

with each factor a Legendre symbol. It inherits congruence-invariance, multiplicativity in the top argument, and a matching law of quadratic reciprocity.

**One distinction to keep straight.** If $\left(\frac{a}{n}\right) = -1$, then $a$ is definitely a quadratic non-residue mod $n$ — an odd number of the prime factors have $a$ as a non-residue. But if $n$ is composite and $\left(\frac{a}{n}\right) = 1$, that does *not* mean $a$ is a residue: it could be a non-residue mod every prime factor, and the two $-1$s multiply to $+1$. For $a$ to be a genuine residue mod composite $n$, it must be a residue mod each prime factor.

The Jacobi symbol's practical value is that it can be computed efficiently *without factoring $n$*, using its properties and reciprocity. That makes it the workhorse of the **Solovay-Strassen primality test**: a number $n$ is probably prime if $\left(\frac{a}{n}\right) \equiv a^{(n-1)/2} \pmod{n}$ holds for many random $a$ — an identity that always holds for primes and fails for at least half the $a$ when $n$ is composite.

## Where this shows up

- **Groups and cyclic groups** — Diffie-Hellman and elliptic-curve cryptography live in cyclic groups; their security is the discrete logarithm problem.
- **GF($p$)** — the field for discrete-log protocols and much of public-key cryptography.
- **GF($2^n$)** — the field AES computes in; its S-box inverse and column mixing are field operations, giving the cipher its non-linearity and diffusion.
- **Quadratic residues** — the hardness of square roots mod a composite drives Goldwasser-Micali, Rabin, and Blum-Goldwasser encryption.
- **Legendre and Jacobi symbols** — efficient residuosity tests, and the core of the Solovay-Strassen primality test used when generating keys.

## The one idea to keep

Cryptography does not use "advanced maths" so much as it uses *finite* maths. Restrict arithmetic to a group, a ring, or a field with finitely many elements, and two useful things happen: every operation is well-defined and reversible with the key, and certain natural questions — what is the discrete logarithm, what is the square root, what are the prime factors — become computationally hard. The structures in this post are the settings where that trade is favourable.
