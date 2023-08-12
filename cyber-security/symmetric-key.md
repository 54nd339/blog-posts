---
title: Symmetric Key Cryptography - The Maths and the Ciphers
description: The discrete maths symmetric ciphers run on - modular arithmetic, linear congruences, and GF(2^n) field arithmetic - followed by the structures and internals of DES, Blowfish, and AES, and the Diffie-Hellman exchange that gives two parties a shared key over an open channel.
date: 2023-08-12
draft: false
slug: /cyber-security/symmetric-key
tags:
  - Security
  - Cryptography
---

Symmetric encryption uses one secret key for both encryption and decryption. The [overview](/citadel/cyber-security/cyber-security) covered the difference between stream and block ciphers and the modes of operation; this post goes into the internals. First the maths these ciphers compute in — modular arithmetic, linear congruences, and finite-field arithmetic — then the workings of DES, Blowfish, and AES, and finally the Diffie-Hellman exchange that lets two parties agree on a key without ever sending it.

## Modular arithmetic

Two integers $a$ and $b$ are **congruent modulo $n$**, written $a \equiv b \pmod{n}$, when $n$ divides $a - b$. The set $\{0, 1, \ldots, n-1\}$ is a **complete residue system**: every integer is congruent to exactly one of them. Congruence mod $n$ is an equivalence relation (reflexive, symmetric, transitive), and it respects arithmetic — if $a \equiv b$ and $c \equiv d \pmod n$, then $a + c \equiv b + d$ and $ac \equiv bd \pmod n$.

**Modular exponentiation.** Computing $a^k \bmod n$ by repeated squaring is far cheaper than $k$ multiplications. For $3^{13} \bmod 10$: square up, $3^1 \equiv 3$, $3^2 \equiv 9$, $3^4 \equiv 81 \equiv 1$, $3^8 \equiv 1$; then $13 = 8 + 4 + 1$, so $3^{13} \equiv 1 \cdot 1 \cdot 3 \equiv 3 \pmod{10}$.

**Modular multiplicative inverse.** The inverse $a^{-1}$ satisfies $a \cdot a^{-1} \equiv 1 \pmod{n}$, and it exists exactly when $\gcd(a, n) = 1$. The **extended Euclidean algorithm** finds integers $x, y$ with $ax + ny = \gcd(a, n)$; when that gcd is $1$, reducing mod $n$ gives $ax \equiv 1 \pmod{n}$, so $x$ is the inverse. See [number theory](/citadel/maths/number-theory).

## Linear congruences

A **linear congruence** $ax \equiv b \pmod{n}$ asks for integer $x$. Let $d = \gcd(a, n)$:

- If $d \nmid b$, there are no solutions.
- If $d \mid b$, there are exactly $d$ incongruent solutions modulo $n$.
- If $\gcd(a, n) = 1$, there is a unique solution, $x \equiv a^{-1} b \pmod{n}$.

These turn up in cryptanalysis and inside key-schedule steps.

## GF($2^n$) arithmetic

Block ciphers, AES especially, compute in the binary field **GF($2^n$)**. An element is a polynomial of degree less than $n$ with coefficients in GF(2) — 0 or 1 — equivalently an $n$-bit string. The theory is in [the algebra post](/citadel/cyber-security/maths); here is the arithmetic.

**Addition** adds coefficients mod 2, which is bitwise XOR. In GF($2^3$):

$$(x^2 + x) + (x + 1) = x^2 + (1+1)x + 1 = x^2 + 1$$

or in bits, `110` XOR `011` = `101`. Subtraction is the same as addition.

**Multiplication** multiplies the polynomials, then reduces modulo a fixed **irreducible polynomial** of degree $n$ to stay in the field. In GF($2^3$) with $m(x) = x^3 + x + 1$, multiply $(x^2 + 1)(x + 1) = x^3 + x^2 + x + 1$. Now $x^3 \equiv x + 1 \pmod{m(x)}$, so

