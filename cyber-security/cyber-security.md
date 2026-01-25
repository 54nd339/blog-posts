---
title: An Introduction to Cyber Security - Protecting Your Digital World
description: Exploring fundamental cyber security goals, principles, common cryptographic attacks, and classic ciphers like substitution and transposition, along with stream/block ciphers and their modes of operation.
date: 2023-06-06
draft: false
slug: /pensieve/cyber-security
tags:
  - Cyber Security
  - CS Basics
---

Hey digital adventurers! We've spent a lot of time exploring how computer networks allow us to connect and share information. But with all this data flowing around, a super important question arises: how do we keep it safe? Welcome to the crucial world of **Computer Security**! ️
In this post, we'll journey to understand the basics of protecting digital information. We'll look at the main goals of security, the evolution of secret-keeping, some ways attackers try to break these protections, and the fundamental building blocks of encryption that help keep our secrets secret.

---
## A Brief History and Taxonomy of Keeping Secrets

The desire to send messages securely is ancient. Early methods, known as **classic cryptography**, involved clever but often simple techniques like physically rearranging letters or substituting them. As technology advanced, particularly with the advent of computers, cryptography evolved into a complex mathematical science. **Modern cryptography** relies on rigorously analyzed algorithms and computational hardness to provide security.

Cryptography itself can be broadly categorized:
* **Symmetric Key Cryptography:** Uses a single, shared secret key for both encryption and decryption.
* **Asymmetric Key (Public Key) Cryptography:** Uses a pair of keys – a public key for encryption/signature verification and a private key for decryption/signature creation.
* **Cryptographic Hash Functions:** Produce a fixed-size "fingerprint" of data, primarily used for integrity checks and in digital signatures.

We'll touch upon aspects of these throughout our security discussions!

---
## Security Goals and Principles: The CIA Triad and Beyond

When we talk about information security, we often refer to three core objectives, famously known as the **CIA Triad**:

* **Confidentiality:** This means ensuring that information is accessible only to those authorized to have access. Think of it as keeping secrets. We use techniques like encryption and access controls to enforce confidentiality.
* **Integrity:** This principle ensures that information remains accurate, complete, and consistent throughout its entire lifecycle. It's about making sure data hasn't been tampered with or altered by unauthorized users. Hashing algorithms are often used to verify data integrity.
* **Availability:** This ensures that information and resources are accessible to authorized users when they need them. This means preventing disruptions like denial-of-service attacks or system failures.

Beyond the CIA Triad, other important security principles and goals include:

* **Authentication:** Verifying the identity of users or systems. This ensures you are who you say you are. Multi-factor authentication (MFA) strengthens this.
* **Non-Repudiation:** Ensuring that a sender cannot deny sending a message, and a receiver cannot deny receiving one. Digital signatures are a key technology here.
* **Access Control:** Determining who is allowed to access specific data or resources, and what they are allowed to do (e.g., read, write, delete). This often involves role-based access control (RBAC).
* **Data Protection:** Implementing practices to safeguard sensitive data, including encryption, secure storage, and adherence to data protection laws and regulations.

These goals are achieved through various practices like application security, infrastructure security, cryptography, incident response, and vulnerability management.

---
## Cryptographic Attacks: Testing the Defenses

Cryptography aims to protect information by transforming it into secret codes (encryption). However, attackers constantly try to find weaknesses in these systems. A **cryptographic attack** is an attempt to circumvent the security of a cryptographic system, also known as cryptanalysis.

Here are some broad categories and common types of attacks:

