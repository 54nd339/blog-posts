---
title: The Two-Key Tango - Asymmetric Cryptography and Its Mathematical Magic
description: An introduction to asymmetric (public-key) cryptography, exploring its mathematical foundations (primes, factorization, CRT) and key concepts like RSA, hash functions (MD5, SHA), MAC, HMAC, digital signatures, and digital envelopes.
date: 2023-06-08
draft: false
slug: /pensieve/cyber-security/asymmetric-key
tags:
  - Cyber Security
  - CS Basics
---

Welcome back, codebreakers and privacy advocates! In our previous cryptographic journey, we explored symmetric key cryptography, where a single secret key is the gatekeeper for both locking (encrypting) and unlocking (decrypting) information. But what if you need to communicate securely with someone you've never met, without a pre-shared secret? Or what if you need to verify the authenticity and integrity of a message from a sender?

This is where the elegance of **Asymmetric Key Cryptography**, also known as **Public Key Cryptography**, shines! It employs a pair of keys – one public, one private – to perform its magic. Today, we'll unravel the mathematical principles that make this possible and explore some of its most vital applications.

---
## The Mathematical Pillars of Asymmetric Cryptography

Asymmetric cryptography often relies on mathematical problems that are computationally easy to perform in one direction but incredibly hard to reverse without special knowledge (like a private key). This "one-wayness" is its secret sauce.

### 1. Prime Numbers: The Indivisible Building Blocks
A **prime number** is a natural number greater than 1 that has no positive divisors other than 1 and itself. Examples include 2, 3, 5, 7, 11, 13, 17, 19, etc.
* **Why are they crucial?** Many asymmetric algorithms, most notably RSA, rely on the properties of very large prime numbers (often hundreds of digits long). The security of these systems often hinges on the difficulty of factoring a large number that is the product of two such primes.
* Finding large primes is a critical first step in generating key pairs for these systems.

### 2. Primality Testing: Is This Number Prime?
Given that we need large prime numbers, how do we find them or verify if a given large number is indeed prime?
* **Trial Division:** The simplest method is to try dividing the number by all integers up to its square root. This is too slow for very large numbers.
* **Probabilistic Primality Tests:** These tests don't definitively prove a number is prime but can determine if it's "probably prime" with an extremely high degree of certainty (low error probability).
    * **Fermat's Little Theorem states:** If $p$ is a prime number, then for any integer $a$ not divisible by $p$, we have $a^{p-1} \equiv 1 \pmod p$. A Fermat primality test picks a random $a$ and checks if this congruence holds. If it doesn't, the number is definitely composite. If it does, the number *might* be prime (some composite numbers, called Carmichael numbers, pass this test for all $a$ coprime to them).
    * **Miller-Rabin Test:** This is a more sophisticated and widely used probabilistic test. It's an extension of the Fermat test that has a much lower probability of being fooled by composite numbers. If a number passes multiple rounds of Miller-Rabin with different random bases $a$, it is considered prime for cryptographic purposes with very high confidence.
* **Deterministic Primality Tests:** Algorithms like AKS (Agrawal-Kayal-Saxena) can definitively prove primality but are generally slower than probabilistic tests for the sizes of numbers used in cryptography.

### 3. Factorization: The Hard Problem
**Integer factorization** is the process of decomposing a composite number into a product of smaller integers (preferably its prime factors).
* Example: $35 = 5 \times 7$.
* **The Challenge:** While multiplying two large primes is easy, factoring their very large product (e.g., 2048 bits or more) back into the original two primes is computationally infeasible with current algorithms and computing power if the primes are chosen appropriately.
* **Relevance:** The security of the RSA algorithm directly relies on the difficulty of factoring the public modulus $N$ (which is a product of two large primes $p$ and $q$) into its prime factors $p$ and $q$. If an attacker could factor $N$, they could derive the private key.

