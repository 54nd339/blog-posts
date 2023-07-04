---
title: Information Theory and Coding - Entropy, Channels, and Compression
description: Entropy sets a hard floor on how far data can be compressed, and channel capacity sets a ceiling on how fast it can be sent reliably over noise. Shannon's source and channel coding theorems, Huffman and arithmetic coding, error-correcting block and convolutional codes, and how JPEG and MPEG apply all of it.
date: 2023-07-04
draft: false
slug: /miscs/information-theory
tags:
  - Information Theory
  - Coding Theory
  - Compression
---

A file streams to your phone intact, and a large one arrives smaller than it started. Both rest on **information theory**, founded by Claude Shannon: a measure of how much information a source really contains, a limit on how far you can compress it, and a limit on how fast you can send it reliably through noise. This post covers entropy and the source coding theorem, channels and the channel coding theorem, error-correcting codes, and how JPEG and MPEG put the theory to work.

## Entropy: measuring uncertainty

Before a fair coin flip there is uncertainty; seeing the result gives you **information** and removes that uncertainty. The more uncertain the event, the more information its outcome carries.

**Entropy** $H$ quantifies the *average* uncertainty of a source. A fair die has high entropy — many roughly equal possibilities. A coin that almost always lands heads has low entropy — the outcome is nearly known in advance. For a message, $H$ is the minimum average number of bits needed to represent the information it carries.

Put a number on it. A single outcome of probability $p$ carries $-\log_2 p$ bits of **surprise** — certain events ($p = 1$) carry none, rare events carry a lot. Entropy is the surprise averaged over all outcomes of a source $X$ with symbol probabilities $p_1, \dots, p_n$:

$$ H(X) = -\sum_{i=1}^{n} p_i \log_2 p_i $$

The $\log_2$ makes the unit the **bit**; using $\ln$ gives *nats*, $\log_{10}$ gives *dits*, and only the scale changes. $H$ is largest, $\log_2 n$, when every symbol is equally likely, and drops to $0$ when one symbol has all the probability.

Worked example: a fair coin has $H = -\tfrac12\log_2\tfrac12 - \tfrac12\log_2\tfrac12 = 1$ bit per flip. A biased coin with $p(\text{heads}) = 0.9$ has $H = -0.9\log_2 0.9 - 0.1\log_2 0.1 \approx 0.47$ bits — so a long run of flips from the biased coin can be stored in under half the space, and that factor is exactly what a compressor is chasing. A source with four equally likely symbols has $H = \log_2 4 = 2$ bits per symbol; skew those probabilities to $\tfrac12, \tfrac14, \tfrac18, \tfrac18$ and $H$ falls to $1.75$ bits.

### The source coding theorem

Shannon's **source coding theorem**: you can compress data from a source down to its entropy $H$ bits per symbol without losing information (on average), and you *cannot* reliably compress below $H$. That is the ultimate limit for lossless compression.

### Reaching it: Huffman and Shannon–Fano

The idea of **source coding**: give frequent symbols short codes, rare symbols long codes.

- **Huffman coding** — builds an optimal **prefix code** (no codeword is a prefix of another, so decoding is unambiguous) by repeatedly merging the two least probable symbols into a subtree until one remains. See [Huffman coding](/citadel/algorithms/HuffmanCoding).
- **Shannon–Fano coding** — recursively splits the symbol set into two subsets of near-equal probability, assigning `0` to one and `1` to the other. Intuitive, but not always optimal like Huffman.

## Channels and capacity

To *transmit* rather than compress, model the link. A **discrete memoryless channel (DMC)** has finite input and output alphabets (**discrete**), and its output at each instant depends only on that instant's input (**memoryless**).

Every channel has a **capacity** $C$ — the maximum rate at which information can pass reliably, like the width of a pipe.

### The channel coding theorem

Shannon again: for a noisy channel of capacity $C$, if your transmission rate $R < C$, coding schemes exist that make the error probability arbitrarily small. If $R > C$, arbitrarily reliable communication is impossible.

For an additive-white-Gaussian-noise channel, the **Shannon–Hartley theorem** gives the capacity:
$$C = B \log_2\!\left(1 + \frac{S}{N}\right)$$
where $C$ is in bits per second, $B$ is the bandwidth in hertz, and $S/N$ is the signal-to-noise ratio. Bandwidth and signal quality together bound the achievable speed.

## Data and voice coding

Voice and audio have structure you can exploit beyond generic compression.

- **DPCM** — encode the *difference* between a sample and a predicted value (often the previous sample). Consecutive audio samples are similar, so differences are small and need fewer bits.
- **ADPCM** — DPCM with an adaptive quantiser step size or prediction filter: larger steps for fast-changing signals, smaller for smooth ones.
- **Adaptive sub-band coding** — split the signal into frequency bands, code each separately (often with ADPCM), and allocate bits by perceptual importance.
- **Delta modulation** — DPCM with a 1-bit quantiser: one bit per sample says whether the signal rose or fell, and the approximation steps up or down by a fixed $\delta$. Suffers **slope overload** when the signal changes too fast and **granular noise** when it is flat.
- **Adaptive delta modulation** — makes $\delta$ adaptive: it grows when the output bits stay the same (slope overload) and shrinks when they alternate.
- **Vocoders and LPC** — at very low bit rates, do not preserve the waveform. Analyse the speech, extract parameters — pitch, energy, vocal-tract shape — transmit those, and resynthesise at the receiver. **Linear predictive coding** models the vocal tract as a filter and predicts each sample from a linear combination of past ones; the filter coefficients, pitch, and energy are what get sent — a recipe for the sound rather than the sound.

