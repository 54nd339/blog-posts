---
title: Symmetric Key Cryptography - The Art of Secret Codes & Its Mathematical Heart
description: Delving into the mathematics behind symmetric key crypto (modular arithmetic, linear congruence, GF(2^n) fields) and exploring modern ciphers like DES, AES, Blowfish, and the Diffie-Hellman key exchange.
date: 2023-06-07
draft: false
slug: /pensieve/cyber-security/symmetric-key
tags:
  - Cyber Security
  - CS Basics
---

Greetings, aspiring cryptologists! In our previous overview, we scratched the surface of symmetric key cryptography. Now, prepare for a more profound exploration. We'll dissect the mathematical engines that drive these ciphers and then examine some iconic symmetric algorithms with greater scrutiny, including the elegant dance of key exchange.

### A Glimpse into Modern Crypto's Past

While the desire for secret communication is ancient, relying on manual ciphers like Caesar's, the 20th century revolutionized cryptography. The World Wars saw sophisticated mechanical cipher machines. However, the digital age, beginning with Claude Shannon's foundational work in the 1940s linking cryptography to information theory, and the subsequent development of computers, paved the way for **modern cryptography**. This era brought about algorithmically complex ciphers, a shift from linguistic characteristics to pure mathematics, and public, peer-reviewed designs. The development of standards like DES in the 1970s, and later AES at the turn of the millennium, marked significant milestones in making strong cryptography widely available.

Fasten your seatbelts; we're going deeper into the rabbit hole of cryptographic theory and practice!

---
## The Mathematical Backbone: Precision in Secrecy

The strength of modern symmetric ciphers isn't accidental; it's forged in the robust fires of discrete mathematics. These concepts ensure that operations are well-defined, reversible (with the key), and incredibly difficult to undo without it.

### 1. Modular Arithmetic: The Clockwork of Cryptography
Modular arithmetic, as we've touched upon, confines numbers to a finite range, much like the hours on a clock. This is crucial for creating predictable yet complex transformations.

**Definition:** Two integers $a$ and $b$ are said to be **congruent modulo $n$** if their difference $(a-b)$ is an integer multiple of $n$ (i.e., $n$ divides $(a-b)$). This is written as:
$a \equiv b \pmod n$
The integer $n$ is the **modulus**. The set of integers $\{0, 1, \ldots, n-1\}$ forms a **complete residue system modulo $n$**. Any integer is congruent modulo $n$ to exactly one integer in this set.

**Key Properties and Operations:**
* **Reflexivity:** $a \equiv a \pmod n$
* **Symmetry:** If $a \equiv b \pmod n$, then $b \equiv a \pmod n$.
* **Transitivity:** If $a \equiv b \pmod n$ and $b \equiv c \pmod n$, then $a \equiv c \pmod n$.
These mean congruence modulo $n$ is an equivalence relation.

* **Addition:** If $a \equiv b \pmod n$ and $c \equiv d \pmod n$, then $a+c \equiv b+d \pmod n$.
    So, $( (A \pmod n) + (B \pmod n) ) \pmod n = (A+B) \pmod n$.
* **Subtraction:** $( (A \pmod n) - (B \pmod n) ) \pmod n = (A-B) \pmod n$.
* **Multiplication:** $( (A \pmod n) \times (B \pmod n) ) \pmod n = (A \times B) \pmod n$.
* **Exponentiation (Modular Exponentiation):** Computing $a^k \pmod n$ efficiently is vital. This can be done using the method of repeated squaring (also known as exponentiation by squaring), which is much faster than direct multiplication for large exponents. For example, to compute $3^{13} \pmod{10}$:
    $3^1 \equiv 3 \pmod{10}$
    $3^2 \equiv 9 \pmod{10}$
    $3^4 \equiv 9^2 \equiv 81 \equiv 1 \pmod{10}$
    $3^8 \equiv 1^2 \equiv 1 \pmod{10}$
    $13 = 8 + 4 + 1$. So, $3^{13} = 3^8 \cdot 3^4 \cdot 3^1 \equiv 1 \cdot 1 \cdot 3 \equiv 3 \pmod{10}$.

