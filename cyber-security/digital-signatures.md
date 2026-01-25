---
title: Digital Seals of Trust - Advanced Signatures (ElGamal, Schnorr), Blind Signatures, and E-Voting Concepts
description: Exploring advanced digital signatures like ElGamal and Schnorr, the formal security notion of EUF-CMA, the intriguing concept of blind signatures, and cryptographic applications in electronic voting.
date: 2023-06-14
draft: false
slug: /pensieve/cyber-security/digital-signatures
tags:
  - Cyber Security
  - CS Basics
---

Welcome back, guardians of digital integrity! We've seen how asymmetric cryptography and hash functions lay the groundwork for **digital signatures**, providing authenticity, integrity, and non-repudiation for our messages. While RSA signatures are a cornerstone, the world of digital signatures is richer and more diverse.

Today, we'll delve deeper. We'll start by understanding what it *formally* means for a signature scheme to be secure. Then, we'll explore alternative signature schemes like ElGamal and Schnorr, uncover the fascinating concept of blind signatures where content is signed unseen, and briefly touch upon the complex application of cryptographic principles in electronic voting.

---
## Digital Signatures Revisited: The Core Promise

Let's quickly recap: a digital signature scheme typically involves two main processes:

1. **Signing:** The sender uses their **private key** to sign a message (usually the hash of the message).
2. **Verification:** Anyone with the sender's **public key** can verify the signature to confirm the sender's identity and that the message hasn't been tampered with.

This provides:

* **Authenticity:** The public key verifies the signature was created by the corresponding private key holder.
* **Integrity:** As the signature is usually on a hash of the message, any change to the message will invalidate the signature.
* **Non-repudiation:** The signer cannot easily deny having signed the message, as only they possess the private key.

---
## Defining "Secure" Signatures: Existential Unforgeability (EUF-CMA)

How do we measure the security of a digital signature scheme? One of the strongest and most widely accepted formal security notions is **Existential Unforgeability under a Chosen Message Attack (EUF-CMA)**.

* **The "Game":**
  1. **Setup:** The adversary is given the signer's public key.
  2. **Queries (Chosen Message Attack):** The adversary can request signatures on a number of messages of their choice ($m_1, m_2, \ldots, m_k$) from a "signing oracle" (which has the private key). The oracle provides the valid signatures ($\sigma_1, \sigma_2, \ldots, \sigma_k$).
  3. **Forgery Attempt:** The adversary's goal is to output a new message-signature pair $(m^*, \sigma^*)$ such that:
     * $m^*$ is a message that the adversary *did not* previously query to the signing oracle.
     * $\sigma^*$ is a valid signature for $m^*$ under the given public key.
* **Winning the Game:** The adversary "wins" if they succeed in producing such a valid forgery for a new message.
* **Security Definition:** A digital signature scheme is EUF-CMA secure if any probabilistic polynomial-time (PPT) adversary has at most a negligible probability of winning this game.
* **"Existential" Unforgeability:** This is a very strong guarantee. It means the adversary cannot create a valid signature for *any* new message, even if that message is nonsensical or has no particular meaning. They don't have to forge a signature on a *specific, meaningful* message; any new valid pair will do.

Achieving EUF-CMA security is a primary goal for modern digital signature schemes.

---
## Alternative Digital Signature Schemes

While RSA signatures are common, other schemes based on different mathematical problems also exist, offering various trade-offs in terms of signature size, speed, and underlying assumptions.

### ElGamal Digital Signature Scheme

The ElGamal signature scheme, developed by Taher Elgamal in 1985, is based on the difficulty of the **Discrete Logarithm Problem (DLP)** in a cyclic group, typically the multiplicative group of integers modulo a prime $p$ (i.e., $\mathbb{Z}_p^*$).

* **Key Generation:**

  1. Choose a large prime $p$ and a generator $g$ of the multiplicative group $\mathbb{Z}_p^*$.
  2. Choose a random integer $x$ (private key) such that $1 < x < p-1$.
  3. Compute $y = g^x \pmod p$ (public key).

  * Public parameters: $(p, g, y)$. Private key: $x$.
