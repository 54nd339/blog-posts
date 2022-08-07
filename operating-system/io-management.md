---
title: I/O Management - Polling, Interrupts, DMA, and Disk Scheduling
description: How the OS moves data between the CPU and a zoo of peripherals - the three ways to drive a device (polling, interrupts, DMA), the kernel I/O subsystem and device drivers, buffering, caching and spooling, and disk scheduling algorithms from FCFS to C-LOOK.
date: 2022-08-07
draft: false
slug: /operating-system/io-management
tags:
  - Operating Systems
  - File Systems
  - Scheduling
---

A CPU that cannot talk to a keyboard, screen, disk, or network is useless. The catch is that those devices differ wildly — in speed, in how they transfer data, in their command protocols — and the [OS](/citadel/operating-system/os) has to present all of them to applications through one clean interface while keeping each one fed efficiently.

This post covers the three mechanisms for driving a device, the subsystem that hides device differences, the buffering that smooths the speed gap, and disk scheduling.

## Devices, ports, buses, controllers

I/O devices fall into **human-interface** (keyboard, mouse, display, audio), **storage** (HDD, SSD, optical, tape), and **transmission** (network interfaces, modems). They connect through common concepts:

- **Ports** — a connection point (USB port, serial port).
- **Buses** — a shared electrical pathway (PCI Express, USB).
- **Controllers (host adapters)** — hardware that speaks a device's protocol, exposing **registers** for commands, status, and data.

## Driving a device

### Polling (programmed I/O)

The CPU loops on the controller's status register waiting for a "busy" bit to clear, then writes a command and transfers a word. Simple, but the CPU **busy-waits** — wasted cycles for slow devices, and if it steps away it can miss the brief ready window and lose data.

### Interrupts

Instead of polling, the CPU does other work. When the controller finishes, it raises a signal on an **interrupt-request line**. The CPU finishes its current instruction, saves state, and runs the device's **interrupt handler**, which services the device and returns. Far more efficient, but each interrupt costs a state save/restore and handler run — expensive if a fast device interrupts for every few bytes.

### Direct Memory Access

For bulk transfers, interrupting per byte is hopeless. A **DMA controller** does the transfer itself: the CPU gives it a source address, destination address, and byte count; the DMA controller moves the data directly between device and memory, grabbing the memory bus when it needs it (**cycle stealing**); and it raises *one* interrupt when the whole transfer completes. The CPU runs other work throughout. The cost is the DMA hardware and slight CPU slowdown when they contend for the bus.

## The kernel I/O subsystem

The OS hides per-device messiness behind the **kernel I/O subsystem**:

- **Device drivers** — per-device modules that know a device's registers and protocol and present a standard interface upward.
- **Uniform interface** — applications use the same system calls (`open`, `read`, `write`, `close`) regardless of device.
- **Device categories** — **block devices** (disks, SSDs; fixed-size blocks, random access), **character devices** (keyboards, serial ports; byte streams, sequential), **network devices** (via **sockets**), and **clocks and timers**. An escape hatch like `ioctl()` sends device-specific commands directly.

**Blocking, nonblocking, asynchronous I/O:**

- **Blocking** — the process suspends until the operation completes. Simplest.
- **Nonblocking** — the call returns immediately with whatever it could transfer; the application checks back. Good for responsive UIs.
- **Asynchronous** — the call starts the I/O and returns; the OS signals the process on completion. Powerful, hardest to program.

**Vectored (scatter/gather) I/O** — one call (`readv`, `writev`) transfers to or from several memory buffers, saving the overhead of multiple calls.

## Buffering, caching, spooling

- **Buffering** — hold data in memory during a transfer to (a) bridge a speed mismatch (fast CPU, slow printer), (b) adapt between transfer sizes (network packets vs application buffers), and (c) provide **copy semantics** — the data written is the version the application had at the call, even if it modifies its buffer right after. **Double buffering** fills one buffer while the other is processed.
- **Caching** — keep a *copy* of frequently used data in faster memory. A buffer holds the only copy in transit; a cache holds a redundant copy for speed. Often combined as the **buffer cache**.
- **Spooling** — queue output on disk for a device that serves one job at a time (a printer), then feed it sequentially.

The subsystem also handles **device reservation** (exclusive access, e.g. a tape drive), **error handling** (retry, log), and **I/O protection** — I/O instructions are privileged, so user programs must go through system calls.

## Disk I/O

Disks are the workhorse of persistent storage, and their access cost is what disk scheduling optimises.

A disk is a large array of **logical blocks** mapped onto physical **sectors**. Reading one involves:

- **Seek time** — moving the read/write arm to the right cylinder. The dominant cost for random HDD access.
- **Rotational latency** — waiting for the sector to spin under the head.
- **Transfer time** — moving the bytes.

When many requests queue for one disk, the OS reorders them with a **disk scheduling algorithm** to cut total seek time:

| Algorithm | Behaviour | Trade-off |
| --- | --- | --- |
| FCFS | serve in arrival order | fair; wild seek patterns |
| SSTF | nearest request first | low seek time; can starve far requests |
| SCAN (elevator) | sweep across the disk, reverse at the end | bounded wait; edges wait longest |
| C-SCAN | sweep one way, jump back to the start | more uniform wait than SCAN |
| LOOK / C-LOOK | like SCAN / C-SCAN but reverse at the last request, not the platter edge | avoids pointless travel |

SSTF and LOOK are common defaults. SSDs have no seek time or rotational latency, so their scheduling instead targets wear levelling and internal parallelism.

Two related disk concerns have their own homes: **[swap space](/citadel/operating-system/memory-management)** — the backing store for demand paging — and **[RAID](/citadel/operating-system/file-system)** — combining disks for performance and redundancy.

## OS design issues, seen through I/O

I/O management surfaces the trade-offs that run through the whole OS:

- **Managing diversity** — one interface over an ever-changing device zoo (solved by layering and drivers).
- **Performance versus abstraction** — every abstraction layer that simplifies programming adds overhead.
- **Efficiency** — caching, buffering, DMA, and scheduling exist to hide how slow I/O is relative to the CPU.
- **Reliability** — graceful error handling; journaling for fast crash recovery.
- **Protection and security** — privileged instructions and system calls stop user code touching devices directly.
- **Flexibility** — how easily new device or file-system types can be added.
- **Simplicity versus functionality** — more features means more complexity and more ways to fail.

## The one idea to keep

The progression polling → interrupts → DMA is the OS handing off ever more of the transfer work so the CPU can do something useful while I/O happens. Everything above that — drivers, the uniform interface, buffering, caching, disk scheduling — exists to paper over the fact that devices are wildly slower and more varied than the processor that commands them.
