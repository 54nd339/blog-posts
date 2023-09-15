---
title: Applied Cryptanalysis - Breaking Real Implementations
description: The crypto-theory posts prove schemes secure under assumptions. This is the other side - the attacks that recover keys and plaintext when an implementation gets a detail wrong. A reused nonce, a small RSA exponent, a padding error message, a biased random number - each collapses a scheme that is sound on paper.
date: 2023-09-15
draft: false
slug: /cyber-security/applied-cryptanalysis
tags:
  - Security
  - Cryptography
  - Cryptanalysis
---

The other cryptography posts — [symmetric](/citadel/cyber-security/symmetric-key), [asymmetric](/citadel/cyber-security/asymmetric-key), [the algebra](/citadel/cyber-security/maths), [formal notions](/citadel/cyber-security/formal-attacks) — establish what security means and why the standard primitives have it. This post is the complement: the attacks that work not because AES or RSA is broken, but because an implementation reused a nonce, chose a small exponent, leaked a timing difference, or seeded a PRNG badly. In practice this is where crypto fails, and it's the substance of CTF `crypto` challenges.

The mindset: don't attack the primitive, find the implementation bug. A random 2048-bit RSA modulus with proper padding and no oracle is not going to fall to anything here — look for what's *wrong*.

## RSA weaknesses

RSA encrypts as $c = m^e \bmod n$; security assumes factoring $n$ is hard and $e$ is used correctly.

- **Small $e$, no padding.** With $e = 3$ and $m^3 < n$, then $c = m^3$ over the integers and $m = \sqrt[3]{c}$ — just take the cube root. Textbook RSA with a small message.
- **Håstad's broadcast.** The same $m$ sent to $e$ recipients with $e = 3$ and different moduli: CRT the $e$ ciphertexts to get $m^e \bmod (n_1 n_2 n_3)$, which exceeds $m^e$, so take the integer $e$-th root.
- **Wiener's attack.** If the private exponent $d < \tfrac13 n^{1/4}$ (chosen small for fast decryption), the continued-fraction expansion of $e/n$ reveals $d$.
- **Fermat factorization.** If $p$ and $q$ are close, $n = a^2 - b^2$ with $a \approx \sqrt n$; try $a = \lceil\sqrt n\rceil, a+1, \dots$ and check if $a^2 - n$ is a perfect square.
- **Common factor.** Two moduli generated with a weak RNG may share a prime: $\gcd(n_1, n_2) > 1$ factors both instantly. (Real: the 2012 internet-wide scan that factored thousands of TLS keys this way.)
- **Coppersmith.** If you know most of $m$ (a *stereotyped message* — a known prefix and a short unknown suffix) or most of $p$ (a partial key leak), lattice reduction finds the rest, as long as the unknown part is small enough (roughly $n^{1/e}$ for the message case, $n^{1/4}$ for the factor case).
- **Franklin–Reiter** — two related messages ($m_2 = m_1 + r$ for known $r$) under the same small $e$ and $n$: $\gcd$ of the two polynomials $x^e - c_1$ and $(x+r)^e - c_2$ over $\mathbb{Z}_n$ is linear, giving $m_1$.

## Padding and format oracles

The scheme is fine, but the system tells you *why* a decryption failed.

- **CBC padding oracle.** The server decrypts a ciphertext and reveals (by an error message, a status code, or a timing difference) whether the PKCS#7 padding was valid. By manipulating the previous ciphertext block and watching for "valid padding," you recover the plaintext one byte at a time — 256 queries per byte, no key needed. Also lets you *forge* a ciphertext for a chosen plaintext.
- **Bleichenbacher** — the RSA PKCS#1 v1.5 analogue: an oracle for "is the decrypted value a validly-formatted PKCS#1 block" lets you decrypt or sign a message with adaptive queries. The ROBOT attack (2017) showed it still live in many TLS stacks.
- **MAC/timing oracle** — a non-constant-time comparison of a MAC or token leaks, byte by byte, how much of your guess is correct, via response time.

## Symmetric-mode misuse

- **ECB** — identical plaintext blocks give identical ciphertext blocks (the "ECB penguin"); reveals structure, and if you control part of the plaintext prefix you can do byte-at-a-time decryption of a secret suffix.
- **CTR / GCM nonce reuse** — encrypt two messages with the same key and nonce, and the keystream is identical, so $c_1 \oplus c_2 = m_1 \oplus m_2$ — recover both plaintexts with crib-dragging. For **GCM** specifically, nonce reuse also leaks the authentication key $H$, letting you *forge* arbitrary authenticated ciphertexts.
- **CBC bit-flipping** — flipping a bit in ciphertext block $i$ flips the corresponding bit in plaintext block $i+1$ (and garbles block $i$). If block $i$ is attacker-controlled or predictable, you can flip `role=user` to `role=admin`.
- **Differential fault analysis** — induce a fault (a glitch) in a hardware AES computation and compare the faulty and correct ciphertexts; a handful of pairs recovers the last round key, then the full key (`phoenixAES`).