### 4. Chinese Remainder Theorem (CRT)
The Chinese Remainder Theorem provides a way to solve a system of simultaneous linear congruences with coprime moduli.
* **Statement (Simplified):** If you know the remainders of an integer $x$ when divided by several pairwise coprime integers $n_1, n_2, \ldots, n_k$, then you can uniquely determine the remainder of $x$ when divided by the product $N = n_1 n_2 \cdots n_k$.
    That is, if you have:
    $x \equiv a_1 \pmod {n_1}$
    $x \equiv a_2 \pmod {n_2}$
    ...
    $x \equiv a_k \pmod {n_k}$
    where $n_i$ are pairwise coprime, there exists a unique solution for $x$ modulo $N$.
* **Applications in Cryptography:**
    * **Speeding up RSA Decryption/Signing:** RSA private key operations involve modular exponentiation $m = c^d \pmod N$. If the private key holder knows the prime factors $p$ and $q$ of $N$ (where $N=pq$), CRT can be used. Instead of computing $c^d \pmod {pq}$, they can compute:
        $m_1 = c^{d_p} \pmod p$ (where $d_p = d \pmod{p-1}$)
        $m_2 = c^{d_q} \pmod q$ (where $d_q = d \pmod{q-1}$)
        Then, combine $m_1$ and $m_2$ using CRT to find $m \pmod {pq}$. Operations modulo $p$ and $q$ are much faster than operations modulo $N$, leading to a significant speedup (often around 4 times faster).

---
## Asymmetric Key Cryptography in Practice: The Two-Key System

Asymmetric cryptography uses two distinct but mathematically related keys:
* **Public Key:** This key can be freely distributed to anyone without compromising security. It's used for encrypting messages intended for the key pair owner or for verifying digital signatures made by the key pair owner.
* **Private Key:** This key must be kept secret by its owner. It's used for decrypting messages encrypted with the corresponding public key or for creating digital signatures.

Anyone can encrypt a message using your public key, but only you (with your private key) can decrypt it. Similarly, only you can sign a message with your private key, but anyone can verify your signature using your public key.

### RSA Algorithm (Rivest-Shamir-Adleman)
Named after its inventors, RSA is the most widely used public-key cryptosystem. Its security relies on the difficulty of factoring large integers.

**1. Key Generation:**
   a.  Choose two distinct large random prime numbers, $p$ and $q$.
   b.  Compute their product $N = p \times q$. $N$ is the modulus for both public and private keys. Its length (e.g., 2048 bits) is the key length.
   c.  Compute Euler's totient function of $N$: $\phi(N) = (p-1)(q-1)$. $\phi(N)$ is the number of positive integers less than or equal to $N$ that are relatively prime to $N$.
   d.  Choose an integer $e$ (the public key exponent) such that $1 < e < \phi(N)$ and $\text{gcd}(e, \phi(N)) = 1$. A common choice for $e$ is $65537$ ($2^{16}+1$).
   e.  Compute $d$ (the private key exponent) such that $d \cdot e \equiv 1 \pmod{\phi(N)}$. $d$ is the modular multiplicative inverse of $e$ modulo $\phi(N)$. This can be found using the Extended Euclidean Algorithm.
   * **Public Key:** $(N, e)$
   * **Private Key:** $(N, d)$ (or more securely, $(p, q, d, d_p, d_q, q_{inv})$ where $d_p=d \pmod{p-1}$, $d_q=d \pmod{q-1}$, $q_{inv}=q^{-1} \pmod p$ for CRT optimization). $p$ and $q$ must also be kept secret.

**2. Encryption:**
   To encrypt a plaintext message $M$ (represented as an integer $0 \le M < N$), the sender uses the recipient's public key $(N, e)$:
   Ciphertext $C = M^e \pmod N$.

**3. Decryption:**
   To decrypt the ciphertext $C$, the recipient uses their private key $(N, d)$:
   Plaintext $M = C^d \pmod N$.
   This works because $C^d \equiv (M^e)^d \equiv M^{ed} \pmod N$. Since $ed \equiv 1 \pmod{\phi(N)}$, $ed = k\phi(N) + 1$ for some integer $k$. By Euler's theorem, if $\text{gcd}(M,N)=1$, then $M^{\phi(N)} \equiv 1 \pmod N$. So, $M^{ed} \equiv M^{k\phi(N)+1} \equiv (M^{\phi(N)})^k \cdot M^1 \equiv 1^k \cdot M \equiv M \pmod N$. (This also holds if $\text{gcd}(M,N) \neq 1$).