* **Signing a message $M$:**

  1. Choose a random secret integer $k$ (a per-message secret, or nonce) such that $1 < k < p-1$ and $\text{gcd}(k, p-1) = 1$. **This $k$ must be unique for each signature and kept secret.**
  2. Compute $r = g^k \pmod p$.
  3. Compute $s = (H(M) - xr)k^{-1} \pmod{p-1}$, where $H(M)$ is the hash of the message and $k^{-1}$ is the modular multiplicative inverse of $k$ modulo $p-1$.

  * The signature is the pair $(r, s)$.
* **Verification of signature $(r, s)$ on message $M$:**

  1. Verify that $1 \le r < p$. If not, reject.
  2. Compute $V_1 = y^r r^s \pmod p$. (Note: $r^s = (g^k)^s = g^{ks} \pmod p$)
  3. Compute $V_2 = g^{H(M)} \pmod p$.
  4. The signature is valid if $V_1 \equiv V_2 \pmod p$.

  * **Why it works:** $y^r r^s \equiv (g^x)^r (g^k)^s \equiv g^{xr} g^{ks} \equiv g^{xr+ks} \pmod p$.
    Since $s \equiv (H(M) - xr)k^{-1} \pmod{p-1}$, then $ks \equiv H(M) - xr \pmod{p-1}$.
    So, $xr+ks \equiv xr + H(M) - xr \equiv H(M) \pmod{p-1}$.
    Thus, $g^{xr+ks} \equiv g^{H(M)} \pmod p$.
* **Characteristics:** The signature is probabilistic due to the random $k$. If $k$ is ever reused or compromised, the private key $x$ can be easily found. This makes proper random number generation critical. Variations like DSA (Digital Signature Algorithm) were developed based on ElGamal.

### Schnorr Signature Scheme

The Schnorr signature scheme, developed by Claus Schnorr, is also based on the difficulty of the DLP and is known for its relative simplicity and for being one of the first schemes with a provable security reduction (to the DLP) in a random oracle model.

* **Key Generation:** Similar to ElGamal: choose a prime $p$, a prime $q$ that divides $p-1$, a generator $g$ of order $q$ modulo $p$. Choose a private key $x$ and compute public key $y = g^x \pmod p$.

  * Public parameters: $(p, q, g, y)$. Private key: $x$.
* **Signing a message $M$:**

  1. Choose a random secret integer $k$ (nonce), $1 \le k < q$.
  2. Compute $r = g^k \pmod p$.
  3. Compute $e = H(M || r)$ (hash of the concatenation of message and $r$).
  4. Compute $s = (k - xe) \pmod q$.

  * The signature is the pair $(e, s)$. (Sometimes $(r,s)$ is used, where $e$ is derived from $r$).
* **Verification of signature $(e, s)$ on message $M$:**

  1. Compute $r_v = g^s y^e \pmod p$. (This is $g^{k-xe} (g^x)^e = g^{k-xe+xe} = g^k \pmod p$).
  2. Compute $e_v = H(M || r_v)$.
  3. The signature is valid if $e_v = e$.
* **Characteristics:** Schnorr signatures are generally smaller than ElGamal/DSA signatures for equivalent security. It was patented for a time, which limited its early adoption, but the patent has since expired. It forms the basis for many modern signature schemes and has desirable properties for threshold signatures and multi-signatures.

---
## Blind Signatures: Signing Without Seeing the Content

A **blind signature**, a concept introduced by David Chaum, is a form of digital signature in which the content of a message is disguised (blinded) before it is signed. The signer signs the blinded message without knowing its actual content. The resulting signature can then be "unblinded" by the original requester to yield a valid signature on the original, unblinded message.