## Signature and nonce failures

- **ECDSA/DSA nonce reuse** — sign two messages with the same per-signature nonce $k$ and the private key falls out of two linear equations. (The 2010 PS3 break was exactly this — Sony used a constant $k$.)
- **Biased nonce** — even if $k$ is random but a few bits are predictable (from a bad RNG, or a timing leak in the modular reduction), collecting enough signatures and solving the **hidden number problem** with a lattice recovers the key. Minerva, TPM-Fail, and several real CVEs.
- **ecrecover(0)** / not validating the signature format — a malleable or degenerate signature accepted as valid.

## Weak randomness

- **LCG** — a linear congruential generator ($x_{n+1} = ax_n + c \bmod m$) is fully predictable from a couple of outputs; recover $a, c, m$ by algebra, then generate the rest.
- **Mersenne Twister (MT19937)** — Python's `random`, not cryptographic: observe 624 consecutive 32-bit outputs and you can reconstruct the entire internal state and predict all future (and past) outputs.
- **Time-seeded** — `srand(time(NULL))` gives a tiny keyspace; brute the seconds around the known generation time.
- **z3-assisted** — model the generator's update in an SMT solver, feed it the observed outputs as constraints, and let it solve for the seed/state.

## Discrete-log shortcuts

Breaking Diffie–Hellman or a discrete-log signature when the group is weak:

- **Baby-step giant-step** — $O(\sqrt n)$ time and space for a group of order $n$.
- **Pollard's rho** — $O(\sqrt n)$ time, constant space.
- **Pohlig–Hellman** — if the group order factors into small primes, solve the DLP in each small subgroup (cheap) and CRT the results. A DH group whose order is *smooth* is broken.
- **Small subgroup confinement** — a peer that doesn't validate the received public value can be forced into a tiny subgroup where the DLP is trivial.

## Lattices

The recurring heavy tool. **LLL** (and BKZ) reduce a lattice basis to short, nearly-orthogonal vectors, and many crypto problems reduce to "find a short vector":

- **Coppersmith** (above) — small roots of a polynomial mod $n$.
- **Approximate common divisor** — given $c_i = p q_i + r_i$ with small noise $r_i$ ("noisy multiples" of a secret $p$), a lattice recovers $p$.
- **Hidden number problem** — the biased-nonce ECDSA attack.
- **Knapsack / subset-sum** cryptosystems — broken by lattice reduction.
- **LWE with hints** — extra linear information about the secret shrinks the lattice enough to solve.

## Classical and hashing

- **Repeating-key XOR** — guess the key length (Hamming distance / index of coincidence), then solve each key-byte position by frequency analysis (crib `flag{` or `HTB{`).
- **Hash length extension** — for `MAC = H(secret || message)` with a Merkle–Damgård hash (MD5, SHA-1, SHA-256), an attacker who knows `MAC` and `len(message)` can compute a valid MAC for `message || padding || anything` without the secret. Use HMAC.
- **Magic hashes / type juggling** — PHP's `==` treats `"0e123..."` as `0`, so two different strings whose MD5 both start `0e` and are all digits compare equal.

## Tooling

**SageMath** (lattices, Coppersmith, elliptic curves, number theory), **z3** (constraint solving for PRNGs and logic bugs), `pycryptodome` / `sympy`, **RsaCtfTool** (throws every RSA attack at a key), `phoenixAES` (DFA), **CyberChef** (encodings and classical ciphers).

## The one idea to keep

Sound schemes fall to implementation mistakes, so applied cryptanalysis is pattern-matching the bug: a reused nonce (CTR/GCM keystream reuse, ECDSA key recovery), a small or misused RSA exponent (cube root, Håstad, Wiener, Coppersmith), an error message that distinguishes valid from invalid padding (Bleichenbacher, CBC padding oracle), a non-cryptographic RNG (LCG and MT19937 are fully predictable from outputs), or a group with smooth order (Pohlig–Hellman). Lattice reduction (LLL) is the common hammer, and the first move is always to find what's *wrong* rather than attack the primitive.
