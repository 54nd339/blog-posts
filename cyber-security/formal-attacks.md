---
title: The Gauntlet - Formal Security Notions and Cryptographic Attack Models
description: Understanding cryptographic security beyond 'unbreakable' - Exploring IND-CPA, IND-CCA1/2, Non-Malleability (NM-CPA/CCA2), and how these formal attack models define modern cipher strength.
date: 2023-06-12
draft: false
slug: /pensieve/cyber-security/formal-attacks
tags:
  - Cyber Security
  - CS Basics
---

Welcome back, cryptographic connoisseurs! We've journeyed through mathematical foundations and looked at various ciphers. But when cryptographers design a new encryption scheme, how do they argue it's "secure"? It's not enough to say "it looks complicated." They use **formal notions of security** and define specific **attack models** to rigorously test and prove (or at least provide strong evidence for) a cryptosystem's strength.

Today, we're stepping into this more theoretical arena to understand concepts like Indistinguishability and Non-Malleability under different attack scenarios. These ideas are crucial for classifying how robust a cipher truly is.

---
## What Does "Secure" Really Mean in Cryptography?

Simply being unable to find the secret key isn't the only measure of a cipher's security. What if an attacker could learn *something* about the plaintext without knowing the key? Or what if they could cleverly change a ciphertext so that it decrypts to a related, still useful, message?

To address these nuances, cryptographers define security in terms of "games" played between an **adversary** (the attacker) and a **challenger** (who has access to the cryptographic algorithm and keys). The adversary's goal is to "win" the game, and the cryptosystem is considered secure if any computationally bounded adversary has only a negligible chance of winning.

Let's explore two key security properties: **Indistinguishability** and **Non-Malleability**.

---
## Indistinguishability (IND): Can the Adversary Tell the Difference?

The core idea of indistinguishability is that an adversary should not be able to learn any partial information about a plaintext by looking at its ciphertext. More formally, they shouldn't be able to distinguish between the encryptions of two different messages of their choosing, or between an encryption of a chosen message and an encryption of random data, with a probability significantly better than random guessing.

### IND-CPA: Indistinguishability under Chosen Plaintext Attack

This is a widely accepted baseline for secure encryption.

* **The "Game":**

  1. **Learning Phase:** The adversary can choose any number of plaintexts ($m_1, m_2, \ldots, m_k$) and submit them to an "encryption oracle" (which knows the secret key $K$). The oracle returns the corresponding ciphertexts ($c_1=E_K(m_1), c_2=E_K(m_2), \ldots$). This allows the adversary to see how various messages encrypt.
  2. **Challenge Phase:** The adversary chooses two distinct plaintexts, $M_0$ and $M_1$ (of the same length). They send these to the challenger.
  3. The challenger randomly picks a bit $b \in \{0, 1\}$, encrypts $M_b$ to get the challenge ciphertext $C_b = E_K(M_b)$, and sends $C_b$ back to the adversary.
  4. **Guessing Phase:** The adversary must now guess the value of $b$ (i.e., determine if $C_b$ is the encryption of $M_0$ or $M_1$).
* **Security Definition:** An encryption scheme is IND-CPA secure if any probabilistic polynomial-time (PPT) adversary wins this game (guesses $b$ correctly) with a probability that is at most negligibly greater than $1/2$ (random guessing). The "advantage" of the adversary is $\text{Pr}[\text{Adversary wins}] - 1/2$, and this advantage must be negligible.
* **Implications:**

  * Deterministic public-key encryption schemes (where the same plaintext always encrypts to the same ciphertext with a given public key) can **never** be IND-CPA secure. An adversary can simply encrypt $M_0$ and $M_1$ with the public key and compare with $C_b$.
  * To achieve IND-CPA, encryption must be **probabilistic** or use unique nonces/IVs. For public-key schemes, this often involves random padding (e.g., RSA-OAEP). For symmetric block ciphers, modes like CBC or CTR with random IVs provide IND-CPA security if the underlying block cipher is a secure pseudorandom permutation (PRP).

### IND-CCA1: Indistinguishability under Non-Adaptive Chosen Ciphertext Attack (Lunchtime Attack)

This notion strengthens IND-CPA by giving the adversary access to a decryption oracle *before* they see the challenge ciphertext.

* **The "Game":**

  1. **Pre-Challenge Decryption Oracle Access:** The adversary can query a decryption oracle with any ciphertexts $c_i$ of their choice (except, implicitly, the challenge ciphertext they will later receive) and obtain the corresponding plaintexts $m_i = D_K(c_i)$.
  2. **Challenge Phase:** The adversary chooses two distinct plaintexts, $M_0$ and $M_1$, and sends them to the challenger.
  3. The challenger randomly picks $b \in \{0, 1\}$, encrypts $M_b$ to get $C_b = E_K(M_b)$, and sends $C_b$ to the adversary.
  4. **Guessing Phase:** The adversary outputs a guess $b'$. **Crucially, the adversary cannot query the decryption oracle with $C_b$ or any ciphertext derived from $C_b$ after this point.** This is why it's sometimes called a "lunchtime attack" – the oracle access is only available before the main challenge.
* **Security Definition:** Similar to IND-CPA, the adversary's advantage in guessing $b$ must be negligible. IND-CCA1 is a stronger security notion than IND-CPA.

### IND-CCA2: Indistinguishability under Adaptive Chosen Ciphertext Attack

This is often considered the "gold standard" for security of encryption schemes, especially public-key ones. It allows the adversary to make decryption queries *even after* receiving the challenge ciphertext.

