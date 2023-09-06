---
title: Formal Security Notions - IND-CPA, IND-CCA, and Non-Malleability
description: How cryptographers define what "secure" means for an encryption scheme, using games between an adversary and a challenger - indistinguishability under chosen-plaintext and chosen-ciphertext attack, non-malleability, and the hierarchy relating them.
date: 2023-09-06
draft: false
slug: /cyber-security/formal-attacks
tags:
  - Security
  - Cryptography
---

"The attacker can't find the key" is not a definition of security. What if they could learn whether a ciphertext encrypts "attack" or "retreat" without ever recovering the key? What if they could take a ciphertext of a bid and turn it into a ciphertext of one dollar less, without knowing the bid? To rule these out, cryptographers define security precisely, as a **game** between an **adversary** and a **challenger**, and call a scheme secure when any efficient adversary wins only negligibly more often than chance.

This post covers the two families of notion — **indistinguishability** and **non-malleability** — under the standard attack models, and how they relate.

## The game framework

A security notion is a game. The challenger holds the cryptographic algorithm and keys. The adversary interacts with it — sometimes with access to an encryption or decryption **oracle** — and tries to win a specific objective. The scheme is secure under that notion if every probabilistic polynomial-time adversary's probability of winning exceeds the trivial baseline by only a negligible amount.

## Indistinguishability

The idea: an adversary should learn *nothing* about a plaintext from its ciphertext — in particular, should not be able to distinguish the encryptions of two messages of their own choosing.

### IND-CPA (chosen-plaintext attack)

The baseline for secure encryption.

1. **Learning phase.** The adversary submits plaintexts of their choice to an encryption oracle and gets the ciphertexts, seeing how messages encrypt.
2. **Challenge phase.** The adversary chooses two distinct plaintexts $M_0$ and $M_1$ of equal length and sends them.
3. The challenger flips a bit $b$, encrypts $M_b$, and returns the challenge ciphertext $C_b$.
4. **Guessing phase.** The adversary outputs a guess for $b$.

The scheme is **IND-CPA secure** if the adversary's **advantage** — $\Pr[\text{guess correct}] - \tfrac{1}{2}$ — is negligible.

A consequence: **deterministic** public-key encryption can never be IND-CPA secure, because the adversary just encrypts $M_0$ and $M_1$ themselves with the public key and compares. IND-CPA requires **probabilistic** encryption or unique nonces/IVs — [RSA-OAEP](/citadel/cyber-security/asymmetric-key) for public-key schemes, or [CBC or CTR mode](/citadel/cyber-security/symmetric-key) with random IVs for a block cipher that is a secure pseudorandom permutation.

### IND-CCA1 (non-adaptive chosen-ciphertext attack)

Stronger: the adversary also gets a **decryption oracle** — but only *before* seeing the challenge ciphertext.

1. The adversary queries the decryption oracle with ciphertexts of their choice and gets the plaintexts.
2. The challenge phase proceeds as in IND-CPA: choose $M_0, M_1$, receive $C_b$.
3. The adversary guesses $b$, with no further oracle access.

The name "lunchtime attack" captures the picture — the adversary has decryption access only for a limited window, then the challenge arrives. IND-CCA1 implies IND-CPA.

### IND-CCA2 (adaptive chosen-ciphertext attack)

The strongest standard notion, and the target for serious public-key schemes. The decryption oracle is available *before and after* the challenge.

1. **Oracle phase 1.** The adversary decrypts ciphertexts of their choice.
2. **Challenge.** Choose $M_0, M_1$; receive $C_b$.
3. **Oracle phase 2.** The adversary continues decrypting any ciphertext $c' \ne C_b$.
4. The adversary guesses $b$.

The one restriction — the adversary may not ask the oracle to decrypt $C_b$ itself — is what keeps the game meaningful. IND-CCA2 protects against an active attacker who modifies the challenge ciphertext into related ciphertexts and submits those to learn about the plaintext. Achieving it needs careful design: OAEP for RSA, authenticated encryption for symmetric ciphers.

## Non-malleability

A separate concern: given $C = E_K(M)$, can the adversary produce a *different* ciphertext $C'$ whose plaintext $M'$ bears a known, meaningful relationship to $M$? If $M$ is an auction bid, the adversary should not be able to turn $C$ into a ciphertext of $M - 1$ — even without learning $M$.

**Non-malleability (NM)** is the property that they cannot. It matters wherever an active attacker can tamper with ciphertexts in transit.

- **NM-CPA.** The adversary has an encryption oracle, is given a challenge ciphertext $C$ for a message $M$, and outputs ciphertexts $C'_1, \ldots, C'_k$ (none equal to $C$). It wins if the decryptions $M'_i$ satisfy some non-trivial relation to $M$ with significant probability.
- **NM-CCA2.** As NM-CPA, plus a decryption oracle before and after the challenge, with the restriction that $C$ itself may not be queried.

## The hierarchy

The notions form a chain of strength:

$$\text{IND-CCA2} \implies \text{IND-CCA1} \implies \text{IND-CPA} \implies \text{security against KPA and COA}$$

Each stronger notion gives the adversary a superset of the weaker one's capabilities, so security against the stronger attack implies security against the weaker.

Connecting the two families:

- For public-key encryption, **IND-CCA2 is equivalent to NM-CCA2** — protecting against distinguishing plaintexts with a decryption oracle also protects against meaningfully modifying ciphertexts.
- For symmetric encryption, a scheme providing **authenticated encryption** (confidentiality plus ciphertext integrity) generally reaches IND-CCA2.
- IND-CPA relates to NM-CPA, though the exact equivalence is more delicate.

In practice, IND-CCA2 (equivalently NM-CCA2) is the goal for deployed public-key encryption, and IND-CPA — usually via CBC or CTR with random IVs, strengthened to authenticated encryption — is the baseline for symmetric.

## The one idea to keep

Security is defined by what an adversary *cannot do* in a precisely specified game, not by the absence of a known break. Indistinguishability says they cannot tell which plaintext a ciphertext hides; non-malleability says they cannot tamper it into a related message. The attack models — chosen-plaintext, non-adaptive chosen-ciphertext, adaptive chosen-ciphertext — grade how much power the adversary is granted, and IND-CCA2 grants the most a real active attacker plausibly has.
