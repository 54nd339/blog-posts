---
title: The Memory Hierarchy - From Registers to Cloud Storage
description: Fast memory is small and expensive; large memory is slow. The hierarchy that makes a machine feel like it has both, the latency of each tier from registers to remote storage, and why byte order still matters when data crosses systems.
date: 2024-12-20
draft: false
slug: /interview/computer-memories
tags:
  - System Design
  - Interview Prep
  - Memory
---

A CPU register answers in well under a nanosecond. A read from a cloud object store can take the better part of a second — nine orders of magnitude slower. No single technology is both fast enough for the processor and cheap enough to hold everything, so machines stack several, each a different point on the speed-versus-cost curve, and move data between them so that the common case hits a fast tier.

This walks the tiers from the CPU outward with real latency figures, then covers endianness — the byte-order question that stays invisible until binary data moves between two machines that disagree.

## Why there is a hierarchy at all

Faster memory costs more per bit and therefore comes in smaller amounts; slower memory is cheap enough to buy in bulk. The hierarchy exploits **locality** — programs reuse recently touched data and touch nearby addresses next — to keep the working set in the small fast tiers and everything else in the large slow ones. Done well, the system has close to the speed of the fastest tier and close to the capacity of the largest.

## The tiers, fast to slow

**CPU registers** — on the execution units themselves. Sub-nanosecond access, the fastest storage there is. A core has maybe 32 or 64 general-purpose registers, 64 bits each — a few hundred bytes total. They hold the operands, addresses, program counter, and flags the CPU is working on right now. Volatile.

**CPU cache (L1/L2/L3)** — SRAM on or beside the chip, buffering RAM.

- **L1**, per core, split into instruction and data, tens of KB, about **1 ns**.
- **L2**, per core or shared by a few, hundreds of KB to a few MB, about **10 ns**.
- **L3**, shared across all cores, several MB, slower again.

The CPU checks L1, then L2, then L3; a hit returns fast, a miss falls through to RAM. Volatile.

**Main memory (RAM)** — DRAM on the motherboard, reached over the memory bus. About **100 ns**, gigabytes of capacity, holds the OS, running programs, and their working data. Volatile — gone at power-off.

**Secondary storage** — persistent, non-volatile:

- **SSD** — flash cells, no moving parts. Reads around **100 µs**, hundreds of GB to several TB. Holds the OS, applications, and files.
- **HDD** — spinning platters and a moving head, so every access pays seek time plus rotational latency: **single-digit milliseconds**. Cheapest per bit at large capacity; now mostly bulk and archival storage.

**Remote storage** — cloud object stores (S3, GCS, Azure Blob), NAS, SAN, tape. Access from milliseconds to seconds depending on network and service; capacity effectively unbounded. Backups, archives, data lakes, globally shared storage.

| Tier | Access time | Cost/bit | Capacity | Volatile |
| --- | --- | --- | --- | --- |
| Registers | ~1 ns or less | highest | bytes | yes |
| Cache L1–L3 | ~1–50 ns | very high | KB–MB | yes |
| RAM | ~50–100 ns | high | GB | yes |
| SSD | ~25–100 µs | medium | GB–TB | no |
| HDD | ~2–10 ms | low | TB | no |
| Remote | ms–seconds | lowest | PB+ | no |

The ratios are the point: L1 to RAM is roughly 1 to 100, RAM to SSD another 1 to 1000, SSD to a spinning disk another 100. A [cache](/citadel/interview/caching) sits between any two adjacent tiers for the same reason — to keep the hot fraction one level up. The persistent tiers and their access models are covered in [storage systems](/citadel/interview/storage-systems).

## Endianness: byte order across the boundary

When a value spans more than one byte — a 4-byte integer like `0x12345678` — the machine has to choose which byte lands at the lowest address.

- **Little-endian** puts the least significant byte first: address `n` holds `78`, then `56`, `34`, `12`. Used by x86, AMD64, and most ARM.
- **Big-endian** puts the most significant byte first: `12`, `34`, `56`, `78`. Used by older SPARC and 68000, IBM mainframes — and, importantly, **network byte order**: TCP/IP protocol fields are big-endian.

It stays invisible until binary data crosses between systems that disagree. Copy those four bytes verbatim from a big-endian sender to a little-endian receiver and `0x12345678` is read back as `0x78563412`. The fix in network code is to convert multi-byte fields to network order before sending with `htonl`/`htons` and back to host order on receipt with `ntohl`/`ntohs`. The same care applies to binary file formats and any direct memory manipulation. Some processors are bi-endian, selectable at boot.

## The hierarchy is a set of latency cliffs

Writing fast code is mostly about which side of a cliff your accesses land on — a cache miss to RAM costs ~100 cycles, a page fault to SSD costs hundreds of thousands, and a call to remote storage costs tens of millions. The numbers above are worth carrying in your head precisely because the design choice — inline this, batch that, cache the other — is almost always about not falling down the next step.
