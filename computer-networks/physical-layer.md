---
title: The Physical Layer - Signals, Modulation, and Capacity
description: Bits do not travel on a wire; voltages, light pulses, and radio waves do. How a stream of 1s and 0s becomes a signal that survives the trip, why Nyquist and Shannon put hard ceilings on the bit rate, and how modulation and multiplexing pack more data into the same bandwidth.
date: 2023-04-01
draft: false
slug: /computer-networks/physical-layer
tags:
  - Networking
  - TCP/IP
---

Every layer of the [network stack](/citadel/computer-networks/cn) above this one deals in packets and frames — discrete, digital things. The physical layer is where those bits become something that can actually move through copper, glass, or air: a changing voltage, a pulse of light, a modulated radio wave. And it's where physics imposes limits the upper layers can't argue with — there is a maximum bit rate for a given slice of spectrum, full stop.

This post is how bits become signals, the two theorems that cap the data rate, and the tricks (modulation, multiplexing) that get the most out of what's allowed.

## Signals and what degrades them

A signal is a physical quantity varying over time. Sending data means varying it in a way the receiver can decode back to bits. Three enemies of that:

- **Attenuation** — the signal weakens with distance (and more so at higher frequencies), so it must be amplified or regenerated periodically.
- **Noise** — thermal noise, crosstalk from adjacent wires, electromagnetic interference. The **signal-to-noise ratio** (SNR), usually in decibels, is the key figure of merit.
- **Distortion** — different frequency components travel at slightly different speeds or attenuate differently, smearing pulses into each other (**intersymbol interference**).

**Bandwidth** here means a range of frequencies (Hz) the channel passes — not the "bandwidth" of everyday speech, which is really data rate (bits/s). The distinction matters because the theorems below relate the two.

## The two ceilings

### Nyquist — the noiseless limit

For a channel of bandwidth `B` Hz carrying signals with `M` distinguishable levels, the maximum symbol rate is `2B` symbols/s, so the bit rate is:

$$C = 2B \log_2 M \quad \text{bits/s}$$

More levels per symbol pack more bits — 4 levels = 2 bits/symbol, 16 levels = 4 bits/symbol. The catch: with a fixed transmit power, more levels sit closer together and noise flips them more easily. Nyquist assumes *no* noise, so it says you can go arbitrarily fast by adding levels — which is why you need the second theorem.

### Shannon–Hartley — the noisy limit

With noise, the ceiling is:

$$C = B \log_2\!\left(1 + \frac{S}{N}\right) \quad \text{bits/s}$$

where `S/N` is the linear signal-to-noise ratio. This is a hard physical bound: no coding scheme, however clever, beats it. A 3.1 kHz phone channel with 30 dB SNR (`S/N = 1000`) caps at ~30 kbit/s — which is exactly why dial-up modems plateaued at 56k (using digital tricks on part of the path). Increasing the data rate means widening `B` (more spectrum) or raising SNR (more power, or getting closer). Both cost money or are regulated, which is why spectrum is expensive.

## Line coding

Even on a "digital" link, raw 1s and 0s as high/low voltage have problems: a long run of the same bit gives no transitions for the receiver's clock to lock onto, and a DC bias that some media can't carry. **Line coding** shapes the bit stream:

- **NRZ** (non-return-to-zero) — high = 1, low = 0. Simple, but long runs lose clock sync.
- **NRZI** — a *transition* means 1, no transition means 0 (or vice versa). Encodes data in changes, which helps.
- **Manchester** — every bit has a mid-bit transition (the direction is the data). Self-clocking, but doubles the required bandwidth. Used in classic Ethernet.
- **4B/5B, 8B/10B, 64B/66B** — map each group of data bits to a slightly larger code group chosen to guarantee frequent transitions and DC balance. 8B/10B (Gigabit Ethernet, PCIe) trades 25% overhead for reliable clock recovery; 64B/66B (10G+) cuts that to ~3%.

## Modulation

To send digital data over a channel that carries a **carrier wave** (radio, or a passband link), vary a property of the carrier:

- **ASK** (amplitude-shift keying) — 1 and 0 are two amplitudes. Simple, noise-sensitive.
- **FSK** (frequency-shift keying) — two frequencies. Robust; used in low-rate radio and old modems.
- **PSK** (phase-shift keying) — shift the carrier's phase. **BPSK** = 2 phases (1 bit/symbol); **QPSK** = 4 phases (2 bits/symbol).
- **QAM** (quadrature amplitude modulation) — vary amplitude *and* phase together, giving a **constellation** of points. 16-QAM = 4 bits/symbol, 256-QAM = 8, 1024-QAM (Wi-Fi 6) = 10. Higher-order QAM needs a cleaner channel (points are packed tighter), so a link adapts its constellation to the current SNR — this is why your Wi-Fi rate drops as you walk away from the router.

Note **bit rate vs baud rate**: baud is symbols/second; bit rate is baud × bits-per-symbol. 256-QAM at 1 Mbaud is 8 Mbit/s.

## Multiplexing

Sharing one physical medium among many signals:

- **FDM** (frequency-division) — each signal gets its own frequency band, all sent simultaneously (radio stations, cable TV, the old analog phone trunk).
- **TDM** (time-division) — each signal gets a repeating time slot. **Synchronous** TDM wastes slots when a source is idle; **statistical** TDM allocates slots on demand (this is basically what packet switching does).
- **WDM** (wavelength-division) — FDM for fibre: many colours of light in one strand, each a full channel. DWDM packs dozens of wavelengths, taking a single fibre to terabits per second.
- **OFDM** (orthogonal frequency-division) — split the data across many closely-spaced subcarriers, each carrying a low-rate stream with its own QAM constellation. Overlapping-but-orthogonal spacing is bandwidth-efficient, and per-subcarrier equalisation handles frequency-selective fading well. The basis of Wi-Fi, LTE/5G, and DSL.
- **Spread spectrum** — deliberately spread a signal over far more bandwidth than it needs, via a hopping pattern (**FHSS**) or a chip code (**DSSS**/CDMA). Resists narrowband interference and jamming, and lets many users share a band by using different codes.

## Media and switching

- **Twisted pair** — cheap, ubiquitous (Cat 6/6a for 10 GbE over short runs); susceptible to interference, hence the twisting.
- **Coax** — better shielding, legacy for cable.
- **Fibre** — single-mode for long haul (one light path, lowest dispersion, tens of km without repeaters), multi-mode for short reaches. Immune to EMI, enormous bandwidth.
- **Wireless** — sub-6 GHz (range, penetration) vs mmWave (huge bandwidth, short range, blocked by walls).

Finally, the physical layer sets the switching model: **circuit switching** dedicates a path end to end (guaranteed rate, wasted when idle — the old phone network); **packet switching** shares links statistically (efficient, variable delay — the internet). And two delays that always matter: **transmission delay** (packet size ÷ link rate — time to push the bits out) and **propagation delay** (distance ÷ signal speed — time for the first bit to arrive), which is why a satellite link has low bandwidth-delay issues but ~250 ms of latency you can't engineer away.

## The one idea to keep

Bits don't travel — signals do, and Shannon's `C = B log₂(1 + S/N)` is a hard ceiling on how fast: to go faster you need more spectrum or a better signal-to-noise ratio, nothing else. Line coding keeps the receiver's clock locked and the DC balanced; modulation (QAM constellations) packs multiple bits per symbol, as many as the current SNR allows; and multiplexing (TDM, WDM, OFDM) shares one medium among many streams. Everything above this layer inherits these limits.
