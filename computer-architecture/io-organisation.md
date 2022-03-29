---
title: I/O Organisation - Polling, Interrupts, DMA, and the Standard Buses
description: How a processor communicates with devices that run far slower than it does - memory-mapped versus port-mapped addressing, program-controlled polling, interrupt handling with priority and vectoring, direct memory access and bus arbitration, parallel and serial interface circuits, and the PCI, SCSI, and USB buses.
date: 2022-03-29
draft: false
slug: /computer-architecture/io-organisation
tags:
  - Computer Architecture
  - I/O
  - Buses
---

A CPU core executes billions of instructions a second. A keyboard produces a few bytes when someone types; a disk delivers data in bursts with millisecond gaps. I/O organisation is the set of mechanisms that bridges that mismatch without the processor burning its time waiting.

There are three escalating answers — check the device yourself (polling), let it tell you when it's ready (interrupts), or take yourself out of the loop entirely (DMA) — plus the interface circuits that adapt bus signals to device signals, and the standard buses that let any device plug in and be found. The [operating-system view](/citadel/operating-system/io-management) covers device drivers and disk scheduling; this is the hardware.

## Reaching a device

A device's registers have to be addressable. Two schemes:

- **Memory-mapped I/O** — device registers live in the ordinary memory address space, and any load/store instruction can reach them. No special instructions; simplest for software.
- **Port-mapped (isolated) I/O** — devices have their own address space, reached with dedicated `IN` / `OUT` instructions, with a bus control line marking the access as I/O. Devices can then decode fewer address lines.

Either way, each device sits behind an **I/O interface circuit**: an **address decoder** that recognises the device's address, one or more **data registers** (`DATAIN` for input, `DATAOUT` for output), **status registers** with flags (input-ready, output-ready, busy, error), and control logic tying it to the bus.

## Program-controlled I/O

The processor drives the transfer itself. To read a character from a keyboard: read the status flag; if it says a character is ready (`SIN = 1`), read `DATAIN`; if not, loop back and check again. This **polling** is trivial to implement and wastes the processor in a spin loop the whole time the device is slow.

## Interrupts

Instead, let the device raise an **interrupt-request line** when it needs service. The processor:

1. finishes the current instruction,
2. saves its state — at least the PC and **processor status word (PSW)** — usually onto the stack,
3. identifies which device interrupted,
4. loads that device's **interrupt-service routine (ISR)** address into the PC,
5. runs the ISR (transfer the data, clear the device),
6. restores the saved state and resumes the interrupted program.

**Interrupt latency** is the delay from the request to the first ISR instruction; real-time systems need it small and bounded.

Interrupts can be **enabled or disabled** — globally via an interrupt-enable bit in the PSW, and individually via an enable bit in each device's interface. The usual convention is to disable interrupts automatically on entering an ISR and re-enable them on return, so an ISR isn't interrupted by the same or a lower-priority device.

### Many devices

Two problems appear once several devices can interrupt.

**Identifying the interrupter.** *Polling* — the ISR reads each candidate device's status register to find the one that requested. Simple, linear in the device count. *Vectored interrupts* — the device supplies a **vector**, an index into an **interrupt vector table** of ISR addresses, so the processor jumps straight to the right handler.

**Priority and simultaneity.** If several devices interrupt at once, or a high-priority device interrupts during a low-priority ISR, something must arbitrate:

- **Separate request/acknowledge lines** per device or priority level, with logic that picks the highest active request.
- **Daisy chaining** — devices in series; the acknowledge signal propagates from highest priority to lowest, and the first device with an active request blocks it from going further and identifies itself. Priority is electrical position.
- Hybrids: several priority lines, each with its own daisy chain.

### Exceptions

**Exception** is the umbrella term for any event that diverts normal execution. I/O interrupts are one kind; others include hardware errors (memory parity, illegal opcode, divide-by-zero), debugging support (**trace mode** raises an exception after every instruction; **breakpoints** raise one at a chosen address), and **privilege violations** (a user-mode program attempting a privileged instruction, which switches the processor to supervisor mode). All use the same save-state-and-vector machinery.

## Direct memory access

For moving a large block between a device and memory, involving the processor in every word is pointless. A **DMA controller (DMAC)** does it directly. The processor programs the DMAC with the starting memory address, the transfer count, the direction, and the device; the DMAC then becomes **bus master**, generates the addresses, and runs the transfer, raising an interrupt only when the whole block is done.

Because the DMAC and the processor both want the bus, **arbitration** is needed:

- **Cycle stealing** — the DMAC grabs the bus for one or a few transfers, then releases it, interleaving with processor activity.
- **Burst (block) mode** — the DMAC holds the bus for the entire block, briefly stalling the processor.
- The arbiter itself is **centralized** (one arbiter grants mastership, often with daisy-chained grant lines) or **distributed** (devices run a protocol among themselves by ID or priority).

## Interface circuits: parallel and serial

A **parallel port** moves many bits at once. On input, an encoder loads `DATAIN` and sets a status flag (`SIN`); the processor reads the flag, then the data. On output, the processor waits for an idle flag (`SOUT`), writes `DATAOUT`, and the interface strobes a `Valid` line to the device.

A **serial port** moves one bit at a time over a single line, which suits longer distances. It uses **shift registers** to convert between the parallel bus word and the serial bit stream. **Double-buffering** — a shift register plus a separate data buffer — lets the next character start arriving while the processor is still reading the previous one. The **bit rate** is configurable to match the device.

## Standard buses

Modern machines pair a fast **processor bus** near the CPU with one or more **expansion buses**, joined by a **bridge** that translates between them.

**PCI** is a processor-independent expansion bus. It supports burst transfers, multiplexes address and data on shared **AD** lines, and runs transactions from an **initiator** to a **target** using control signals `FRAME#` (transaction length), `C/BE#` (command / byte enables), `IRDY#` (initiator ready), `TRDY#` (target ready), and `DEVSEL#` (target selected). Each device carries a **configuration ROM** the system reads at boot to identify and configure it — plug-and-play.

**SCSI** connects storage devices. They're not in the processor's memory map; a **SCSI controller** (host adapter) bridges the SCSI bus to the system bus and transfers via DMA. Communication is packet-based between **initiators** and **targets**, and a target can **disconnect** from the bus after receiving a command — freeing it while the drive seeks — then **reselect** the initiator when data is ready, so many operations overlap.

**USB** is a tiered-star tree: a host controller (root hub) at the top, then hubs, then devices. The host assigns each device a unique 7-bit address when it's plugged in, and schedules all traffic, which flows in packets. **Isochronous** transfers — audio, video — get guaranteed slots within 1 ms frames. The cable has four wires: two for power (so a device can be bus-powered) and two for differential data.

## The one idea to keep

The three I/O methods are three budgets for the processor's attention: polling spends all of it, interrupts spend it only on events, DMA spends almost none. Beyond a handful of devices, interrupts need vectoring and priority arbitration to stay manageable, and the standard buses exist to make discovery and arbitration uniform so that any device — a disk, a camera, a network card — plugs into the same slot and works.
