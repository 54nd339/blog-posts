---
title: Digital Signatures - EUF-CMA, ElGamal, Schnorr, and Blind Signatures
description: What it formally means for a signature scheme to be secure (EUF-CMA), the discrete-log-based ElGamal and Schnorr schemes with their verification maths, blind signatures where the signer never sees what they sign, and how these primitives are used in electronic voting.
date: 2023-09-15
draft: false
slug: /cyber-security/digital-signatures
tags:
  - Security
  - Digital Signatures
  - Cryptography
---

A digital signature proves a message came from a specific private-key holder and has not changed since. [RSA signatures](/citadel/cyber-security/asymmetric-key) are the common case, but the field is wider: there is a precise definition of what "secure" means for a signature, alternative schemes built on the discrete logarithm problem, and a variant where the signer signs a message they cannot read. This post covers all of that, and closes with how signatures are used in electronic voting.

## What "secure" means: EUF-CMA

The standard security notion for signatures is **existential unforgeability under a chosen-message attack**. Like the [encryption security notions](/citadel/cyber-security/formal-attacks), it is a game:

1. **Setup.** The adversary is given the signer's public key.
2. **Queries.** The adversary asks a signing oracle (which holds the private key) for signatures on messages $m_1, \ldots, m_k$ of their choice and receives valid signatures.
3. **Forgery.** The adversary outputs a pair $(m^*, \sigma^*)$ where $m^*$ was *not* among the queried messages and $\sigma^*$ is a valid signature on it.

The adversary **wins** if it produces such a forgery. A scheme is **EUF-CMA secure** if any probabilistic polynomial-time adversary wins with only negligible probability.

"Existential" is a strong bar: the adversary does not have to forge a signature on a specific *meaningful* message. Any valid signature on any new message — even nonsense — counts as a break.

## ElGamal signatures

The ElGamal signature scheme (Taher Elgamal, 1985) rests on the difficulty of the discrete logarithm problem in $\mathbb{Z}_p^*$.

- **Key generation.** Pick a large prime $p$ and a generator $g$ of $\mathbb{Z}_p^*$. Choose a private key $x$ with $1 < x < p - 1$ and compute the public key $y = g^x \bmod p$. Public parameters: $(p, g, y)$.
- **Signing $M$.** Choose a random per-message secret $k$ with $1 < k < p - 1$ and $\gcd(k, p - 1) = 1$. This $k$ must be **unique per signature and kept secret**. Compute:
  $$r = g^k \bmod p, \qquad s = (H(M) - xr)\,k^{-1} \bmod (p - 1)$$
  The signature is $(r, s)$.
- **Verification.** Check $1 \le r < p$, then compute $V_1 = y^r r^s \bmod p$ and $V_2 = g^{H(M)} \bmod p$. The signature is valid if $V_1 \equiv V_2$.

**Why it works.** $y^r r^s \equiv (g^x)^r (g^k)^s \equiv g^{xr + ks} \pmod p$. From the signing equation, $ks \equiv H(M) - xr \pmod{p-1}$, so $xr + ks \equiv H(M) \pmod{p-1}$, and therefore $g^{xr + ks} \equiv g^{H(M)} \pmod p$.

The signature is probabilistic because of $k$. If $k$ is ever reused or leaked, the private key $x$ can be recovered from the two signing equations — which makes the random number generator security-critical. DSA is a standardised descendant of ElGamal.

## Schnorr signatures

The Schnorr scheme (Claus Schnorr) is also discrete-log-based, and is notable for its simplicity and for having an early provable security reduction to the DLP in the random-oracle model.

- **Key generation.** Choose a prime $p$, a prime $q$ dividing $p - 1$, and a generator $g$ of order $q$ modulo $p$. Private key $x$, public key $y = g^x \bmod p$. Parameters: $(p, q, g, y)$.
- **Signing $M$.** Choose a random $k$ with $1 \le k < q$. Compute:
  $$r = g^k \bmod p, \qquad e = H(M \,\|\, r), \qquad s = (k - xe) \bmod q$$
  The signature is $(e, s)$.
- **Verification.** Compute $r_v = g^s y^e \bmod p$ and $e_v = H(M \,\|\, r_v)$. Valid if $e_v = e$.

  The reconstruction works because $g^s y^e = g^{k - xe}\,(g^x)^e = g^{k - xe + xe} = g^k \pmod p$, so $r_v = r$ and the hash matches.

