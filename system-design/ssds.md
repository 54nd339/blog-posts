---
title: How an SSD Works - The Controller, Parallel Flash, and the Translation Layer
description: "An SSD is faster than a hard drive for three reasons: no moving parts, dozens of flash chips written in parallel, and a translation layer that hides flash's awkward erase-before-write rule. A look at each, and the architecture that ties them together."
date: 2024-12-13
draft: false
slug: /system-design/ssds
tags:
  - System Design
  - Storage
  - Hardware
---

A solid-state drive reads about 10x and writes about 20x faster than a spinning hard disk, with access latency around 100 microseconds. It isn't one trick — it's the absence of moving parts, plus parallel flash chips, plus a controller doing real work to hide how flash actually behaves.

An SSD stores bits in flash memory cells built from floating-gate transistors. No platters, no heads, no motor — it's closer to a very large, very fast USB stick than to a hard drive.

![The SSD architecture: a host interface (SATA/PCIe) into a controller containing a processor, a RAM buffer, a buffer manager, and a flash controller, which drives multiple NAND flash packages over parallel channels](../images/ssd-architecture.png "Host interface, controller (processor, RAM buffer, flash controller), and parallel NAND packages.")

## The architecture

- **Host interface (SATA or PCIe)** — how the drive receives commands from the OS.
- **SSD controller** — the brain: an embedded processor that takes host commands ("read this block", "write this data") and drives the flash. It contains a **RAM buffer** that caches hot data and, importantly, holds mapping information; a **buffer manager**; and a **flash controller** that talks to the NAND chips directly.
- **Flash memory packages** — where data lives, organised into "gangs" reached over multiple parallel channels.

## Why it's fast

### Parallelism

The controller operates many NAND chips at once. When the OS writes several pages, the controller spreads them across multiple flash chips and writes them simultaneously, multiplying effective bandwidth. A hard drive has one read/write head and can touch one track on one platter at a time.

### The Flash Translation Layer

The OS addresses storage as a flat sequence of **logical block addresses (LBAs)**. Flash is organised into pages and blocks with an awkward rule: a block must be *erased* before any page in it can be rewritten. The **Flash Translation Layer (FTL)** bridges the two.

When the OS writes to a logical page, the controller picks any free physical page on any chip, writes there, and records the logical-to-physical mapping (in the RAM buffer). A later read of that logical page follows the mapping straight to the right physical page. Because writes can go anywhere, the FTL also spreads them evenly across cells (**wear leveling**, since flash cells wear out after a finite number of erases) and reclaims stale pages in the background (**garbage collection**).

### No mechanical latency

A hard drive pays **seek time** (moving the head to the track) and **rotational latency** (waiting for the sector to spin under the head) on every access. An SSD has neither — access is electronic.

## SSD versus HDD

| | SSD | HDD |
| --- | --- | --- |
| Speed | 10–20x faster on many ops | slower |
| Mechanism | electronic flash | spinning platters, moving heads |
| Shock resistance | high | low |
| Noise | silent | audible |
| Cost per GB | higher | lower |
| Power | lower | higher |
| Fragmentation | largely irrelevant (FTL) | degrades performance |

## The takeaway

The mechanical-latency win is the intuitive one, but the controller is what makes an SSD more than fast flash: parallelism across chips for bandwidth, and the FTL turning flash's erase-before-write constraint into something the OS never has to see. That 100-microsecond figure is one rung on [the latency ladder](/citadel/random-thoughts/latency-number), well below RAM and well above disk seeks; see also [the memory hierarchy](/citadel/interview/computer-memories). For a deep treatment, Emmanuel Goossaert's *Coding for SSDs* is the standard reference.
