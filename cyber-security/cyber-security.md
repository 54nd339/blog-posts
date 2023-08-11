---
title: Cybersecurity - Goals, Cryptographic Attacks, and Cipher Basics
description: The CIA triad and the security goals around it, the taxonomy of cryptographic attacks from ciphertext-only to chosen-ciphertext, the classical substitution and transposition ciphers and why they fall, information hiding, and the difference between stream and block ciphers and their modes of operation.
date: 2023-08-11
draft: false
slug: /cyber-security/cyber-security
tags:
  - Security
  - Cryptography
---

Every secure system is defending something specific against someone specific. Before any algorithm, you name the goal — keep this secret, detect any tampering, stay available — and the threat model — what can the attacker see, what can they choose, what can they measure. This post is that groundwork: the security goals, the ways attackers come at a cipher, the classical ciphers and why they break, and the two families of modern symmetric encryption.

The rest of the category goes deeper: [symmetric](/citadel/cyber-security/symmetric-key) and [asymmetric](/citadel/cyber-security/asymmetric-key) key cryptography, the [maths underneath](/citadel/cyber-security/maths), [formal security notions](/citadel/cyber-security/formal-attacks), [digital signatures](/citadel/cyber-security/digital-signatures), [network security](/citadel/cyber-security/network-security), [software security](/citadel/cyber-security/info-security), and the [research frontier](/citadel/cyber-security/advanced).

## Security goals

Information security is usually anchored on the **CIA triad**:

- **Confidentiality** — information is accessible only to those authorised. Enforced with encryption and access control.
- **Integrity** — information stays accurate and unaltered through its lifecycle. Verified with hash functions.
- **Availability** — information and resources are reachable when authorised users need them. Threatened by denial-of-service and failures.

Around the triad sit further goals:

- **Authentication** — verifying that an entity is who it claims to be; multi-factor authentication strengthens it.
- **Non-repudiation** — a sender cannot later deny sending a message. Digital signatures provide it.
- **Access control** — deciding who may do what to which resource, often via role-based access control.
- **Data protection** — safeguarding sensitive data through encryption, secure storage, and legal compliance.

Cryptography itself splits into three categories: **symmetric key** (one shared secret for both encryption and decryption), **asymmetric (public) key** (a public key and a private key), and **cryptographic hash functions** (a fixed-size fingerprint of data, for integrity and signatures).

## Cryptographic attacks

A **cryptographic attack** — cryptanalysis — tries to circumvent a cryptosystem. The models are distinguished by what the attacker has access to:

| Attack | What the attacker has |
| --- | --- |
| **Ciphertext-only** | Only encrypted messages |
| **Known-plaintext** | Some plaintext and its matching ciphertext |
| **Chosen-plaintext** | Can encrypt plaintexts of their choice and see the ciphertexts |
| **Chosen-ciphertext** | Can decrypt ciphertexts of their choice and see the plaintexts |

Chosen-ciphertext is the most powerful model, and a scheme secure against it is secure against the weaker ones. Beyond these:

- **Brute force** — try every key. Feasibility depends on key size: an 8-bit key has $2^8 = 256$ possibilities; modern keys are long enough to make this infeasible.
- **Frequency analysis** — against simple substitution, compare symbol frequencies in the ciphertext with known language frequencies (E is the most common letter in English).
- **Man-in-the-middle** — intercept communication between two parties and alter it undetected.
- **Side-channel** — exploit the physical implementation rather than the algorithm: timing, power consumption, electromagnetic emission.
- **Denial of service against a cryptosystem** — flood an algorithm with work to disrupt availability.

Attackers are **passive** (eavesdrop without altering anything, hard to detect) or **active** (modify or inject data).

## Classical ciphers

**Substitution ciphers** replace units of plaintext with ciphertext units while keeping their order.

- **Caesar (shift) cipher** — shift each letter a fixed number of places. The key is the shift; there are only 25 non-trivial keys, so brute force breaks it instantly, and frequency analysis breaks it too.
- **Monoalphabetic substitution** — the ciphertext alphabet is an arbitrary permutation of the plaintext alphabet. The key space is $26! \approx 4 \times 10^{26}$, far too large for brute force — but letter frequencies are preserved, so frequency analysis still breaks it. Writing ciphertext in fixed-length blocks hides word boundaries but not the frequencies.

**Transposition ciphers** keep the plaintext letters and rearrange their order.

