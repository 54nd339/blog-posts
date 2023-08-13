---
title: Asymmetric Key Cryptography - RSA, Hashes, MACs, and Signatures
description: How public-key cryptography works, the number theory it rests on - primality testing, factorisation, the Chinese Remainder Theorem - a full account of RSA with a runnable implementation, cryptographic hash functions and the birthday bound, MACs and HMAC, the digital envelope, and RSA digital signatures.
date: 2023-08-13
draft: false
slug: /cyber-security/asymmetric-key
tags:
  - Security
  - Public-Key
  - Cryptography
---

Symmetric cryptography has one problem it cannot solve on its own: how do two people who have never met agree on a secret key? Asymmetric — **public-key** — cryptography answers it with a pair of mathematically linked keys. One is public and can be handed to anyone; the other is private and never leaves its owner. Anyone can encrypt a message to you with your public key, but only your private key decrypts it; only you can sign with your private key, but anyone can verify with your public key.

This post covers the maths that makes such a pair possible, RSA in full with a runnable implementation, and the primitives built alongside it — hash functions, MACs, digital envelopes, and signatures. The [public key](/citadel/cyber-security/public-key) post covers the schemes beyond RSA.

## The maths it rests on

Asymmetric cryptography needs a function that is easy one way and hard to reverse without a secret. RSA gets that from factoring.

**Primality testing.** Key generation needs large primes. Trial division is too slow, so probabilistic tests are used:

- **Fermat test** — Fermat's little theorem says that for a prime $p$ and $a$ not divisible by $p$, $a^{p-1} \equiv 1 \pmod{p}$. Pick a random $a$; if the congruence fails, $n$ is definitely composite. If it holds, $n$ is *probably* prime — but some composites, the **Carmichael numbers**, pass for every coprime $a$.
- **Miller-Rabin** — a stronger test that Carmichael numbers do not fool. Passing many rounds with random bases means prime with very high confidence.
- **AKS** is deterministic and proves primality outright, but is slower than probabilistic tests at cryptographic sizes.

**Factorisation.** Multiplying two large primes is fast; factoring a well-chosen 2048-bit product back into them is infeasible with known algorithms. RSA's security is exactly this gap: if you could factor the public modulus $N = pq$, you could derive the private key.

**Chinese Remainder Theorem.** If you know an integer's remainders modulo several pairwise-coprime numbers, you can reconstruct its remainder modulo their product, uniquely. RSA uses this to speed up private-key operations: instead of $c^d \bmod N$, compute $c^{d_p} \bmod p$ and $c^{d_q} \bmod q$ (with smaller exponents and moduli) and combine, roughly four times faster. See [number theory](/citadel/maths/number-theory).

## RSA

**Key generation:**