* **Modular Multiplicative Inverse:** For an integer $a$, its modular multiplicative inverse modulo $n$, denoted $a^{-1}$, is an integer such that $a \cdot a^{-1} \equiv 1 \pmod n$.
    An inverse $a^{-1} \pmod n$ exists if and only if $\text{gcd}(a, n) = 1$ (i.e., $a$ and $n$ are relatively prime or coprime).
    The **Extended Euclidean Algorithm** is used not only to find $\text{gcd}(a,n)$ but also to find integers $x$ and $y$ such that $ax + ny = \text{gcd}(a,n)$. If $\text{gcd}(a,n)=1$, then $ax + ny = 1$. Taking this modulo $n$, we get $ax \equiv 1 \pmod n$, so $x$ (or $x \pmod n$) is the modular inverse of $a$.

### 2. Linear Congruence: Solving Algebraic Equations in Modular Systems
A linear congruence takes the form:
$ax \equiv b \pmod n$
We seek integer solutions for $x$.

* **Existence and Number of Solutions:** Let $d = \text{gcd}(a, n)$.
    * If $d$ does not divide $b$ (denoted $d \nmid b$), then the congruence $ax \equiv b \pmod n$ has no solutions.
    * If $d$ divides $b$ (denoted $d | b$), then the congruence has exactly $d$ incongruent solutions modulo $n$.
* **Solving when $\text{gcd}(a,n)=1$:** If $a$ and $n$ are coprime, then $d=1$. Since $1$ divides any $b$, there is always a unique solution modulo $n$. This solution is given by $x \equiv a^{-1}b \pmod n$, where $a^{-1}$ is the modular multiplicative inverse of $a$ modulo $n$.

These equations are foundational in cryptanalysis and in certain steps of cryptographic algorithms.

### 3. GF($2^n$) Fields (Galois Fields): The Realm of Binary Polynomials
Galois Fields, denoted GF($p^n$) or $\mathbb{F}_{p^n}$, are finite fields with $p^n$ elements, where $p$ is a prime and $n \ge 1$. Cryptography, especially symmetric block ciphers like AES, heavily utilizes **binary fields GF($2^n$)**.

* **Elements:** In GF($2^n$), elements can be represented in several ways:
    1.  As polynomials of degree less than $n$ with coefficients from GF(2) (i.e., 0 or 1).
        Example for GF($2^3$): $\{0, 1, x, x+1, x^2, x^2+1, x^2+x, x^2+x+1\}$.
    2.  As $n$-bit binary vectors, where each bit corresponds to a coefficient of the polynomial.
        Example for GF($2^3$): $x^2+1 \leftrightarrow (1 \cdot x^2 + 0 \cdot x + 1 \cdot 1) \leftrightarrow \text{binary string '101'}$.

* **Addition (+):** Polynomial addition is done by adding corresponding coefficients modulo 2. This is equivalent to a bitwise XOR operation on their binary vector representations.
    Example in GF($2^3$): $(x^2+x) + (x+1)$
    $= (1\cdot x^2 + 1\cdot x + 0\cdot 1) + (0\cdot x^2 + 1\cdot x + 1\cdot 1)$
    $= (1+0)x^2 + (1+1)x + (0+1)1$
    $= 1x^2 + 0x + 1\cdot1 = x^2+1$.
    Binary: `110` XOR `011` = `101`.
    Note that in GF(2), $1+1=0$, so subtraction is the same as addition (XOR).

* **Multiplication ($\cdot$):**
    1.  Multiply the polynomials as usual.
    2.  Take the result modulo an **irreducible polynomial** $m(x)$ of degree $n$ over GF(2). An irreducible polynomial is one that cannot be factored into polynomials of smaller positive degrees with coefficients in GF(2). This reduction ensures the result is also a polynomial of degree less than $n$ and thus remains an element of GF($2^n$).
    Example in GF($2^3$) using the irreducible polynomial $m(x) = x^3+x+1$:
    Let $A(x) = x^2+1$ (`101`) and $B(x) = x+1$ (`011`).
    $A(x) \cdot B(x) = (x^2+1)(x+1) = x^3+x^2+x+1$.
    Now, we reduce $x^3+x^2+x+1$ modulo $x^3+x+1$:
    Since $x^3 \equiv x+1 \pmod{x^3+x+1}$ (because $x^3+x+1 \equiv 0 \Rightarrow x^3 \equiv -(x+1) \equiv x+1$ as coefficients are mod 2).
    So, $x^3+x^2+x+1 \equiv (x+1) + x^2+x+1 \equiv x^2 + (x+x) + (1+1) \equiv x^2+0x+0 \equiv x^2$.
    Thus, $(x^2+1)(x+1) \equiv x^2 \pmod{x^3+x+1}$ in GF($2^3$).
    Binary: `101` $\cdot$ `011` = `100`.

