---
title: The Fast Fourier Transform - Computing a DFT in O(n log n)
description: The discrete Fourier transform decomposes a signal into frequencies but costs N^2 to compute directly. The FFT splits the sequence into even and odd samples, recurses, and recombines with roots of unity, dropping the cost to N log N.
date: 2021-02-18
draft: false
slug: /algorithms/FastFourierTransform
tags:
  - Algorithms
  - Divide and Conquer
---

The **discrete Fourier transform** (DFT) takes $N$ samples of a signal and returns $N$ complex numbers, each the amplitude and phase of one frequency. It's behind MP3 and JPEG compression, Wi-Fi and 5G modulation, and MRI reconstruction. Computed from the definition it costs $O(N^2)$, which is hopeless for a few seconds of 44.1 kHz audio. The **fast Fourier transform** is the algorithm — Cooley and Tukey, 1965, though Gauss had it in 1805 — that computes the same DFT in $O(N \log N)$.

## The transform

For a sequence $x_0, \ldots, x_{N-1}$, the DFT is

$$X_k = \sum_{n=0}^{N-1} x_n \, e^{-i 2\pi k n / N}, \qquad k = 0, \ldots, N-1.$$

Direct evaluation is $N$ terms for each of $N$ outputs: $O(N^2)$.

## Splitting even and odd

Assume $N$ is a power of 2. Separate the sum into even-indexed and odd-indexed terms:

$$X_k = \sum_{m=0}^{N/2-1} x_{2m}\, e^{-i 2\pi k (2m)/N} + \sum_{m=0}^{N/2-1} x_{2m+1}\, e^{-i 2\pi k (2m+1)/N}.$$

In the first sum $e^{-i 2\pi k (2m)/N} = e^{-i 2\pi k m /(N/2)}$, which is exactly the DFT basis for an $N/2$-point transform. Factor $e^{-i 2\pi k/N}$ out of the second sum and the same thing happens. So

$$X_k = E_k + W_N^k \, O_k,$$

where $E$ is the DFT of the even samples, $O$ the DFT of the odd samples, and $W_N^k = e^{-i 2\pi k/N}$ is the **twiddle factor**. Because $E$ and $O$ have period $N/2$ and $W_N^{k + N/2} = -W_N^k$, the upper half comes free:

$$X_{k + N/2} = E_k - W_N^k \, O_k, \qquad k = 0, \ldots, N/2 - 1.$$

One $N$-point DFT becomes two $N/2$-point DFTs plus $N/2$ multiply-add pairs. The base case: the DFT of a single sample is itself.

## The code

```python
import cmath

def fft(x, invert=False):
    n = len(x)
    if n == 1:
        return list(x)
    even = fft(x[0::2], invert)
    odd = fft(x[1::2], invert)
    ang = (2j if invert else -2j) * cmath.pi / n
    out = [0] * n
    for k in range(n // 2):
        t = cmath.exp(ang * k) * odd[k]
        out[k] = even[k] + t
        out[k + n // 2] = even[k] - t
    return out


def dft(x):                                   # O(n^2), for checking
    n = len(x)
    return [sum(x[m] * cmath.exp(-2j * cmath.pi * k * m / n) for m in range(n))
            for k in range(n)]


signal = [1, 2, 3, 4, 5, 6, 7, 8]
assert all(abs(a - b) < 1e-9 for a, b in zip(fft(signal), dft(signal)))
```

(Passing `invert=True` flips the exponent's sign; divide the result by $N$ for the true inverse.)

## Multiplying polynomials

The classic application. The coefficients of a product polynomial are a **convolution** of the two input coefficient lists: $(a \cdot b)_k = \sum_{i} a_i \, b_{k-i}$. Evaluating that directly is $O(n^2)$ — the schoolbook method. But the DFT turns convolution into pointwise multiplication: transform both coefficient lists, multiply them entry by entry, transform back.

```python
def multiply(a, b):
    need = len(a) + len(b) - 1
    n = 1
    while n < need:
        n <<= 1                                  # pad to a power of two >= the result length
    fa = fft(a + [0] * (n - len(a)))
    fb = fft(b + [0] * (n - len(b)))
    fc = [p * q for p, q in zip(fa, fb)]
    c = fft(fc, invert=True)
    return [round(v.real / n) for v in c[:need]]  # round: integer coeffs, kill FP noise

assert multiply([1, 2], [1, 3, 5]) == [1, 5, 11, 10]   # (1+2x)(1+3x+5x^2)
```

**Big integers** are just polynomials evaluated at the base: write each number as a coefficient list of digits (base $10$, or $10^9$ for speed), `multiply`, then propagate carries. That is the core of Schönhage–Strassen and of Python's own `int` multiplication for large operands.

Two caveats with the complex-number FFT: coefficients around $10^{15}$ or larger lose precision in `double`, and everything is floating point. When the answer must be exact and modular — coefficients taken mod some prime — use the **number-theoretic transform** (NTT): run the identical butterfly, but replace $e^{-2\pi i/N}$ with a primitive $N$-th root of unity in $\mathbb{Z}_p$ (for $p = 998244353$, the element $3$ generates one for every power-of-two $N \mid p-1$). No floats, no rounding.

## Cost

$$T(N) = 2T(N/2) + O(N) = O(N \log N).$$

There are $\log N$ levels of recursion, each doing $O(N)$ work across all its subproblems. This recursive version uses $O(N \log N)$ space; iterative in-place implementations reorder the input by bit-reversed index and run in $O(N)$ space.

## What it unlocks

Beyond spectral analysis, the FFT makes **convolution** cheap: to convolve two length-$N$ sequences, transform both ($O(N \log N)$), multiply pointwise ($O(N)$), and transform back. That's why FFT underlies fast polynomial and big-integer multiplication (Schönhage–Strassen), digital filters, and OFDM in Wi-Fi and LTE.

Once multiplication is $O(N \log N)$, so is a surprising amount else: **string matching with wildcards** (encode characters as roots of unity, and a zero in the correlation marks a match position), counting pairs with a given sum, "in how many ways can these coin values total $k$", and polynomial division, interpolation, and multipoint evaluation. The competitive-programming treatment — NTT with arbitrary moduli, online convolution, the full [polynomial toolkit](/citadel/cp/operations-on-polynomials) — builds directly on this recurrence, which it shares with [merge sort](/citadel/algorithms/SortingSearching) and [closest pair of points](/citadel/algorithms/ClosestPair): the divide-and-conquer signature of an $O(n \log n)$ algorithm.
