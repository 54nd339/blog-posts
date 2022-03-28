---
title: Memory Technology - SRAM, DRAM, ROM, and Memory Modules
description: The parts a memory system is built from - static versus dynamic RAM cells and why DRAM needs refresh, the RAS/CAS to SDRAM to DDR evolution, the ROM and flash family, how small chips are combined into a wide memory module, and how interleaving widens the memory pipe.
date: 2022-03-28
draft: false
slug: /computer-architecture/memory-system
tags:
  - Computer Architecture
  - Memory
  - DRAM
---

"RAM" is one word for two technologies with opposite trade-offs. The cells in your CPU's cache and the cells in your DIMMs are built differently, run at different speeds, cost different amounts per bit, and one of them silently forgets its contents thousands of times a second unless the memory controller keeps rewriting them.

This post is the physical layer: what SRAM and DRAM cells actually are, how a DRAM chip is addressed and how that addressing sped up from asynchronous RAS/CAS to DDR5, the non-volatile ROM and flash family, and the board-level tricks — wide modules, banking, interleaving — that turn a pile of narrow chips into a fast, wide main memory. The *hierarchy* built on top of these parts — locality, caches, virtual memory — is [the next post](/citadel/computer-architecture/hierarchical-memory).

## Access time, cycle time, and the gap

A processor reaches memory by putting an **address** on the bus; $k$ address bits select one of $2^k$ locations. Data comes back in fixed groups called **words** (32 or 64 bits), though each byte has its own address. Two timing numbers describe a memory:

- **Access time** — from issuing a request to the data being available.
- **Cycle time** — the minimum gap between two successive requests. Always ≥ access time, because the array needs to recover.

The problem the rest of computer architecture dances around: a modern core runs an instruction in well under a nanosecond, while a main-memory access is tens of nanoseconds — a factor of roughly 100. No single memory is both big and that fast at a price anyone will pay. The answer is the **memory hierarchy**: registers, then L1 cache, then L2 (and L3), then main memory, then disk. Going down, capacity rises, speed falls, and cost per bit falls. This post is the levels; the mechanics that make them act as one memory are [hierarchical memory](/citadel/computer-architecture/hierarchical-memory).

## SRAM versus DRAM

Both are volatile — lose power, lose data — but that's where the similarity ends.

**Static RAM (SRAM)** stores each bit in a latch, typically six transistors. It holds its value as long as it's powered, with no maintenance, and it's fast — a few nanoseconds. It's also bulky and expensive per bit. That's why it's used for caches, where speed is everything and capacity is small.

**Dynamic RAM (DRAM)** stores each bit as charge on a tiny capacitor, gated by a single transistor. One transistor plus one capacitor is far denser and cheaper than six transistors — so DRAM is main memory. The price: the capacitor leaks, so every row must be read and rewritten on a schedule (**refresh**), typically every few milliseconds, and access is slower than SRAM.

## Inside a DRAM chip

The cells sit in a rectangular array. Each row is roughly one word wide and shares a **word line**; an **address decoder** activates one row, and **sense/write amplifiers** on the column **bit lines** read or write it.

To save chip pins, the address is **multiplexed**: the row half is sent first and latched by the **Row Address Strobe (RAS)**, then the column half, latched by the **Column Address Strobe (CAS)**. From that base, DRAM interfaces got faster in steps:

- **Fast Page Mode (FPM)** — once a row is open (latched by RAS), read many columns in it by pulsing CAS with new column addresses, skipping the RAS each time. Good for streaming a block.
- **Synchronous DRAM (SDRAM)** — operation is tied to a clock, so timing is tighter and higher. **Burst mode**: give one starting address and the chip generates the following consecutive addresses itself, one per clock.
- **DDR SDRAM** — transfers data on *both* the rising and falling clock edges, doubling bandwidth at the same clock. The cell array is split into several **banks** so one can be opening a row while another delivers data. DDR3, DDR4, DDR5 are successive tightenings of this.

## ROM and flash

**Read-only memory** keeps its contents with the power off, which is why firmware — the boot code (BIOS/UEFI), embedded software — lives there. The family, by how it's written:

- **Masked ROM** — contents fixed during fabrication.
- **PROM** — user-writable once, with a programmer.
- **EPROM** — erased by ultraviolet light through a window, then reprogrammed. A development-era part.
- **EEPROM** — erased and rewritten electrically, byte by byte, in circuit.
- **Flash memory** — an EEPROM variant where reads reach individual cells but *erase and write happen a whole block at a time*. Dropping byte-level writes makes it much denser and cheaper, which is why SSDs, USB sticks, and memory cards are flash.

## Building a bigger memory from small chips

One chip rarely has the capacity or the word width a system wants, so chips are combined on a **memory module**.

- **Wider word.** Four 8-bit-wide chips wired in parallel, each supplying 8 bits of every 32-bit word.
- **More words.** Add banks of chips. **Chip Select (CS)** lines, decoded from the high-order address bits, activate one bank per access.
- **SIMM vs DIMM.** A *Single* In-line Memory Module has the same signals on both sides of its edge connector; a *Dual* In-line Memory Module has independent contacts per side, allowing a wider data path (64 bits and up). DIMMs are what plug into today's motherboards.
- **Memory controller.** The CPU emits a full address; DRAM wants it split into row and column halves with RAS/CAS timing. The memory controller does that translation and generates the strobes. It used to sit in the chipset; now it's usually on the CPU die.

## Memory interleaving

Even with a wide module, a single DRAM array delivers one access at a time. **Interleaving** splits main memory into several independent modules and lays addresses across them so accesses can overlap.

- **Low-order interleaving** — the low address bits pick the module. Consecutive words 0, 1, 2, 3 land in modules 0, 1, 2, 3. All four modules can be driven at once, so fetching a cache line or any run of consecutive words is roughly $n$ times faster with $n$ modules. This is the common choice.
- **High-order interleaving** — the high address bits pick the module, so a whole contiguous region sits in one module. It does nothing for a single block transfer, but it lets several unrelated requests aimed at *different* regions proceed in parallel.

## The one idea to keep

Speed, capacity, and cost pull against each other and no single memory technology wins all three — so systems don't try. SRAM buys speed for the small stuff near the core, DRAM buys capacity for main memory and pays for it with refresh and latency, and module width, banking, and low-order interleaving widen the path between them. The parts here are what the [hierarchy](/citadel/computer-architecture/hierarchical-memory) assembles into the illusion of one large, fast memory.