1. Choose two distinct large random primes $p$ and $q$.
2. Compute $N = pq$ (the modulus) and $\phi(N) = (p-1)(q-1)$ (Euler's totient of $N$).
3. Choose a public exponent $e$ with $1 < e < \phi(N)$ and $\gcd(e, \phi(N)) = 1$. A common choice is $65537 = 2^{16} + 1$.
4. Compute the private exponent $d \equiv e^{-1} \pmod{\phi(N)}$ with the extended Euclidean algorithm.

The **public key** is $(N, e)$; the **private key** is $(N, d)$, with $p$, $q$ kept secret.

**Encryption** of a message $M$ (an integer with $0 \le M < N$) with the recipient's public key: $C = M^e \bmod N$.

**Decryption** with the private key: $M = C^d \bmod N$.

**Why it works.** $C^d \equiv M^{ed} \pmod{N}$. Since $ed \equiv 1 \pmod{\phi(N)}$, we have $ed = k\phi(N) + 1$ for some $k$. When $\gcd(M, N) = 1$, Euler's theorem gives $M^{\phi(N)} \equiv 1 \pmod{N}$, so

$$M^{ed} \equiv M^{k\phi(N) + 1} \equiv (M^{\phi(N)})^k \cdot M \equiv M \pmod{N}$$

(and the CRT shows it holds even when $\gcd(M, N) \ne 1$).

### A runnable implementation

Textbook RSA — no padding, so not for production, but enough to see every step work, including the CRT speed-up:

```python
import random
from math import gcd

def is_probable_prime(n, rounds=20):
    if n < 2:
        return False
    for p in (2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37):
        if n % p == 0:
            return n == p
    d, s = n - 1, 0
    while d % 2 == 0:
        d, s = d // 2, s + 1
    for _ in range(rounds):
        a = random.randrange(2, n - 1)
        x = pow(a, d, n)
        if x in (1, n - 1):
            continue
        for _ in range(s - 1):
            x = pow(x, 2, n)
            if x == n - 1:
                break
        else:
            return False
    return True

def gen_prime(bits):
    while True:
        cand = random.getrandbits(bits) | (1 << (bits - 1)) | 1
        if is_probable_prime(cand):
            return cand

def keygen(bits=512):
    p, q = gen_prime(bits), gen_prime(bits)
    while q == p:
        q = gen_prime(bits)
    N, phi = p * q, (p - 1) * (q - 1)
    e = 65537
    assert gcd(e, phi) == 1
    d = pow(e, -1, phi)
    return (N, e), (N, d), (p, q)

def encrypt(m, pub):
    N, e = pub
    return pow(m, e, N)

def decrypt_crt(c, priv, p, q):
    N, d = priv
    dp, dq = d % (p - 1), d % (q - 1)
    qinv = pow(q, -1, p)
    m1, m2 = pow(c, dp, p), pow(c, dq, q)
    h = (qinv * (m1 - m2)) % p
    return m2 + h * q

pub, priv, (p, q) = keygen(256)
for msg in [0, 1, 2, 42, 123456789, pub[0] - 1]:
    c = encrypt(msg, pub)
    assert decrypt_crt(c, priv, p, q) == msg
    assert pow(c, priv[1], pub[0]) == msg   # plain decryption agrees with CRT
```

RSA is used both for encryption (of small data, typically a symmetric key) and for signatures.

## Cryptographic hash functions

A hash function takes an input of any length and returns a fixed-size **digest**. Cryptographic hashes differ from checksums like CRC — CRC catches accidental corruption; a cryptographic hash must resist a malicious search for collisions. Its properties:

1. **Pre-image resistance** — given a digest $h$, infeasible to find any $m$ with $H(m) = h$.
2. **Second pre-image resistance** — given $m_1$, infeasible to find a different $m_2$ with $H(m_1) = H(m_2)$.
3. **Collision resistance** — infeasible to find *any* two distinct inputs with the same digest.
4. **Deterministic** — same input, same digest.
5. **Avalanche effect** — flipping one input bit changes about half the output bits.

**The birthday bound.** In a room of 23 people there is a better-than-even chance two share a birthday, because you are comparing all *pairs*. The same combinatorics applies to hashes: finding some collision is far easier than hitting a specific target. For an $n$-bit digest, a brute-force collision search costs about $2^{n/2}$ operations, versus $2^n$ for a pre-image. So $k$ bits of security against collisions needs a $2k$-bit digest — which is why digest sizes grew.

| Function | Digest | Status |
| --- | --- | --- |
| **MD5** | 128-bit | Broken — collisions in seconds ($2^{64}$ bound); do not use for security |
| **SHA-1** | 160-bit | Insecure — collisions publicly demonstrated ($2^{80}$ bound); being retired |
| **SHA-2** | 224/256/384/512-bit | SHA-256 ($2^{128}$) and SHA-512 currently secure |
| **SHA-3 (Keccak)** | variable | Different internal design (sponge construction); secure |

Hashes are used for integrity checks, password storage, and — hashing before signing — digital signatures.

## MAC and HMAC

A **message authentication code** is a short tag that proves a message came from someone holding a **shared secret key** and was not altered. The sender combines message and key through a MAC algorithm to produce the tag; the receiver, who has the same key, recomputes it and compares. Because both parties (and only they) can generate the tag, a MAC gives authentication and integrity but **not non-repudiation** — either party could have produced it.

**HMAC** builds a MAC from a hash function and a key with two nested hashes:

$$\mathrm{HMAC}(K, m) = H\big((K' \oplus \text{opad}) \,\|\, H((K' \oplus \text{ipad}) \,\|\, m)\big)$$

where $K'$ is the key adjusted to block length, and `ipad`, `opad` are fixed padding constants. Its security reduces to the underlying hash's, and it is efficient and widely deployed.

## The digital envelope

Symmetric encryption is fast on bulk data; asymmetric encryption is slow but solves key distribution. A **digital envelope** combines them:

1. Generate a random symmetric **session key** for this message.
2. Encrypt the message with the session key.
3. Encrypt the session key with the recipient's **public key**.
4. Send both the encrypted message and the encrypted session key.
5. The recipient decrypts the session key with their **private key**, then decrypts the message.

This is how [PGP, S/MIME](/citadel/cyber-security/network-security), and [TLS](/citadel/interview/https) move data.

## RSA digital signatures

A digital signature provides authentication, integrity, and non-repudiation.

**Signing.** The sender hashes the message, $h = H(M)$, then applies the private key to the hash: $S = h^d \bmod N$. Hashing first is essential — it is faster than signing the whole message and it blocks certain forgery attacks. The sender transmits $M$ and $S$.

**Verification.** The receiver computes $h' = H(M)$ independently, applies the sender's *public* key to the signature, $h_v = S^e \bmod N$, and checks $h_v = h'$. If they match, the message came from the private-key holder and has not changed since it was signed.

The formal definition of signature security (EUF-CMA) and schemes beyond RSA — ElGamal, Schnorr, blind signatures — are in [digital signatures](/citadel/cyber-security/digital-signatures).

## The one idea to keep

Public-key cryptography turns a hard number-theory problem — factoring, for RSA — into a usable asymmetry: a public operation anyone can do and a private one only the key holder can. Everything else is built around that. Hashes compress a message to a fixed fingerprint so signatures are cheap and integrity is checkable; the digital envelope uses the slow asymmetric operation only on a short session key and lets fast symmetric encryption carry the data.