* **Multiplicative Inverse:** Every non-zero element $a$ in GF($2^n$) has a unique multiplicative inverse $a^{-1}$ such that $a \cdot a^{-1} = 1$. This can be found using an adaptation of the Extended Euclidean Algorithm for polynomials.

GF($2^n$) fields, particularly GF($2^8$) (elements are bytes), are critical in AES for the `SubBytes` (S-box) and `MixColumns` steps, providing essential non-linearity and diffusion.

---
## Symmetric Key Cryptography in Action: A Detailed Look

Recall that symmetric key algorithms use the same secret key for both encryption and decryption.

### Modern Block Ciphers: Building Secure Boxes
Modern block ciphers are designed to withstand a barrage of cryptanalytic attacks. They achieve this primarily through Shannon's principles of **confusion** and **diffusion**, implemented iteratively over multiple rounds.

* **Confusion:** Aims to make the relationship between the key and the ciphertext as complex and non-linear as possible. If a single bit of the key is changed, a significant portion of the ciphertext should change in a pseudorandom way. S-boxes (Substitution boxes) are the primary tools for achieving confusion.
* **Diffusion:** Aims to spread the influence of individual plaintext bits over many ciphertext bits. This means a change in one plaintext bit should result in changes to roughly half the ciphertext bits. Permutation layers (P-boxes) or other linear mixing layers are used for diffusion.

**Common Iterative Structures:**
* **Feistel Network:**
    * The plaintext block is split into two halves, Left ($L_0$) and Right ($R_0$).
    * For each round $i$ (from 1 to $r$):
        $L_i = R_{i-1}$
        $R_i = L_{i-1} \oplus F(R_{i-1}, K_i)$
        where $F$ is a round function and $K_i$ is the subkey for round $i$.
    * After $r$ rounds, the halves might be swapped one last time.
    * **Key Advantage:** The decryption process is identical to encryption, using the same $F$ function but with the subkeys $K_i$ applied in reverse order ($K_r, K_{r-1}, \ldots, K_1$). This simplifies implementation.
    * Example: DES.

* **Substitution-Permutation Network (SPN):**
    * The entire block is processed in each round.
    * Each round typically consists of:
        1.  **Key Mixing:** Combining the block with a round key (often via XOR).
        2.  **Substitution:** Applying S-boxes to provide confusion (non-linear transformation).
        3.  **Permutation:** Transposing bits or bytes to provide diffusion across the block.
    * Example: AES.

### Modern Stream Ciphers: Continuous Encryption
Stream ciphers generate a pseudorandom **keystream** (a sequence of bits $s_1, s_2, \ldots$) from the secret key $K$ (and usually a nonce/IV). This keystream is then XORed with the plaintext bits ($p_1, p_2, \ldots$) to produce ciphertext bits ($c_1, c_2, \ldots$):
$c_i = p_i \oplus s_i$
Decryption is the same XOR operation: $p_i = c_i \oplus s_i$.

* **Keystream Generator Properties:**
    * The keystream must be computationally infeasible to predict without the key.
    * It should have a long period (before it starts repeating).
    * It should have good statistical randomness properties.
    * **Crucially, a keystream should never be reused for different plaintexts with the same key.** If $c_1 = p_1 \oplus s$ and $c_2 = p_2 \oplus s$, then $c_1 \oplus c_2 = p_1 \oplus p_2$. An attacker with $c_1$ and $c_2$ can derive $p_1 \oplus p_2$, which leaks significant information about the plaintexts.
