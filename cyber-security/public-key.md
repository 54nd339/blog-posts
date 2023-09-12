---
title: Public Key Cryptography Beyond RSA - ECC, Homomorphic, and IBE
description: The public-key schemes and ideas past RSA - why encryption must be probabilistic, the Blum-Goldwasser cryptosystem, homomorphic encryption that computes on ciphertext, elliptic-curve cryptography and its small keys, and identity-based encryption where your public key is your email address.
date: 2023-09-12
draft: false
slug: /cyber-security/public-key
tags:
  - Security
  - Public-Key
  - Cryptography
---

[RSA](/citadel/cyber-security/asymmetric-key) is the public-key system people learn first, but it is one point in a wide space. This post covers the rest: why textbook RSA is not enough and encryption must be randomised, a cryptosystem whose security provably equals factoring, encryption you can compute on without decrypting, the elliptic-curve approach that does more with smaller keys, and a scheme where your public key is simply your identity.

## Why encryption must be probabilistic

A **deterministic** encryption algorithm always maps a given plaintext and key to the same ciphertext. For public-key encryption, where the public key is known to everyone, that is a fatal leak. If an attacker suspects a message is one of a few possibilities — "YES" or "NO", a bid from a small range — they encrypt each candidate with the public key and compare against the intercepted ciphertext. So no deterministic public-key scheme can be secure.

**Probabilistic encryption** injects randomness, so encrypting the same plaintext twice with the same public key gives different ciphertexts. The goal is **semantic security**: an attacker cannot tell which of two chosen messages a ciphertext encrypts, better than guessing. This is achieved either with a **randomised padding scheme** — RSA-OAEP adds random padding before the RSA operation — or with a scheme that is probabilistic by construction. The formal version of this security notion, IND-CPA, is in [formal attack models](/citadel/cyber-security/formal-attacks).

## Blum-Goldwasser

The Blum-Goldwasser cryptosystem (Blum and Goldwasser, 1984) is probabilistic by design, and its security provably equals the difficulty of factoring.

- **Key generation.** Choose large primes $p, q$ with $p \equiv q \equiv 3 \pmod 4$, and set $N = pq$ — a **Blum integer**. Public key $N$; private key $(p, q)$. Primes of this form make square roots easy to compute for whoever knows the factorisation.
- **Encryption** of bits $m_1, \ldots, m_k$. Pick a random $x_0$ with $\gcd(x_0, N) = 1$. Generate a keystream by iterating $x_{i+1} = x_i^2 \bmod N$ and taking $b_{i+1} = x_{i+1} \bmod 2$. The ciphertext is $(c_1, \ldots, c_k, x_{k+1})$ with $c_i = m_i \oplus b_i$.
- **Decryption.** Knowing $p$ and $q$, the receiver computes square roots to walk the $x_i$ sequence backward from $x_{k+1}$ to $x_1$, regenerates the $b_i$, and recovers $m_i = c_i \oplus b_i$.
- **Security.** Semantically secure if deciding quadratic residuosity modulo a Blum integer is hard — which ties back to factoring. See [quadratic residues](/citadel/cyber-security/maths).

## Homomorphic encryption

Ordinarily, to compute on data you must decrypt it. **Homomorphic encryption** lets certain computations run directly on ciphertext, so that decrypting the result gives the same answer as computing on the plaintexts. Writing $E(m)$ for the encryption of $m$:

- **Additively homomorphic** — a ciphertext operation $\oplus_{\text{enc}}$ with $E(m_1) \oplus_{\text{enc}} E(m_2) = E(m_1 + m_2)$.
- **Multiplicatively homomorphic** — likewise for multiplication.

Three tiers:

- **Partially homomorphic (PHE)** — supports one operation without limit. Unpadded RSA is multiplicatively homomorphic; the Paillier cryptosystem is additively homomorphic.
- **Somewhat homomorphic (SHE)** — a limited number of both operations.
- **Fully homomorphic (FHE)** — arbitrary additions and multiplications, so any computable function can run on encrypted data. Craig Gentry gave the first plausible construction in 2009.

Applications: processing sensitive data in the cloud without granting the provider access, privacy-preserving analytics, encrypted search. The obstacle is cost — FHE is still far slower and produces much larger ciphertexts than ordinary encryption, though it is improving. Nearly all FHE is built from [lattices](/citadel/cyber-security/advanced).

## Elliptic-curve cryptography

ECC does public-key cryptography using the algebraic structure of elliptic curves over finite fields.

- **The curve.** Over the reals, an elliptic curve is $y^2 = x^3 + ax + b$ with $4a^3 + 27b^2 \ne 0$ (so it has no singular points). In cryptography the curve is defined over a finite field, GF($p$) or GF($2^m$); the "points" are the field-element pairs $(x, y)$ satisfying the equation, plus a special **point at infinity** $\mathcal{O}$.
- **The group operation.** A geometric "addition": the line through points $P$ and $Q$ meets the curve at a third point $-R$, and $P + Q$ is defined as $R$, its reflection across the x-axis. With $\mathcal{O}$ as the identity, the points form an **abelian group**. There are closed-form algebraic formulas for the addition.
- **The hard problem.** Given a base point $G$ and a point $P = kG$ (that is, $G$ added to itself $k$ times), recovering $k$ is the **elliptic-curve discrete logarithm problem**, infeasible for large $k$.

**Why use it.** ECC reaches the same security as RSA or finite-field Diffie-Hellman with much smaller keys — a 256-bit ECC key is roughly a 3072-bit RSA key. Smaller keys mean faster computation, less power, and less bandwidth, which makes ECC the default for phones, IoT devices, and smart cards.

**Where it is used.** ECDH (key agreement, the elliptic-curve Diffie-Hellman), [ECDSA](/citadel/cyber-security/digital-signatures) (signatures), modern [TLS](/citadel/interview/https), and the cryptocurrencies Bitcoin and Ethereum.

## Identity-based encryption

Traditional public-key infrastructure makes users obtain certificates that bind their public key to their identity. **Identity-based encryption** (proposed by Shamir in 1984, first practical construction by Boneh and Franklin in 2001) removes that step: a user's public key *is* a string derived from their identity, such as an email address.

- A trusted **Private Key Generator** holds a **master secret key**.
- To encrypt to `bob@example.com`, Alice uses that string directly as the public key.
- To decrypt, Bob authenticates to the PKG, which uses its master secret and Bob's identity string to compute Bob's private key.

**Advantage:** no certificate lookup or distribution — the public key is just the recipient's known identity. **Drawbacks:** **key escrow** — the PKG can generate, and therefore knows, every user's private key, a large trust assumption and a single point of compromise; the PKG must be highly secure and always available; and revocation is more awkward than with certificates. IBE fits enterprise settings where a central trusted authority is acceptable. Compare [PKI-based identity management](/citadel/interview/identity-management).

## Hash functions, in this context

Detailed in [asymmetric key cryptography](/citadel/cyber-security/asymmetric-key), hash functions recur throughout public-key work: messages are hashed before signing (efficiency and security), hashes verify that a public key or certificate is untampered, and hashes appear inside padding schemes like RSA-OAEP to add structure and randomness before the public-key operation.

## The one idea to keep

Public-key cryptography is a collection of hard problems dressed as usable operations — factoring for RSA and Blum-Goldwasser, the elliptic-curve discrete logarithm for ECC. The variations exist because the trade-offs differ: ECC buys small keys, homomorphic encryption buys computation on ciphertext at a steep performance price, and IBE buys away certificate management at the cost of trusting a key generator with everyone's private key.
