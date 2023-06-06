---
title: Multimedia Compression - Redundancy, Codecs, and Standards
description: Why multimedia files must be compressed, the difference between lossless and lossy, the five kinds of redundancy that compression exploits, the core algorithms - RLE, Huffman, arithmetic, LZW, DPCM - and how JPEG, the MPEG family, H.264, and AAC combine them.
date: 2023-06-06
draft: false
slug: /computer-graphics/compression
tags:
  - Computer Graphics
  - Compression
  - Multimedia
---

An uncompressed HD video runs to gigabytes per minute. Streaming it, storing a music library on a phone, or serving images on a web page only works because the data is squeezed first — often to a tenth or a hundredth of its raw size. This post covers how: the two broad strategies, the kinds of redundancy that make squeezing possible, the handful of core algorithms, and how the well-known standards assemble them.

It supports [audio and video](/citadel/computer-graphics/audio-video) and [text and image](/citadel/computer-graphics/text-image).

## Codec

A **codec** is a **coder-decoder**. The **coder** (encoder) applies algorithms to shrink the original data; the **decoder** reverses the process — exactly, for lossless compression, or approximately, for lossy. A codec can be software (a media-player plugin) or hardware (a chip in a camera). Different codecs target different media and strike different balances between compression ratio, quality, and computational cost.

## Lossless versus lossy

**Lossless compression** discards nothing. Decompression reproduces the original bit for bit. It works by removing statistical redundancy, achieves modest ratios, and is required where fidelity is non-negotiable — text, executables, medical images, archival masters. ZIP is the familiar example.

**Lossy compression** achieves far higher ratios by discarding information judged imperceptible — details the eye or ear is least likely to notice. The loss is irreversible and grows as you compress harder. Most multimedia you consume is lossy: JPEG images, MP3 and AAC audio, MPEG and H.264 video. Push it too far and the artifacts show — blocky JPEGs, "swishy" MP3s.

## Five kinds of redundancy

Compression removes redundancy, and there are five kinds worth naming:

- **Coding redundancy** — the symbol codes are not optimally efficient. If every character uses 8 bits but some appear far more often than others, shorter codes for the frequent ones save space. Huffman coding targets this.
- **Inter-pixel (spatial) redundancy** — adjacent pixels in an image tend to be similar, so storing the difference between neighbours costs fewer bits than storing each absolute value.
- **Inter-frame (temporal) redundancy** — consecutive video frames are nearly identical in low-motion scenes, so encoding only the differences between frames is far cheaper than encoding each independently.
- **Psycho-visual redundancy** — the eye is insensitive to certain information, such as high-frequency colour variation. Lossy image and video compression discards it.
- **Psycho-acoustic redundancy** — the ear is insensitive to certain frequencies, especially quiet sounds near louder ones (the masking effect). Lossy audio compression discards those components.

## Lossless techniques

**Run-length encoding (RLE).** Replace a run of identical values with the value and a count: `AAAAABBBCCCCCC` becomes `5A3B6C`. Effective for data with long constant stretches — fax images, flat-colour graphics, intermediate steps in other codecs.

```python
def rle_encode(data):
    out = []
    i = 0
    while i < len(data):
        j = i
        while j < len(data) and data[j] == data[i]:
            j += 1
        out.append((data[i], j - i))
        i = j
    return out

def rle_decode(pairs):
    return "".join(sym * count for sym, count in pairs)

s = "AAAAABBBCCCCCC"
assert rle_decode(rle_encode(s)) == s
assert rle_encode(s) == [("A", 5), ("B", 3), ("C", 6)]
```

**Huffman coding.** A variable-length code that assigns shorter bit strings to more frequent symbols, built from a binary tree of symbol frequencies. Used inside JPEG, MP3, and the ZIP family. The tree construction is worked through in [Huffman coding](/citadel/algorithms/HuffmanCoding).

**Arithmetic coding.** Another entropy coder, usually beating Huffman on ratio. It represents an entire message as a single fraction in $[0, 1)$, narrowing the interval more for frequent symbols so they cost fewer bits. Used in JPEG 2000 and H.264, at higher computational cost.

**Lempel-Ziv-Welch (LZW).** A dictionary method: it builds a table of sequences seen in the data and, when a sequence recurs, emits the table index instead of the sequence. Used in GIF, TIFF, PDF, and the Unix `compress` utility.

The theoretical floor on how far entropy coding can go is set by the **entropy** of the source — see [information theory](/citadel/miscs/information-theory).

## Lossy technique: DPCM

**Differential pulse-code modulation** is predictive coding. Instead of the absolute value of each sample, it encodes the difference between the sample and a predicted value (often the previous sample). Differences are typically small, so they need fewer bits. DPCM appears in telephony audio, image compression, and as a component inside larger lossy codecs.

## Standards

### Image

- **GIF** — LZW (lossless), limited to 256 colours. Suited to flat-colour graphics, logos, and simple animation.
- **JPEG** — primarily lossy. It transforms the image into the frequency domain with the **discrete cosine transform** (DCT), **quantises** the DCT coefficients (the main lossy step, discarding high-frequency detail the eye barely registers), then entropy-codes the result, usually with Huffman. The compression level is adjustable.

### Video

Video compression combines spatial compression within a frame (like JPEG) with temporal compression between frames.

- **H.261** — early videoconferencing over ISDN; introduced macroblocks and motion compensation.
- **H.263** — better quality at lower bitrate; web and videoconferencing.
- **H.264 / AVC (MPEG-4 Part 10)** — highly efficient and near-universal: Blu-ray, streaming, HDTV, cameras.
- **MPEG-1** — Video CD and early digital audio (MP3 is MPEG-1 Audio Layer III); around 1.5 Mbps.
- **MPEG-2** — DVD, digital television, Super Video CD; supports interlaced video and higher bitrates.
- **MPEG-4** — a broad standard with many parts. Part 2 (DivX, Xvid) targeted internet video; Part 10 is H.264. It also covers audio (AAC) and 3D content.
- **MPEG-7** — *not* a compression standard. It is a standard for **describing** multimedia content — metadata about structure, features (colour, texture, motion), and semantics — to make large collections searchable and filterable.

### Audio

- **AMR (adaptive multi-rate)** — speech-optimised, standard in mobile telephony; adapts its bitrate to network conditions.
- **AAC (advanced audio coding)** — lossy, generally better quality than MP3 at the same bitrate. Part of MPEG-4; used by Apple, YouTube, and digital radio.

## The one idea to keep

Compression is redundancy removal, and there are only two kinds of redundancy to remove: statistical (some symbols and patterns repeat, so recode them shorter — always reversible) and perceptual (some detail is below the threshold of human sight or hearing, so drop it — never reversible). Every standard is a specific recipe of transforms and entropy coders built on those two ideas.