* **Common Components:**
    * **Linear Feedback Shift Registers (LFSRs):** Simple hardware components that can produce sequences with long periods. An LFSR's output is a linear function of its previous state. Multiple LFSRs are often combined in non-linear ways to produce stronger keystreams.
    * Many modern designs are based on ARX principles (Addition, Rotation, XOR) or specific mathematical structures.

### Diffie-Hellman Key Exchange Algorithm: The Genesis of Shared Secrets
While not a symmetric encryption algorithm itself, Diffie-Hellman is paramount because it solves the problem of establishing a shared secret key for symmetric encryption over an insecure channel.

Let $p$ be a large prime number and $g$ be a generator (a primitive root modulo $p$). These are public parameters.
1.  Alice chooses a secret integer $a$ (her private key), $1 \le a < p-1$.
    She computes her public key $A = g^a \pmod p$.
    Alice sends $A$ to Bob.
2.  Bob chooses a secret integer $b$ (his private key), $1 \le b < p-1$.
    He computes his public key $B = g^b \pmod p$.
    Bob sends $B$ to Alice.
3.  Alice computes the shared secret $S = B^a \pmod p$.
    Since $B = g^b \pmod p$, Alice computes $S = (g^b)^a \pmod p = g^{ba} \pmod p$.
4.  Bob computes the shared secret $S = A^b \pmod p$.
    Since $A = g^a \pmod p$, Bob computes $S = (g^a)^b \pmod p = g^{ab} \pmod p$.

Now, Alice and Bob both possess the same secret $S = g^{ab} \pmod p$, which they can use as a key for a symmetric cipher.
The security relies on the **Discrete Logarithm Problem (DLP)**: Given $p, g,$ and $A = g^a \pmod p$, it is computationally infeasible to find the secret $a$ if $p$ is sufficiently large and well-chosen.

### Data Encryption Standard (DES)
* **Structure:** 64-bit block cipher using a 16-round Feistel network.
* **Key:** 64-bit key, but only 56 bits are effective (8 are parity bits). Subkeys ($K_i$) of 48 bits are generated for each round via a key schedule involving permutations and left shifts.
* **Encryption Process:**
    1.  Initial Permutation (IP) of the 64-bit plaintext block.
    2.  The block is split into $L_0$ (32 bits) and $R_0$ (32 bits).
    3.  For 16 rounds ($i=1 \ldots 16$):
        $L_i = R_{i-1}$
        $R_i = L_{i-1} \oplus F(R_{i-1}, K_i)$
        The round function $F(R_{i-1}, K_i)$ involves:
        * **Expansion (E):** $R_{i-1}$ (32 bits) is expanded to 48 bits.
        * **Key Mixing:** XORed with the 48-bit round key $K_i$.
        * **Substitution (S-Boxes):** The 48 bits are divided into eight 6-bit chunks. Each chunk goes into one of eight different S-boxes, each outputting 4 bits. S-boxes are the core non-linear component.
        * **Permutation (P):** The resulting 32 bits are permuted.
    4.  After 16 rounds, $L_{16}$ and $R_{16}$ are swapped (this is a final swap, so $R_{16}$ becomes the left half of the output and $L_{16}$ the right).
    5.  Final Permutation (FP, which is IP$^{-1}$) is applied.
* **Weaknesses:**
    * **Key Size (56 bits):** Vulnerable to brute-force attacks (demonstrated in the late 1990s).
    * Susceptible to differential cryptanalysis and linear cryptanalysis, though 16 rounds provide reasonable resistance against these for the given key size.
* **Triple DES (3DES):** To extend its life, 3DES applies DES three times: $C = E_{K3}(D_{K2}(E_{K1}(P)))$. This increases the effective key length. It's much slower but offered better security than single DES.