RSA is used for both encryption (of small amounts of data, like symmetric keys) and digital signatures.

### Cryptographic Hash Functions: Digital Fingerprints
A cryptographic hash function is a mathematical algorithm that takes an input (or "message") of arbitrary length and returns a fixed-size string of bits, called a **hash value**, **message digest**, or simply **hash**. It's important to distinguish these from non-cryptographic hash functions (like simple checksums, e.g., CRC used in networking for error detection), which are designed to detect accidental data corruption rather than resist malicious attempts to find collisions or reverse the hash.

**Key Properties of Cryptographic Hash Functions:**
1.  **Pre-image Resistance (One-way):** Given a hash value $h$, it should be computationally infeasible to find any input $m$ such that $H(m) = h$.
2.  **Second Pre-image Resistance (Weak Collision Resistance):** Given an input $m_1$, it should be computationally infeasible to find another input $m_2 \neq m_1$ such that $H(m_1) = H(m_2)$.
3.  **Collision Resistance (Strong Collision Resistance):** It should be computationally infeasible to find any two distinct inputs $m_1$ and $m_2$ such that $H(m_1) = H(m_2)$.
4.  **Deterministic:** The same input message always results in the same hash value.
5.  **Avalanche Effect:** A small change in the input message (e.g., flipping a single bit) should produce a significantly different hash value.

Hash functions are used for verifying data integrity, password storage (by hashing passwords), digital signatures, and more.

#### The Birthday Problem and Collision Attacks
The "Birthday Problem" in probability shows that in a group of just 23 people, there's a greater than 50% chance that at least two share the same birthday. This concept applies to hash functions: finding *any* two inputs that hash to the same output (a collision) is significantly easier than finding an input that hashes to a *specific* pre-determined output (pre-image attack) or finding another input that hashes to the same value as a *specific known* input (second pre-image attack).

For a hash function producing an $n$-bit output, a brute-force collision attack (a "birthday attack") has an average complexity of roughly $2^{n/2}$ operations, rather than $2^n$ for a pre-image attack. This means that to achieve $k$ bits of security against collision attacks, a hash function needs to produce an output of $2k$ bits. This is why newer hash functions have larger output sizes.

* **MD5 (Message Digest 5):**
    * Produces a 128-bit hash value.
    * **Security Status:** MD5 is considered broken for collision resistance (collisions can be found in seconds) and should no longer be used for security purposes like digital signatures or SSL certificates. It offers only $2^{64}$ collision resistance due to the birthday attack.
* **SHA (Secure Hash Algorithm) Family:** A series of cryptographic hash functions developed by NIST.
    * **SHA-1:** Produces a 160-bit hash value. Collision resistance is around $2^{80}$. Like MD5, SHA-1 is now considered insecure against well-funded attackers, and its use is being phased out. Collisions have been publicly demonstrated.
    * **SHA-2:** A family of hash functions with different digest sizes: SHA-224, SHA-256, SHA-384, SHA-512, SHA-512/224, SHA-512/256. SHA-256 (collision resistance $2^{128}$) and SHA-512 are commonly used and currently considered secure.
    * **SHA-3 (Keccak):** Selected through a public competition to provide an alternative to SHA-2, with a different internal structure ("sponge construction"). It also offers various output sizes and is considered secure.

### Message Authentication Code (MAC): Authenticity and Integrity with Symmetric Keys
A MAC is a short piece of information used to authenticate a message and ensure its integrity. Unlike digital signatures (which use asymmetric keys), MACs are generated and verified using a **shared secret key (symmetric key)**.
* **How it works:**
    1. The sender combines the message with the shared secret key and processes it with a MAC algorithm to produce a MAC tag.
    2. The sender transmits the message and the MAC tag.
    3. The receiver, who also knows the shared secret key, independently computes the MAC tag on the received message using the same algorithm and key.
    4. If the computed tag matches the received tag, the receiver is assured that the message came from a party who knows the secret key and that the message has not been altered.
