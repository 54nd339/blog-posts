---
title: The Public Key Panorama - RSA, ECC, Probabilistic & Homomorphic Encryption, and More
description: Exploring the diverse world of public key cryptography beyond RSA, including probabilistic and homomorphic encryption, Elliptic Curve Cryptosystems (ECC), Blum-Goldwasser, and Identity Based Encryption (IBE).
date: 2023-06-13
draft: false
slug: /pensieve/cyber-security/public-key
tags:
  - Cyber Security
  - CS Basics
---

Welcome back, crypto-explorers! We've previously laid the groundwork for asymmetric (public-key) cryptography, with a deep dive into the mechanics of RSA and the mathematical principles like prime numbers and factorization that make it secure. Public-key cryptography, with its revolutionary concept of separate keys for encryption and decryption, opened up a new world of possibilities for secure communication and digital trust.

Today, we'll broaden our view of this public-key panorama. While RSA is a giant in this field, it's not the only player. We'll explore other innovative public-key systems and crucial concepts like probabilistic encryption, the almost magical homomorphic encryption, the efficient Elliptic Curve Cryptosystems (ECC), and more!

---
## A Quick Refresher: Public Key Cryptography Principles

At its heart, public-key cryptography involves a pair of mathematically linked keys for each user:

* A **public key**, which can be shared with anyone, used for encrypting messages to the user or verifying their digital signatures.
* A **private key**, kept secret by the user, used for decrypting messages sent to them or for creating digital signatures.