* **Analogy:** Imagine putting a letter (message) inside an envelope with a piece of carbon paper on top of the letter, then sealing the envelope. You give this sealed envelope to someone to sign. They sign the outside of the envelope. Due to the carbon paper, their signature also transfers onto the letter inside. You then open the envelope and retrieve your letter, now bearing their signature, even though they never saw the letter's content.
* **General Process (e.g., using RSA):**

  1. **Blinding (Requester):** Alice wants Bob (the signer) to sign message $M$. Alice chooses a random secret "blinding factor" $r$. She computes a blinded message $M' = M \cdot r^e \pmod N$, where $(N,e)$ is Bob's public key. Alice sends $M'$ to Bob.
  2. **Signing (Signer):** Bob signs the blinded message $M'$ using his private key $d$: $S' = (M')^d \pmod N = (M \cdot r^e)^d \pmod N = M^d \cdot (r^e)^d \pmod N = M^d \cdot r^{ed} \pmod N = M^d \cdot r \pmod N$. Bob sends $S'$ back to Alice.
  3. **Unblinding (Requester):** Alice can remove the blinding factor from $S'$ to get the valid signature $S$ on her original message $M$. She computes $S = S' \cdot r^{-1} \pmod N = (M^d \cdot r) \cdot r^{-1} \pmod N = M^d \pmod N$.
* **Properties:**

  * The signer does not see the content of the message they are signing.
  * The unblinded signature is a valid signature on the original message, verifiable with the signer's public key.
  * The signer cannot link the blinded message they signed with the later unblinded signature-message pair (unlinkability, crucial for privacy).
* **Applications:**

  * **Digital Cash / E-cash Systems:** To allow anonymous spending. A bank can blindly sign a "digital coin" (a unique number), which the user can then unblind and spend anonymously. The bank can verify its own signature on the coin without knowing which user it originally issued the blinded coin to.
  * **Privacy-Preserving Protocols:** Such as secure electronic voting schemes where votes need to be authenticated (signed) but remain anonymous.

---
## Electronic Voting (E-Voting): A Cryptographic Challenge

Electronic voting aims to use electronic means to aid or take care of casting and counting votes. Designing secure and trustworthy e-voting systems is a significant challenge, and cryptography plays a vital role in attempting to meet the stringent requirements.

* **Key Security Requirements for E-Voting Systems:**
  * **Privacy/Anonymity/Ballot Secrecy:** Votes should remain secret; it should be impossible to link a voter to their vote.
  * **Integrity:** Votes must be recorded as cast and not altered. Vote counts must be accurate.
  * **Accuracy:** The system should correctly capture and count votes.
  * **Verifiability:**
    * *Individual Verifiability:* Voters should be able to check that their vote was correctly recorded.
    * *Universal Verifiability:* Anyone should be able to verify that the election outcome is correct based on the recorded votes, without compromising voter privacy.
  * **Authenticity:** Only eligible voters can vote, and only once.
  * **Uncoercibility/Receipt-Freeness:** Voters should not be able to prove to a third party (e.g., a vote buyer or coercer) how they voted. This prevents vote selling and coercion.
  * **Robustness/Availability:** The system should be able to withstand failures and attacks.
  * **Fairness:** No early results should be available before the polls close.
* **Role of Cryptography:**
  * **Encryption:** To protect the secrecy of votes during transmission and storage.
  * **Digital Signatures:** To authenticate voters, election officials, and voting machines, and to ensure the integrity of vote records and tallies.
  * **Hash Functions:** To ensure the integrity of vote data.
  * **Blind Signatures:** Explored in some protocols to allow voters to get their ballots validated (signed) by an authority without the authority knowing the vote itself, thus enabling anonymous but authorized voting.
  * **Homomorphic Encryption:** Theoretically allows for tallying encrypted votes without decrypting individual votes, enhancing privacy.
  * **Zero-Knowledge Proofs:** Can be used to prove certain properties (e.g., a vote is valid, a tally is correct) without revealing underlying sensitive information.
* **Challenges:** Despite the cryptographic tools, building a comprehensive e-voting system that satisfies all these properties perfectly and is also transparent, easy to use, and auditable by non-experts is extremely difficult. Concerns about software vulnerabilities, hardware tampering, insider threats, and overall system complexity remain significant. It's an active area of research and debate.

---
## Key Takeaways

Digital signatures are more than just an RSA application; they are a rich field with diverse schemes and profound security considerations.

* The formal notion of **Existential Unforgeability under Chosen Message Attack (EUF-CMA)** provides a strong benchmark for secure signature schemes.
* Signature schemes like **ElGamal** and **Schnorr**, both based on the Discrete Logarithm Problem, offer alternatives to RSA with different characteristics.
* **Blind signatures** offer a powerful cryptographic tool for achieving unlinkable authorization, crucial for privacy-centric applications like digital cash and some e-voting protocols.
* **Electronic voting** highlights the complex interplay of various cryptographic primitives in attempting to solve real-world challenges that require a high degree of security, privacy, and verifiability.

These advanced concepts underscore the ongoing innovation in cryptography, pushing the boundaries of what's possible in securing our digital interactions and societal processes.