### Blowfish
* **Structure:** 64-bit block cipher, 16-round Feistel network.
* **Key:** Variable length from 32 bits to 448 bits.
* **Key Expansion:** A complex and relatively slow key expansion process generates round subkeys (eighteen 32-bit subkeys $P_1, \ldots, P_{18}$) and fills four $256 \times 32$-bit S-boxes (each S-box has 256 32-bit entries).
    * Initially, P-array and S-boxes are initialized with digits of $\pi$.
    * The P-array is XORed with the key bits.
    * A block of all zeros is encrypted with the current P-array and S-boxes. The output replaces $P_1, P_2$. This output is then encrypted again to replace $P_3, P_4$, and so on, for all P-array entries and then all S-box entries. This makes the S-boxes key-dependent.
* **Round Function $F$:**
    * Input 32-bit half-block $R_{i-1}$ is divided into four 8-bit quarters ($a,b,c,d$).
    * $F(R_{i-1}) = ( (S_1[a] + S_2[b] \pmod{2^{32}}) \oplus S_3[c] ) + S_4[d] \pmod{2^{32}}$. (Addition is mod $2^{32}$).
* **Security:** No effective cryptanalysis is known for 16-round Blowfish. Its security relies on the large, key-dependent S-boxes and the complex key schedule. The 64-bit block size is its main modern concern for encrypting large volumes of data under a single key (due to birthday attacks related to block collisions).

### Advanced Encryption Standard (AES) - Rijndael
* **Structure:** SPN (Substitution-Permutation Network), not Feistel. Operates on 128-bit blocks (plaintext and ciphertext).
* **Key Sizes & Rounds:**
    * AES-128: 128-bit key, 10 rounds.
    * AES-192: 192-bit key, 12 rounds.
    * AES-256: 256-bit key, 14 rounds.
* **The State:** Data block is represented as a $4 \times 4$ matrix of bytes (the state).
* **Round Structure (except final round which omits MixColumns):**
    1.  **`SubBytes` (Substitution):** Each byte of the state is independently replaced using a fixed 16x16 S-box. This S-box is constructed by:
        * Taking the multiplicative inverse in GF($2^8$) (with $00 \rightarrow 00$). The irreducible polynomial for AES's GF($2^8$) is $m(x) = x^8+x^4+x^3+x+1$.
        * Applying an affine transformation over GF(2): $b'_i = b_i \oplus b_{(i+4)\%8} \oplus b_{(i+5)\%8} \oplus b_{(i+6)\%8} \oplus b_{(i+7)\%8} \oplus c_i$, where $c_i$ is the $i$-th bit of the byte $63_{16}$ or `01100011`. This transformation provides non-linearity and resists simple algebraic attacks.
    2.  **`ShiftRows` (Permutation):** Bytes in each row of the state are cyclically shifted to the left. Row 0 is not shifted, Row 1 by 1 byte, Row 2 by 2 bytes, Row 3 by 3 bytes. This provides diffusion across bytes in a row.
    3.  **`MixColumns` (Mixing/Diffusion):** Each column of the state is treated as a 4-term polynomial over GF($2^8$) and multiplied modulo $x^4+1$ by a fixed polynomial $a(x) = \{03\}x^3 + \{01\}x^2 + \{01\}x + \{02\}$ (where coefficients like $\{02\}, \{03\}$ are specific bytes representing elements in GF($2^8$)). This operation ensures that all bytes in an output column depend on all four bytes of the input column, providing significant diffusion within columns. This step is omitted in the final round.
    4.  **`AddRoundKey`:** The 128-bit round key (derived from the main key via a key schedule) is XORed with the current state.
* **Key Schedule:** Generates the required round keys from the initial cipher key. It involves word rotations, S-box substitutions (using the same S-box as `SubBytes`), and XORing with round constants (Rcon).
* **Security:** AES is the current global standard. It's considered highly secure against known forms of cryptanalysis when implemented correctly. Its strength comes from its multiple rounds, well-understood mathematical properties of its components, and resistance to both linear and differential cryptanalysis.

---
## Final Musings on Symmetric Security

The journey from simple modular arithmetic to complex ciphers like AES demonstrates a beautiful interplay of mathematical theory and practical engineering. Symmetric key cryptography provides the workhorses for much of our daily secure communication, from encrypted files to secure web sessions (where it's used after a key is exchanged).

Understanding these deeper details reveals why certain operations are chosen and how they contribute to the overall security goal: making it computationally infeasible for an attacker to decipher information without the secret key.