- **Rail fence** — write the plaintext diagonally over a number of "rails", then read off row by row. `HELLO WORLD` on 2 rails gives rows `HLOOL` and `ELWRD`, so ciphertext `HLOOLELWRD`.
- **Columnar transposition** — write the message in rows of fixed width (from a keyword), then read out column by column in the order given by the keyword's letters sorted alphabetically.
- **Scytale** — the ancient mechanical version: wrap a strip around a rod of a specific diameter, write along it, unwrap.

Transposition preserves the plaintext's letter frequencies, and the main attack is **anagramming** — rearranging the ciphertext letters into meaningful words.

## Information hiding

Sometimes the goal is to hide that a message exists at all.

- **Steganography** — concealing a message so no one suspects it is there. Techniques include **least-significant-bit insertion** (altering the low bits of image or audio samples, imperceptibly) and hiding data in unused file-header space. It differs from cryptography, where the encrypted message is obviously present but unreadable. The challenge is embedding enough data without creating statistically detectable anomalies in the cover medium.
- **Digital watermarking** — embedding information into a signal to prove authenticity or ownership. Watermarks can be **visible** (a channel logo) or **invisible** (detectable only by software), and **robust** (survive compression and scaling) or **fragile** (destroyed by any modification, so their absence signals tampering). Used for copyright, copy control, tamper detection, and broadcast monitoring.

## Stream versus block ciphers

Modern symmetric ciphers process plaintext in one of two ways.

**Stream ciphers** encrypt one symbol — a bit or byte — at a time. A **keystream** is generated from the key (and usually a nonce or IV) and XORed with the plaintext.

- Fast, and an error in one symbol does not propagate.
- Low diffusion — each ciphertext symbol depends on one plaintext symbol.
- **Reusing a keystream** with the same key for two plaintexts is catastrophic: XORing the two ciphertexts cancels the keystream and reveals the XOR of the plaintexts.
- **Synchronous** stream ciphers generate the keystream independently of the data; **self-synchronising** ones derive it from previous ciphertext. Examples: RC4 (now considered weak), ChaCha20.

**Block ciphers** encrypt fixed-size blocks (64 or 128 bits), padding the last block if needed, through multiple rounds of substitution and permutation.

- High **diffusion** — one plaintext bit affects many ciphertext bits. Block ciphers implement Shannon's **confusion** (a complex ciphertext-key relationship) and **diffusion** (plaintext statistics spread throughout the ciphertext).
- Generally considered more robust than stream ciphers.
- Examples: DES (outdated), 3DES, AES (the current standard).

Block ciphers typically protect data at rest and in protocols; stream ciphers suit real-time or continuous data.

## Block cipher modes of operation

A block cipher encrypts exactly one block. A **mode of operation** defines how to apply it repeatedly to a longer message. Most modes need a unique **initialisation vector** per encryption so that encrypting the same plaintext twice produces different ciphertext.

| Mode | Mechanism | Notes |
| --- | --- | --- |
| **ECB** | Each block encrypted independently | Identical plaintext blocks give identical ciphertext — leaks patterns (the "ECB penguin"). Avoid. |
| **CBC** | XOR each plaintext block with the previous ciphertext block; IV for the first | Chains blocks; a ciphertext error garbles that block and flips bits in the next. Encryption sequential, decryption parallelisable. |
| **CFB** | Encrypt the previous ciphertext, XOR with plaintext | Turns the block cipher into a self-synchronising stream cipher; supports sub-block units. |
| **OFB** | Encrypt the IV repeatedly to make a keystream, XOR with plaintext | A synchronous stream cipher; ciphertext bit errors do not propagate. IV must be unique. |
| **CTR** | Encrypt an incrementing counter to make a keystream | Parallelisable, no error propagation; secure with unique counters. Widely used. |

Modes like **GCM** (Galois/Counter Mode) and **CCM** add **authentication** — integrity and origin — on top of confidentiality.

## The one idea to keep

Security starts with two questions, not an algorithm: what property are you protecting (confidentiality, integrity, availability, or non-repudiation), and what can the attacker do (only see ciphertext, or choose plaintexts, or choose ciphertexts). The classical ciphers all fail because they leak structure — letter frequencies survive substitution, and the letters themselves survive transposition. Modern symmetric ciphers fix that with confusion and diffusion, and a mode of operation is what turns a single-block primitive into something you can use on a real message.