## Error control coding

Transmission and storage introduce errors — flipped bits from noise or media defects. **Error control coding** adds structured redundancy so errors can be detected and corrected. This underlies the [data link layer](/citadel/computer-networks/datalink-layer)'s integrity checks.

### Linear block codes

Take $k$ message bits, add $r = n - k$ parity bits, produce an $n$-bit **codeword**. **Linear**: the sum of any two codewords is a codeword, which makes them tractable to analyse.

**Syndrome decoding.** Multiply the received vector by the parity-check matrix $H^{\mathsf T}$. A zero result (the **syndrome**) means no detectable error; a non-zero syndrome identifies the most likely error pattern, which is then corrected.

**Minimum distance** $d_{\min}$ — the smallest Hamming distance between any two distinct codewords. To *detect* up to $t$ errors you need $d_{\min} \ge t + 1$; to *correct* up to $t$ you need $d_{\min} \ge 2t + 1$. Larger $d_{\min}$ means more capability at the cost of more parity bits.

### Cyclic codes

A subclass of linear block codes with an extra property: any cyclic shift of a codeword is also a codeword. This allows efficient encoding and decoding with shift registers.

- **Generator polynomial** $g(x)$ defines the code: encode by multiplying the message polynomial by $x^{n-k}$ and taking the remainder mod $g(x)$ as parity.
- **Parity-check polynomial** $h(x)$, related by $g(x)\,h(x) = x^n - 1$.
- The encoder is a linear-feedback shift register wired by $g(x)$; the syndrome of a received $r(x)$ is $r(x) \bmod g(x)$.
- **Cyclic redundancy checks (CRCs)** — the error-detection codes in Ethernet and Wi-Fi — are cyclic codes.

### Convolutional codes

Rather than fixed blocks, convolutional codes encode a continuous stream: each set of $n$ output bits depends on the current $k$ input bits *and* a number of preceding input bits (the **constraint length**) — codes with memory. The **Viterbi algorithm** is the standard decoder. Used in GSM and satellite links.

## Compression standards

- **Lossless** — the original is perfectly recoverable. Huffman, LZW (in GIF), run-length encoding. For text, code, medical images.
- **Lossy** — discards information judged imperceptible, for much higher ratios. JPEG, MPEG, MP3.

### Text compression

- **Static Huffman** — fixed code table from a probability model of the whole text.
- **Adaptive (dynamic) Huffman** — builds the tree as it processes the data, so the table tracks changing symbol frequencies.
- **Arithmetic coding** — represents the entire message as one fraction in $[0, 1)$; higher-probability symbols take larger sub-intervals, using fewer bits. Often beats Huffman.

### Image compression

- **GIF** — LZW, limited to 256 colours per frame; good for logos and simple animation.
- **TIFF** — flexible container, lossless (LZW) or lossy (JPEG inside).
- **CCITT Group 3 / Group 4** — run-length and modified Huffman, tuned for black-and-white scanned documents and faxes.

**JPEG**, for photographic stills ([more detail](/citadel/computer-graphics/compression)):

1. **Colour transform** — RGB to a luminance/chrominance space (YCbCr); the eye is less sensitive to colour detail than brightness.
2. **Downsample** the chrominance channels.
3. **Discrete cosine transform** on each 8×8 block — converts pixels to frequency coefficients, concentrating energy in the low frequencies.
4. **Quantisation** — divide coefficients by a quantisation table and round; high frequencies are quantised coarsely or to zero. This is the lossy step.
5. **Entropy coding** — losslessly compress the quantised coefficients (Huffman or arithmetic).

**MPEG**, for video ([codec detail](/citadel/computer-graphics/audio-video)):

- **I-frames** — coded independently, like a JPEG; random-access points.
- **P-frames** — predicted from an earlier I- or P-frame by **motion compensation**; only residuals and **motion vectors** are coded.
- **B-frames** — predicted from both past and future reference frames; highest compression.

Motion compensation records how blocks moved between frames instead of resending them. MPEG evolved through MPEG-1, MPEG-2, MPEG-4 Part 2, H.264/AVC, and H.265/HEVC, each improving compression or quality.

## The one idea to keep

Two limits bracket every digital communication system. **Entropy** is the floor: you cannot compress a source below $H$ bits per symbol without losing information. **Channel capacity** $C = B\log_2(1 + S/N)$ is the ceiling: below it, coding can make errors as rare as you like; above it, reliable communication is impossible. Source coding removes redundancy to approach the floor; channel coding adds structured redundancy to approach the ceiling.