The strength of these systems often relies on computationally "hard" mathematical problems – operations that are easy to do in one direction (like multiplying two large primes to get $N$ for RSA) but extremely difficult to reverse without knowing a secret (like factoring $N$ to find $p$ and $q$ if you don't have the private key).

**RSA (Rivest-Shamir-Adleman)**, as we've seen, relies on the difficulty of factoring the product of two large primes. It allows for both encryption and digital signatures.

Now, let's see what else the public-key world has to offer.

---
## Probabilistic Encryption: Adding Randomness for Robust Security

A deterministic encryption algorithm always produces the same ciphertext for the same plaintext and key. In public-key cryptography, where the public key is known to everyone, this is a problem! If an attacker suspects a message is one of a few possibilities (e.g., "YES" or "NO"), they can encrypt each possibility with the public key and compare it to the intercepted ciphertext to learn the plaintext. This means deterministic public-key schemes fail the IND-CPA (Indistinguishability under Chosen Plaintext Attack) security notion.

**Probabilistic Encryption** solves this by introducing randomness into the encryption process. Encrypting the same plaintext multiple times with the same public key will produce different ciphertexts.

* **Goal:** To make it infeasible for an attacker to distinguish between encryptions of different messages, or even between an encryption of a known message and random garbage. This achieves semantic security (IND-CPA).
* **How it's often achieved:**
  * **Randomized Padding Schemes:** For algorithms like RSA, this is typically done by adding random padding to the message before encryption. RSA-OAEP (Optimal Asymmetric Encryption Padding) is a common standard that incorporates randomness and other features to provide strong security.
  * **Inherently Probabilistic Schemes:** Some cryptosystems are designed to be probabilistic from the ground up.

### Blum-Goldwasser Cryptosystem: A Probabilistic Approach

The Blum-Goldwasser (BG) cryptosystem, developed by Manuel Blum and Shafi Goldwasser in 1984, is a public-key scheme that is inherently probabilistic and whose security can be proven to be equivalent to the difficulty of factoring a special type of integer.

* **Key Generation:**
  1. Choose two large distinct prime numbers $p$ and $q$, both congruent to $3 \pmod 4$. (Primes of this form ensure that $-1$ is a quadratic non-residue modulo $p$ and $q$, and that finding square roots is easier for those who know $p$ and $q$).
  2. Compute $N = pq$. This $N$ is called a **Blum integer**.
  3. The public key is $N$. The private key is $(p, q)$.
* **Encryption (of a single bit $m_i$ at a time, or a block of bits):**
  1. Choose a random integer $x_0$ such that $\text{gcd}(x_0, N)=1$.
  2. Generate a pseudorandom sequence of bits (keystream) $b_1, b_2, \ldots, b_k$:
     * $x_{i+1} = x_i^2 \pmod N$
     * $b_{i+1} = x_{i+1} \pmod 2$ (or some other parity bit/hardcore predicate of $x_{i+1}$)
  3. The ciphertext for a plaintext $M = m_1m_2\ldots m_k$ is $(c_1, c_2, \ldots, c_k, x_{k+1})$, where $c_i = m_i \oplus b_i$.
  4. $x_{k+1}$ (the value of $x$ after generating $k$ bits of keystream) is sent along with the encrypted bits.
* **Decryption:**
  1. The receiver, knowing $p$ and $q$, can efficiently compute the "square roots" needed to reverse the $x_{i+1} = x_i^2 \pmod N$ sequence, starting from the received $x_{k+1}$ to reconstruct $x_k, x_{k-1}, \ldots, x_1$. This requires knowing the factorization of $N$.
  2. Once the $x_i$ values are recovered, the keystream bits $b_i$ can be regenerated, and the plaintext bits $m_i = c_i \oplus b_i$ can be recovered.
* **Security:** BG is semantically secure (IND-CPA) under the assumption that determining whether a number is a quadratic residue modulo $N$ (when $N$ is a Blum integer) is computationally hard, which is related to the difficulty of factoring $N$.

---
## Homomorphic Encryption: Computing on Encrypted Data (The Holy Grail?)

Imagine a world where you could send your sensitive data to a cloud service, have it perform computations on that data *without ever decrypting it*, and then get the encrypted result back, which only you can decrypt. This is the promise of **Homomorphic Encryption (HE)**.

* **Core Idea:** A cryptosystem is homomorphic if it allows certain computations to be performed on ciphertexts, such that the decryption of the resulting ciphertext matches the result of performing the equivalent operations on the original plaintexts.
  Symbolically, if $E(m)$ is the encryption of message $m$:
  * An **additively homomorphic** scheme allows $E(m_1) \oplus_{enc} E(m_2) = E(m_1 + m_2)$.
  * A **multiplicatively homomorphic** scheme allows $E(m_1) \otimes_{enc} E(m_2) = E(m_1 \times m_2)$.
* **Types:**
  * **Partially Homomorphic Encryption (PHE):** Supports *either* addition or multiplication on ciphertexts, but not both (e.g., RSA without padding is multiplicatively homomorphic; Paillier cryptosystem is additively homomorphic).
  * **Somewhat Homomorphic Encryption (SHE/SWHE):** Supports a limited number of *both* addition and multiplication operations.
  * **Fully Homomorphic Encryption (FHE):** Supports an arbitrary number of additions and multiplications on ciphertexts, effectively allowing any computable function to be performed on encrypted data. Craig Gentry first demonstrated a plausible FHE scheme in 2009.
* **Applications:** Secure cloud computing (processing sensitive data without giving the cloud provider access), privacy-preserving data mining, secure outsourced computations, encrypted search.
* **Challenges:** FHE schemes are currently very computationally intensive (slower and produce larger ciphertexts than traditional encryption), making them impractical for many real-time applications, but research is rapidly advancing.

---
## Elliptic Curve Cryptosystems (ECC): Leaner, Meaner Security

Elliptic Curve Cryptography (ECC) is an approach to public-key cryptography based on the algebraic structure of elliptic curves over finite fields.

* **What is an Elliptic Curve (simplified)?**
  * Over real numbers, an elliptic curve can be visualized as a curve defined by an equation like $y^2 = x^3 + ax + b$ (for specific $a,b$ such that $4a^3 + 27b^2 \neq 0$ to avoid singularities).
  * In cryptography, these curves are defined over **finite fields**, typically GF($p$) (where $p$ is a large prime) or GF($2^m$). The "points" on the curve are pairs $(x,y)$ that satisfy the curve equation within that finite field, plus a special "point at infinity" ($\mathcal{O}$).
* **The Magic: Elliptic Curve Group Operation:**
  * A special "addition" operation can be defined for points on an elliptic curve. Geometrically (over reals), if you draw a line through two points $P$ and $Q$ on the curve, it will intersect the curve at a third point, say $-R$. The "sum" $P+Q$ is then defined as $R$ (the reflection of $-R$ across the x-axis). There are algebraic formulas for this addition.
  * This addition operation, along with the point at infinity (acting as the identity element), forms an **Abelian group**.
* **Elliptic Curve Discrete Logarithm Problem (ECDLP):**
  * Given a base point $G$ on the curve and another point $P = kG$ (which means adding $G$ to itself $k$ times), it is computationally very difficult to find the integer $k$ if $k$ is large. This is the ECDLP.
* **Advantages of ECC:**
  * **Smaller Key Sizes:** ECC can provide equivalent security to RSA and other discrete log systems (like Diffie-Hellman over $\mathbb{Z}_p^*$) with significantly smaller key sizes. For example, a 256-bit ECC key is generally considered to offer similar security to a 3072-bit RSA key.
  * **Efficiency:** Smaller keys lead to faster computations, lower power consumption, and reduced bandwidth requirements. This makes ECC ideal for resource-constrained devices like smartphones, IoT devices, and smart cards.
* **Applications:**
  * **ECDH (Elliptic Curve Diffie-Hellman):** A key agreement protocol analogous to Diffie-Hellman, but using elliptic curve groups.
  * **ECDSA (Elliptic Curve Digital Signature Algorithm):** A widely used algorithm for digital signatures, analogous to DSA but based on ECC.
  * Used extensively in modern TLS, cryptocurrencies (like Bitcoin and Ethereum), mobile messaging apps.

---
## Identity Based Encryption (IBE): Public Keys from Real-World Identities

Traditional PKI requires users to obtain certificates to bind their public keys to their identities. Identity Based Encryption, proposed by Adi Shamir in 1984 and first practically realized by Boneh and Franklin in 2001, offers an alternative.

* **Core Idea:** A user's public key can be a string directly derived from their real-world identity, such as an email address, phone number, or name.
* **Key Components:**
  * **Private Key Generator (PKG):** A trusted third party that holds a **master secret key**.
  * **Key Generation:** When Alice wants to send an encrypted message to Bob (e.g., `bob@example.com`), Alice uses Bob's identity string (`bob@example.com`) as his public key to encrypt the message.
  * To decrypt, Bob contacts the PKG, authenticates himself, and the PKG uses its master secret key and Bob's identity string to generate Bob's corresponding **private key**. Bob can then use this private key to decrypt messages.
* **Advantages:**
  * Simplified key management for users: no need to look up or distribute public key certificates for senders. The public key is just the recipient's known identity.
* **Challenges/Disadvantages:**
  * **Key Escrow:** The PKG knows everyone's private key (as it generates them), which is a significant trust assumption and potential single point of compromise.
  * **PKG Trust and Availability:** The PKG must be highly secure and continuously available.
  * Revocation can be more complex.
* **Use Cases:** While not as widespread as traditional PKI, IBE has found use in specific enterprise environments or systems where a central trusted authority is acceptable and key management simplification is a high priority.

---
## Cryptographic Hash Functions: Their Role Revisited

While we've detailed hash functions previously, it's worth reiterating their vital role in conjunction with public-key cryptography:

* **Digital Signatures:** Messages are almost always hashed first, and then the relatively small hash value is signed (encrypted with the private key) rather than the entire message. This is for efficiency and security.
* **Integrity Checks:** Verifying that a public key or certificate hasn't been tampered with.
* **Key Derivation:** Sometimes used in the process of deriving cryptographic keys.
* **Padding Schemes:** Used within constructions like RSA-OAEP to provide randomness and structure to messages before public-key encryption.

---
## Key Takeaways

The world of public-key cryptography extends far beyond the foundational RSA:

* **Probabilistic Encryption** (like Blum-Goldwasser, or achieved via padding like RSA-OAEP) is essential for achieving strong semantic security (IND-CPA).
* **Homomorphic Encryption** holds the revolutionary promise of computing directly on encrypted data, though it's still largely in the research and development phase for practical, widespread use.
* **Elliptic Curve Cryptography (ECC)** offers equivalent security to older systems with much smaller key sizes, making it highly efficient and widely adopted in modern applications.
* **Identity Based Encryption (IBE)** provides an alternative to traditional PKI by allowing public keys to be derived from user identities, simplifying key discovery but introducing a trusted Private Key Generator.

These diverse systems and concepts provide a rich toolbox for cryptographers and engineers to build secure communication and data protection solutions, each with its own trade-offs in terms of security properties, efficiency, and underlying mathematical assumptions.