Schnorr signatures are smaller than ElGamal or DSA signatures at equivalent security. The scheme was under patent for years (now expired) and underlies many modern signatures, with good properties for threshold and multi-signatures.

## Blind signatures

A **blind signature** (David Chaum) lets a signer sign a message whose content is hidden from them. The requester disguises ("blinds") the message, the signer signs the blinded version, and the requester "unblinds" the result into a valid signature on the *original*.

With RSA (signer's public key $(N, e)$, private key $d$):

1. **Blinding.** Alice wants Bob to sign $M$. She picks a random blinding factor $r$ and computes $M' = M \cdot r^e \bmod N$, then sends $M'$.
2. **Signing.** Bob signs the blinded message: $S' = (M')^d = M^d \cdot r^{ed} = M^d \cdot r \pmod N$ (since $r^{ed} \equiv r$). He returns $S'$.
3. **Unblinding.** Alice removes the factor: $S = S' \cdot r^{-1} = M^d \pmod N$ — a valid RSA signature on $M$.

```python
import random
from math import gcd

def rsa_keys(bits=256):
    def prime(b):
        while True:
            n = random.getrandbits(b) | (1 << (b - 1)) | 1
            if all(n % p for p in (3, 5, 7, 11, 13)) and pow(2, n - 1, n) == 1:
                if all(pow(random.randrange(2, n - 1), n - 1, n) == 1 for _ in range(20)):
                    return n
    p, q = prime(bits), prime(bits)
    N, phi = p * q, (p - 1) * (q - 1)
    e = 65537
    return N, e, pow(e, -1, phi)

N, e, d = rsa_keys()
M = 42

r = random.randrange(2, N)
while gcd(r, N) != 1:
    r = random.randrange(2, N)

blinded = (M * pow(r, e, N)) % N          # Alice blinds
signed_blinded = pow(blinded, d, N)        # Bob signs, never sees M
S = (signed_blinded * pow(r, -1, N)) % N   # Alice unblinds

assert S == pow(M, d, N)                   # a valid signature on M
assert pow(S, e, N) == M                   # verifies with Bob's public key
```

**Properties:** the signer never sees the content; the unblinded signature verifies normally; and the signer cannot link a blinded message they signed to the later unblinded pair (**unlinkability**). This makes blind signatures the tool for anonymous authorised actions — anonymous **e-cash**, where a bank blindly signs a digital coin it cannot later trace to the withdrawer, and privacy-preserving voting. The e-cash and SET dual-signature applications are also covered in [transaction processing systems](/citadel/dbms/transaction-processing).

## Electronic voting

E-voting is a demanding application because its requirements pull against each other:

- **Ballot secrecy** — no one can link a voter to their vote.
- **Integrity and accuracy** — votes recorded as cast, counted correctly.
- **Verifiability** — *individual* (a voter can check their vote was recorded) and *universal* (anyone can check the outcome matches the recorded votes, without breaking secrecy).
- **Authenticity** — only eligible voters, only once.
- **Receipt-freeness** — a voter cannot prove to anyone else how they voted, which prevents vote-selling and coercion.
- **Fairness** — no partial results before polls close.

Cryptography supplies pieces of this:

- **Encryption** protects vote secrecy in transit and storage.
- **Digital signatures** authenticate voters, officials, and machines, and protect the integrity of records and tallies.
- **Blind signatures** let an authority validate a ballot without seeing the vote — anonymous but authorised voting.
- **Homomorphic encryption** allows tallying encrypted votes without decrypting individual ones.
- **Zero-knowledge proofs** prove a vote is well-formed, or a tally correct, without revealing the underlying data — see [advanced cryptography](/citadel/cyber-security/advanced).

Even with all this, building a full system that satisfies every property while remaining transparent and auditable by non-experts is unsolved in practice — software bugs, hardware tampering, insider threats, and sheer complexity remain open concerns.

## The one idea to keep

A signature scheme is secure only against a precisely defined attacker: EUF-CMA says that even after seeing signatures on messages of their choosing, an adversary cannot produce a valid signature on any new message. ElGamal and Schnorr trade RSA's factoring assumption for the discrete logarithm, with Schnorr's being smaller and cleaner. Blind signatures add one twist — signing without seeing — that turns out to be exactly what anonymous digital cash and coercion-free voting need.