$$x^3 + x^2 + x + 1 \equiv (x + 1) + x^2 + x + 1 \equiv x^2 \pmod{m(x)}$$

In bits, `101` · `011` = `100`. Every non-zero element has a multiplicative inverse, found with the extended Euclidean algorithm for polynomials.

AES works in **GF($2^8$)** — each element is a byte — with $m(x) = x^8 + x^4 + x^3 + x + 1$. Multiplication there is a short loop: repeatedly multiply by $x$ (a left shift, then XOR $m(x)$'s low byte `0x1b` if the shifted-out bit was 1) and accumulate:

```python
def gf_mul(a, b):
    """Multiply two bytes in AES's GF(2^8), modulus x^8+x^4+x^3+x+1."""
    p = 0
    for _ in range(8):
        if b & 1:
            p ^= a
        carry = a & 0x80
        a = (a << 1) & 0xFF
        if carry:
            a ^= 0x1B
        b >>= 1
    return p

# known AES values: MixColumns uses multiplication by 2 and 3
assert gf_mul(0x57, 0x02) == 0xAE
assert gf_mul(0x57, 0x03) == 0xF9
assert gf_mul(0xD4, 0x02) == 0xB3
# multiplication is commutative and 1 is the identity
for a in range(256):
    assert gf_mul(a, 1) == a
    for b in range(0, 256, 17):
        assert gf_mul(a, b) == gf_mul(b, a)
```

This is exactly the operation AES's `MixColumns` performs on each byte of a column.

## Block cipher structure

Modern block ciphers resist cryptanalysis through Shannon's two principles, applied over many rounds:

- **Confusion** — make the ciphertext-to-key relationship complex and non-linear. **S-boxes** (substitution boxes) do this.
- **Diffusion** — spread the influence of each plaintext bit over many ciphertext bits, so flipping one plaintext bit changes about half the ciphertext. Permutation layers do this.

Two iterative structures dominate:

**Feistel network.** Split the block into halves $L_0, R_0$. Each round:

$$L_i = R_{i-1}, \qquad R_i = L_{i-1} \oplus F(R_{i-1}, K_i)$$

with round function $F$ and subkey $K_i$. After the last round the halves are swapped once. The advantage: decryption is the *same* process with the subkeys applied in reverse order, so one implementation does both. DES uses it.

**Substitution-permutation network (SPN).** The whole block is processed each round: mix in a round key, apply S-boxes for confusion, apply a permutation or linear mixing for diffusion. AES uses it.

## Stream ciphers

A stream cipher generates a pseudorandom **keystream** $s_1, s_2, \ldots$ from the key and a nonce, and XORs it with the plaintext: $c_i = p_i \oplus s_i$, with decryption the same XOR. The keystream must be unpredictable without the key, have a long period, and pass statistical randomness tests — and it must **never** be reused with the same key, because $c_1 \oplus c_2 = p_1 \oplus p_2$ leaks the plaintexts. **Linear feedback shift registers** produce long-period sequences cheaply in hardware and are combined non-linearly to strengthen them; many modern designs are built from addition, rotation, and XOR (ARX).

## Diffie-Hellman key exchange

Diffie-Hellman is not an encryption algorithm — it solves the problem of establishing a shared secret over an insecure channel, so a symmetric cipher can then be used.

Public parameters: a large prime $p$ and a generator $g$ (a primitive root mod $p$).

1. Alice picks a secret $a$, computes $A = g^a \bmod p$, sends $A$.
2. Bob picks a secret $b$, computes $B = g^b \bmod p$, sends $B$.
3. Alice computes $S = B^a = g^{ba} \bmod p$.
4. Bob computes $S = A^b = g^{ab} \bmod p$.

Both hold $S = g^{ab} \bmod p$, which they use as a key. An eavesdropper sees $p, g, A, B$ but recovering $a$ from $A = g^a \bmod p$ is the **discrete logarithm problem**, infeasible for a well-chosen large $p$.

## DES

- **Structure** — 64-bit block, 16-round Feistel network.
- **Key** — 64 bits, but only 56 are effective (8 are parity). The key schedule produces a 48-bit subkey per round via permutations and left shifts.
- **Encryption** — an initial permutation, split into $L_0, R_0$ (32 bits each), 16 Feistel rounds, a final swap of $L_{16}$ and $R_{16}$, then the inverse of the initial permutation. The round function $F(R_{i-1}, K_i)$ expands $R_{i-1}$ from 32 to 48 bits, XORs the round key, passes the result through eight S-boxes (6 bits in, 4 bits out each — the non-linear core), and applies a bit permutation.
- **Weaknesses** — the 56-bit key was brute-forced in the late 1990s. It has reasonable resistance to differential and linear cryptanalysis for its 16 rounds, but the key size is fatal.
- **3DES** — apply DES three times, $C = E_{K_3}(D_{K_2}(E_{K_1}(P)))$, for a longer effective key. Much slower, but a stopgap that outlived single DES.

## Blowfish

- **Structure** — 64-bit block, 16-round Feistel.
- **Key** — variable, 32 to 448 bits.
- **Key expansion** — a slow process derives eighteen 32-bit subkeys ($P_1, \ldots, P_{18}$) and four 256-entry 32-bit S-boxes. The P-array and S-boxes start as digits of $\pi$; the P-array is XORed with the key; then an all-zero block is encrypted with the current state, its output replaces $P_1, P_2$, that is re-encrypted to replace $P_3, P_4$, and so on through the P-array and all four S-boxes. The result is **key-dependent S-boxes**.
- **Round function** — the 32-bit half is split into bytes $a, b, c, d$; $F = ((S_1[a] + S_2[b] \bmod 2^{32}) \oplus S_3[c]) + S_4[d] \bmod 2^{32}$.
- **Security** — no effective cryptanalysis of 16-round Blowfish is known. The 64-bit block is its modern weakness: encrypting large volumes under one key risks birthday-bound block collisions.

## AES (Rijndael)

- **Structure** — an SPN, not Feistel. 128-bit blocks. AES-128 uses a 128-bit key and 10 rounds; AES-192, 12 rounds; AES-256, 14 rounds.
- **The state** — the block is a $4 \times 4$ matrix of bytes.
- **Each round** (the final round omits `MixColumns`):
  1. **`SubBytes`** — replace each byte via a fixed S-box, built by taking the multiplicative inverse in GF($2^8$) (with $00 \mapsto 00$) and then an affine transform over GF(2): $b'_i = b_i \oplus b_{(i+4)\bmod 8} \oplus b_{(i+5)\bmod 8} \oplus b_{(i+6)\bmod 8} \oplus b_{(i+7)\bmod 8} \oplus c_i$, where $c$ is the byte `0x63`. This provides non-linearity.
  2. **`ShiftRows`** — cyclically left-shift row 1 by one byte, row 2 by two, row 3 by three; row 0 unchanged. Diffusion across a row.
  3. **`MixColumns`** — treat each column as a polynomial over GF($2^8$) and multiply it modulo $x^4 + 1$ by the fixed $a(x) = \{03\}x^3 + \{01\}x^2 + \{01\}x + \{02\}$. Every output byte of a column depends on all four input bytes. Omitted in the final round.
  4. **`AddRoundKey`** — XOR the 128-bit round key into the state.
- **Key schedule** — derives the round keys with word rotations, S-box substitutions, and XORs with round constants (Rcon).
- **Security** — the global standard, resistant to linear and differential cryptanalysis, with well-understood component maths.

## The one idea to keep

A symmetric cipher is a sequence of reversible operations in a finite algebraic setting — modular arithmetic and GF($2^8$) for AES — arranged so that confusion and diffusion compound over many rounds. Feistel and SPN are two ways to structure those rounds. And Diffie-Hellman is the piece that makes symmetric encryption usable between strangers: it turns the discrete logarithm problem into a shared secret neither party ever transmitted.