* **The "Game":**

  1. **Oracle Access Phase 1:** The adversary can query a decryption oracle with any ciphertexts $c_i$ and obtain plaintexts $m_i$.
  2. **Challenge Phase:** The adversary chooses two distinct plaintexts, $M_0$ and $M_1$, and sends them to the challenger.
  3. The challenger randomly picks $b \in \{0, 1\}$, encrypts $M_b$ to get $C_b = E_K(M_b)$, and sends $C_b$ to the adversary.
  4. **Oracle Access Phase 2:** The adversary can *continue* to query the decryption oracle with any ciphertext $c'_j$, **with the sole restriction that $c'_j \neq C_b$**.
  5. **Guessing Phase:** The adversary outputs a guess $b'$.
* **Security Definition:** The adversary's advantage in guessing $b$ must be negligible.
* **Significance:** IND-CCA2 security protects against active attacks where an adversary might try to modify the challenge ciphertext in various ways and submit these modifications to the decryption oracle to gain information about the original plaintext. Achieving IND-CCA2 often requires careful design, such as using secure padding schemes like OAEP for RSA, or authenticated encryption modes for symmetric ciphers.

---
## Non-Malleability (NM): Can the Adversary Tamper Meaningfully?

Non-malleability is a security property ensuring that an adversary, given a ciphertext $C = E_K(M)$, cannot generate a *different* ciphertext $C'$ such that the plaintext $M'$ (corresponding to $C'$) has a known, meaningful relationship to the original plaintext $M$.
For example, if $M$ is a bid in an auction, an attacker shouldn't be able to change $C$ to $C'$ such that $M'$ is $M-1$ (a slightly lower bid), even if the attacker doesn't learn $M$ itself.

* **Core Idea:** The inability of an adversary to transform a ciphertext into another ciphertext whose corresponding plaintext is related to the original plaintext in a known way.
* This is crucial for protocols where ciphertexts might be manipulated by an active attacker.

Similar to indistinguishability, non-malleability is defined under chosen plaintext and chosen ciphertext attacks:

### NM-CPA: Non-Malleability under Chosen Plaintext Attack

* **The "Game" (Simplified):**
  1. The adversary has access to an encryption oracle.
  2. The adversary chooses a message $M$. The challenger encrypts it to get $C = E_K(M)$.
  3. The adversary outputs a vector of ciphertexts $\vec{C}' = (C'_1, \ldots, C'_k)$, where no $C'_i$ is equal to $C$.
  4. Let $\vec{M}' = (M'_1, \ldots, M'_k)$ be the decryptions of $\vec{C}'$. The adversary wins if they can establish a non-trivial relationship $R(M, \vec{M}')$ that holds with significant probability.

### NM-CCA2: Non-Malleability under Adaptive Chosen Ciphertext Attack

* **The "Game" (Simplified):** Similar to NM-CPA, but the adversary also has access to a decryption oracle (before and after seeing the challenge $C$), with the restriction that they cannot query $C$ itself to the decryption oracle.

---
## Inter-relations Among the Attack Models and Security Notions

These formal notions of security are not isolated; they form a hierarchy of strength:

* **IND-CCA2 implies IND-CCA1:** An adversary with CCA1 capabilities is a restricted version of an adversary with CCA2 capabilities.
* **IND-CCA1 implies IND-CPA:** An adversary with CPA capabilities is a restricted version of an adversary with CCA1 capabilities.
* **IND-CPA implies security against simpler attacks:** If a scheme is IND-CPA secure, it is also secure against known-plaintext attacks (KPA) and ciphertext-only attacks (COA).

For symmetric encryption, if a scheme provides **authenticated encryption** (ensuring both confidentiality and integrity/authenticity of the ciphertext), it often implies IND-CCA2 security.

Regarding non-malleability and indistinguishability:

* It has been shown that **IND-CCA2 security is equivalent to NM-CCA2 security** for public-key encryption schemes. This is a powerful result, meaning that if a scheme protects against an adversary distinguishing plaintexts even with a decryption oracle, it also protects against them meaningfully modifying ciphertexts.
* Similarly, IND-CPA is related to NM-CPA, though the equivalences can be more nuanced depending on the specific definitions.

In practice, achieving **IND-CCA2** (or its equivalent NM-CCA2) is the goal for most modern public-key encryption schemes deployed in critical applications, as it provides the strongest guarantees against powerful, active adversaries. For symmetric encryption, IND-CPA is a common goal (often achieved by modes like CBC or CTR with random IVs), and authenticated encryption modes provide even stronger guarantees, often reaching levels comparable to IND-CCA2.

---
## Key Takeaways

Understanding these formal security notions is vital for appreciating the rigor behind modern cryptographic design and analysis:

* Security isn't just about an attacker not finding the key; it's about preventing them from learning *any* useful information about the plaintext (**Indistinguishability**) or from meaningfully altering ciphertexts (**Non-Malleability**).
* These properties are defined using "games" against adversaries with different capabilities:
  * **IND-CPA** (Chosen Plaintext Attack) is a baseline, requiring probabilistic encryption.
  * **IND-CCA1** (Non-Adaptive Chosen Ciphertext Attack) provides stronger guarantees.
  * **IND-CCA2** (Adaptive Chosen Ciphertext Attack) is a very strong standard, protecting against active attacks involving decryption oracles.
  * Similar notions (NM-CPA, NM-CCA2) exist for non-malleability.
* There's a hierarchy of these security notions, with IND-CCA2 often considered equivalent to NM-CCA2 and representing a high level of security.

While the details can be quite mathematical, the intuition is that these formal models push cryptosystems to their limits, ensuring they are robust against a wide array of sophisticated attacks.