* **Limitation:** MACs do not provide non-repudiation, as anyone who knows the secret key can generate the MAC.

### HMAC (Hash-based MAC): MACs from Hash Functions
HMAC is a specific type of MAC that uses a cryptographic hash function (like SHA-256) along with a secret key.
* It processes the message and key through two nested hash operations, providing strong security guarantees that are tied to the properties of the underlying hash function.
* The construction is typically `HMAC(K, m) = H( (K' ⊕ opad) || H( (K' ⊕ ipad) || m) )`, where $K'$ is the key (derived from $K$), `ipad` and `opad` are specific padding constants, `H` is the hash function, and `||` denotes concatenation.
* HMACs are widely used due to their efficiency and proven security when used with secure hash functions.

### Digital Envelope: Securely Sending Symmetric Keys
Symmetric encryption is fast for large amounts of data, but distributing the symmetric key securely is a challenge. Asymmetric encryption is slower but excels at key distribution. A **digital envelope** combines these strengths:
1.  The sender generates a random symmetric key (session key) for the current message.
2.  The message is encrypted using this symmetric session key.
3.  The symmetric session key itself is then encrypted using the recipient's **public key** (asymmetric encryption).
4.  The sender transmits both the (symmetrically) encrypted message and the (asymmetrically) encrypted session key to the recipient.
5.  The recipient uses their **private key** to decrypt the symmetric session key.
6.  With the decrypted session key, the recipient decrypts the main message.

This technique is commonly used in secure email (PGP, S/MIME) and secure communication protocols like TLS.

### Digital Signature: Authenticity, Integrity, and Non-Repudiation
A digital signature, created using asymmetric cryptography (like RSA), provides:
* **Authentication:** Verifies that the message was sent by the claimed sender (owner of the private key).
* **Integrity:** Ensures the message has not been altered since it was signed.
* **Non-repudiation:** The sender cannot plausibly deny having signed the message, as only they have access to the private key used for signing.

**How it typically works (e.g., with RSA):**
1.  **Signing (Sender):**
    a.  The sender takes the message ($M$) and computes its hash value: $h = H(M)$. Using the hash is crucial for efficiency (signing a small hash is faster than a large message) and security (prevents certain attacks).
    b.  The sender then "encrypts" (or more accurately, performs a signing operation on) this hash value using their **own private key** ($d, N$): Signature $S = h^d \pmod N$.
    c.  The sender sends the original message $M$ (or $h$) along with the signature $S$.

2.  **Verification (Receiver):**
    a.  The receiver receives $M$ and $S$.
    b.  The receiver independently computes the hash of the received message: $h' = H(M)$.
    c.  The receiver "decrypts" (or more accurately, performs a verification operation on) the received signature $S$ using the sender's **public key** ($e, N$): $h_v = S^e \pmod N$. This should yield the original hash $h$ if the signature is valid (i.e., $h_v = (h^d)^e \equiv h \pmod N$).
    d.  The receiver compares $h_v$ with their computed hash $h'$. If they match ($h_v = h'$), the signature is considered valid. This confirms the message originated from the owner of the public key and has not been altered since signing.

---
## Conclusion: The Power of Two Keys

Asymmetric cryptography, with its ingenious use of public and private key pairs built upon hard mathematical problems like factorization, fundamentally changed secure communication. It elegantly solves the problem of key distribution for confidentiality (when used to encrypt session keys for symmetric ciphers) and provides robust mechanisms like digital signatures for ensuring authenticity, integrity, and non-repudiation.

Understanding the mathematical foundations—primes, primality testing, factorization, and the Chinese Remainder Theorem—helps appreciate the strength and subtleties of algorithms like RSA. Combined with cryptographic hash functions and MACs, these tools form a critical part of our modern security infrastructure, enabling trust in a world of open networks.