* **Ciphertext-Only Attack:** The attacker only has access to a collection of ciphertext (encrypted messages). They try to deduce the plaintext or the key from this. While less effective than other types, it can work against weak ciphers.
* **Known-Plaintext Attack (KPA):** The attacker possesses some plaintext and its corresponding ciphertext. This knowledge can help deduce the encryption key, especially for weaker algorithms.
* **Chosen-Plaintext Attack (CPA):** The attacker can choose arbitrary plaintext messages to be encrypted and obtain the corresponding ciphertexts. This gives them more power to analyze the encryption process and potentially find the key.
* **Chosen-Ciphertext Attack (CCA):** The attacker can choose arbitrary ciphertexts, have them decrypted, and obtain the corresponding plaintexts. This is a very powerful attack model that can help reveal the decryption key or other system details.
* **Brute-Force Attack:** The attacker tries every possible key until the correct one is found. The feasibility of this depends on the key size; for an 8-bit key, there are $2^8 = 256$ possibilities, but modern keys are much longer, making simple brute force computationally infeasible.
* **Frequency Analysis:** Particularly effective against simple substitution ciphers. It involves analyzing the frequency of letters or symbols in the ciphertext and comparing them to the known frequencies of letters in the plaintext language (e.g., 'E' is the most common letter in English).
* **Man-in-the-Middle (MitM) Attack:** The attacker intercepts communication between two parties and can alter the data without their knowledge.
* **Side-Channel Attacks:** These attacks exploit information gained from the physical implementation of a cryptosystem, rather than theoretical weaknesses in the algorithms. Examples include analyzing power consumption or electromagnetic radiation.
* **Denial of Service (DoS) Attacks against Cryptosystems:** An attacker might flood an encryption algorithm with data to slow it down or crash it, disrupting availability.

Attackers can be **passive** (eavesdropping to gather information without altering communication) or **active** (modifying data or communication).

---
## Classic Ciphers: The Roots of Secrecy

Let's look at some of the foundational methods of encryption.

### Substitution Ciphers: Swapping Letters
One of the oldest and simplest methods of encryption is the **substitution cipher**. In this method, units of plaintext (usually single letters) are replaced with ciphertext units according to a defined system or key. The order of the units remains the same, but the units themselves are changed.

* **Caesar Cipher (Shift Cipher):** A very basic type where each letter in the plaintext is shifted a certain fixed number of places down or up the alphabet. For example, with a shift of 3, 'A' becomes 'D', 'B' becomes 'E', and so on. 'W' would become 'Z'.
    * **Key:** The number of positions to shift.
    * **Weakness:** There are only 25 possible keys (for the English alphabet), making it very easy to break with a brute-force attack (trying all possible shifts). It's also vulnerable to frequency analysis.
* **Monoalphabetic Substitution (Simple Substitution):** Instead of a fixed shift, the ciphertext alphabet is a jumbled or "mixed" version of the plaintext alphabet. Each plaintext letter maps to a unique ciphertext letter. A keyword can be used to create this mixed alphabet: write the keyword (removing repeated letters), then list the remaining alphabet letters in order.
    * **Key:** The mapping between the plaintext and ciphertext alphabets (e.g., the 26-letter permutation).
    * **Strength:** The number of possible keys is huge (26!, which is about $4 \times 10^{26}$), making brute-force attacks infeasible.
    * **Weakness:** Still vulnerable to frequency analysis because letter frequencies are preserved (e.g., if 'X' is the most common letter in the ciphertext, it likely corresponds to 'E' in the plaintext).

Often, ciphertext from substitution ciphers is written in blocks of fixed length, omitting spaces and punctuation, to help disguise word boundaries.

### Transposition Ciphers: Rearranging the Order
Unlike substitution ciphers that change the letters, **transposition ciphers** (also known as permutation ciphers) keep the original plaintext letters but rearrange their order according to a specific system or key.

* **Rail Fence Cipher:** A simple transposition where plaintext is written downwards and diagonally on successive "rails" of an imaginary fence, then read off in rows.
    * Example: Plaintext "HELLO WORLD" with 2 rails:
        H L O O L
        E L W R D
        Ciphertext: HLOOLELWRD
* **Columnar Transposition:** The message is written out in rows of a fixed length (often determined by the length of a keyword). The ciphertext is then read out column by column, with the columns themselves being read in an order determined by the alphabetical order of the letters in the keyword.
    * Example: Plaintext "WE ARE DISCOVERED FLEE AT ONCE" with keyword "ZEBRA" (column order based on keyword: 5 2 1 3 4)
        W E A R E
        D I S C O
        V E R E D
        F L E E A
        T O N C E
        Reading out in column order (A's column, then B's, then E's, etc., based on ZEBRA -> EABRZ order for reading columns): ASRNE IELOE RCECW DVFTE ODA
        Ciphertext: ASRNEIELOERCECW DVFTEODA
* **Scytale:** An ancient Greek mechanical system. A ribbon was wrapped around a cylinder, and the message was written along the length of the cylinder. When unwrapped, the letters were rearranged. It could only be decrypted by wrapping the ribbon around a cylinder of the same diameter.

Transposition ciphers preserve the original letter frequencies of the plaintext, which can be a clue for cryptanalysis. The main technique for breaking them is **anagramming** – rearranging the ciphertext letters to form meaningful words.

---
## Information Hiding: Secrets Within Secrets

Beyond encrypting data to make it unreadable, sometimes the goal is to hide the very existence of a message or information. This is the realm of **information hiding**.

* **Steganography:**
    * The art and science of writing hidden messages in such a way that no one, apart from the sender and intended recipient, suspects the existence of the message.
    * This differs from cryptography, where the existence of the encrypted message is obvious, but its content is obscured.
    * **Techniques:**
        * **Least Significant Bit (LSB) insertion:** Modifying the LSBs of pixel data in an image or audio file. These changes are often imperceptible to humans.
        * Hiding messages in unused spaces of file headers or within the structure of file formats.
        * Using invisible ink, microdots, or character arrangements in physical documents.
    * **Goal:** Covert communication. The main challenge is to embed enough data without creating statistically detectable anomalies in the cover medium (the file used for hiding).

* **Digital Watermarking:**
    * The process of embedding information (a watermark) into a digital signal (image, audio, video) which may be used to verify its authenticity or the identity of its owners.
    * **Types:**
        * **Visible Watermarks:** Like a TV channel logo on screen.
        * **Invisible Watermarks:** Embedded in a way that they are not perceptible during normal viewing/listening but can be detected by specific software.
    * **Purposes:**
        * Copyright protection: Identifying the owner.
        * Copy prevention/control.
        * Tamper detection: An altered file might damage or reveal the watermark.
        * Broadcast monitoring: Tracking when and where content is aired.
    * **Robustness vs. Fragility:** Some watermarks are designed to be robust (survive common signal processing operations like compression or scaling), while others are fragile (designed to be destroyed by any modification, thus indicating tampering).

Information hiding techniques provide an additional layer of security or utility, often used in conjunction with cryptography. While steganography aims for secrecy of existence, watermarking often aims for resilient, detectable information even if the existence of the watermark is known.

---
## Stream and Block Ciphers: Two Flavors of Symmetric Encryption

Modern symmetric encryption algorithms (where the same key is used for encryption and decryption) generally fall into two categories based on how they process plaintext:

### Stream Ciphers
* Encrypt plaintext one symbol (typically a bit or a byte) at a time.
* They work by generating a **keystream** (a sequence of bits) from the secret key (and often an initialization vector or nonce). This keystream is then combined with the plaintext bits, usually using an XOR operation, to produce the ciphertext.
* **Advantages:**
    * **Speed:** Often faster than block ciphers as they are linear in time and constant in space.
    * **Low Error Propagation:** An error in encrypting one symbol typically doesn't affect subsequent symbols.
* **Disadvantages:**
    * **Low Diffusion:** Information from a plaintext symbol is contained within a single ciphertext symbol, which can be a weakness.
    * **Susceptibility to Insertions/Modifications:** If an attacker breaks the algorithm, they might be able to insert spurious text that looks authentic. Keystream reuse with the same key for different plaintexts is catastrophic.
* **Types:**
    * **Synchronous Stream Ciphers:** The keystream is generated independently of the plaintext or ciphertext. Sender and receiver must be synchronized.
    * **Asynchronous (Self-Synchronizing) Stream Ciphers:** The keystream generation depends on previous ciphertext bits. Good for situations where synchronization might be lost.
* Examples: RC4 (though now considered insecure for many uses), ChaCha20.

### Block Ciphers
* Encrypt plaintext by breaking it into fixed-size **blocks** (e.g., 64 bits, 128 bits) and encrypting one block at a time using the secret key.
* If the input data is larger than a block, it's divided into multiple blocks. If the last block is smaller than the cipher's block size, padding is usually required.
* They often involve multiple rounds of substitution and permutation operations.
* **Advantages:**
    * **High Diffusion:** Information from one plaintext bit is spread across many ciphertext bits, making cryptanalysis harder. They often employ principles of "confusion" (making the relationship between ciphertext and key complex) and "diffusion" (spreading plaintext statistics throughout the ciphertext).
    * Generally considered more secure than stream ciphers due to the complex operations on blocks.
* **Disadvantages:**
    * Can be slower than stream ciphers for encrypting large amounts of data, though modern hardware optimizations (like for AES) can make them very fast.
    * Error propagation can be an issue depending on the mode of operation.
* Examples: DES (Data Encryption Standard - now outdated), 3DES (Triple DES), AES (Advanced Encryption Standard - the current standard).

Block ciphers are typically used to encrypt data at rest (like files, databases) and in communication protocols, while stream ciphers are often favored for real-time communications or when data comes in a continuous flow.

---
## Algorithm Modes (Block Cipher Modes of Operation)

A block cipher encrypts only a single fixed-length block of data. To securely encrypt messages larger than the block size, or to provide other services like authentication, a **mode of operation** is needed. These modes define how to repeatedly apply the block cipher's single-block operation. Most modes require a unique binary sequence called an **Initialization Vector (IV)** for each encryption operation to ensure that encrypting the same plaintext multiple times with the same key produces different ciphertexts.

Here are some common confidentiality modes specified by NIST (SP 800-38A):

* **Electronic Codebook (ECB) Mode:**
    * The simplest mode. Each block of plaintext is encrypted independently using the same key.
    * **Problem:** Identical plaintext blocks encrypt to identical ciphertext blocks. This leaks information about patterns in the plaintext, making it insecure for most applications. (Imagine an image of a penguin encrypted with ECB – you can still see the outline!)
* **Cipher Block Chaining (CBC) Mode:**
    * Before encrypting a plaintext block, it is XORed with the *previous* ciphertext block. An IV is used to XOR with the first plaintext block.
    * This "chains" the blocks together, so identical plaintext blocks will encrypt to different ciphertext blocks if their preceding ciphertext blocks are different.
    * **Error Propagation:** An error in one ciphertext block affects the decryption of that block (making it garbled) and the corresponding bit errors in the next block (due to XOR).
    * Encryption is sequential, but decryption can be parallelized.
* **Cipher Feedback (CFB) Mode:**
    * Turns a block cipher into a self-synchronizing stream cipher.
    * The previous ciphertext block (or a segment of it) is encrypted, and the output is XORed with the current plaintext block (or segment) to produce the current ciphertext block. An IV is used for the first step.
    * Allows encryption of data units smaller than the block size.
    * **Error Propagation:** An error in a ciphertext block affects the decryption of that block and several subsequent blocks until the error bits are shifted out of the input register to the block cipher.
* **Output Feedback (OFB) Mode:**
    * Turns a block cipher into a synchronous stream cipher.
    * The block cipher encrypts an IV to produce the first block of a keystream. Subsequent keystream blocks are produced by encrypting the *output of the previous encryption operation on the IV/previous keystream block*.
    * The keystream is then XORed with the plaintext blocks.
    * **No Error Propagation in ciphertext:** Bit errors in the ciphertext only affect the corresponding plaintext bits upon decryption; they don't propagate. The keystream is independent of plaintext and ciphertext. IV must be unique.
* **Counter (CTR) Mode:**
    * Also turns a block cipher into a stream cipher.
    * It encrypts successive values of a "counter" (which must be unique for each block encrypted with a given key, often an IV that is incremented).
    * The encrypted counter values form the keystream, which is then XORed with the plaintext blocks.
    * **No Error Propagation** (like OFB). Allows for parallel encryption/decryption. Widely used and considered very secure if implemented correctly with unique counters.

Other modes combine confidentiality with **authentication** (ensuring data integrity and origin), such as GCM (Galois/Counter Mode) and CCM (Counter with CBC-MAC).

---
## Key Takeaways

Securing our digital information is a multifaceted challenge!
* The core goals are **Confidentiality, Integrity, and Availability (CIA Triad)**, supported by principles like authentication and non-repudiation.
* Cryptography has evolved from **classic ciphers** (substitution, transposition) to **modern algorithms** based on strong mathematical principles. It broadly falls into **symmetric**, **asymmetric**, and **hash function** categories.
* **Information Hiding** techniques like steganography and watermarking offer ways to conceal data's existence or embed ownership.
* **Cryptographic attacks** aim to exploit weaknesses in ciphers, protocols, or key management.
* Modern symmetric ciphers are often categorized as **stream ciphers** (encrypting bit by bit, often fast) or **block ciphers** (encrypting fixed-size blocks, generally offering high diffusion).
* **Block cipher modes of operation** (like ECB, CBC, CTR) define how block ciphers are applied to data larger than a single block, with significant implications for security and error propagation.

This is just an introduction, but hopefully, it gives you a good foundation for understanding the fascinating and ever-evolving field of computer security and cryptography!
