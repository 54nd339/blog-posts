---
title: Virtualization and Hypervisors
description: Running many operating systems on one machine needs the hardware to trap every privileged instruction a guest executes - which classic x86 did not do. The fixes were binary translation, then paravirtualization, then hardware support, plus a second layer of page tables. Containers get most of the isolation for none of the second kernel.
date: 2023-07-20
draft: false
slug: /operating-system/virtualization
tags:
  - Operating Systems
  - Processes
---

An [operating system](/citadel/operating-system/os) is written assuming it owns the machine — it runs privileged instructions, manages all of physical memory, talks directly to devices. Virtualization runs *several* such operating systems on one physical machine at once, each believing it's alone. The component that makes this work, the **hypervisor** or virtual machine monitor, has to give every guest OS that illusion while keeping them isolated and the host in control.

The interesting history is that x86 — the architecture everyone wanted to virtualize — was *not virtualizable* by the textbook method for its first 20 years, and the workarounds shaped how VMs and, later, containers evolved.

## What "virtualizable" requires

Popek and Goldberg (1974) gave the condition: a machine is efficiently virtualizable if every **sensitive** instruction (one that reads or changes hardware state, or behaves differently by privilege level) is also **privileged** (traps when executed outside the most privileged mode).

If that holds, you build a hypervisor by **trap-and-emulate**: run the guest OS in user mode; whenever it executes a sensitive instruction, the CPU traps to the hypervisor, which emulates the instruction's effect on the *virtual* machine state and returns. The guest never knows it wasn't running privileged.

x86 broke this. Instructions like `popf` (which can modify the interrupt-enable flag) simply *did nothing* to the flag when run in user mode instead of trapping — so a guest kernel's attempt to disable interrupts would silently fail with no trap for the hypervisor to catch. Seventeen such instructions made trap-and-emulate impossible on x86.

## The three approaches

### Binary translation (VMware, ~1999)

Scan the guest kernel's instruction stream just ahead of execution and **rewrite** it: replace every problematic instruction with a call into the hypervisor, cache the translated blocks, and run those. User-mode guest code runs untranslated at native speed; only kernel code pays the translation cost. Clever, complex, and it made x86 virtualization real before the hardware caught up.

### Paravirtualization (Xen, ~2003)

Change the guest OS. Replace the sensitive operations in the kernel with explicit **hypercalls** — a defined API to the hypervisor, like syscalls but one level down. No translation, low overhead, but it requires a modified guest, so it only ever worked for open-source OSes (Linux, BSD) and not Windows. Modern systems keep paravirtualized *drivers* (virtio — a guest disk/network driver that talks a hypervisor-friendly protocol instead of emulating real hardware register-by-register) even when the CPU is virtualized in hardware, because emulating a real NIC is slow.

### Hardware-assisted (Intel VT-x, AMD-V, ~2006)

Add a new CPU mode. **VMX root** mode is the hypervisor; **VMX non-root** mode runs the guest, and in it the previously-non-trapping sensitive instructions now cause a **VM exit** into the hypervisor. A per-guest **VMCS** (virtual machine control structure) configures which events exit and holds the saved guest/host state. Trap-and-emulate finally works on x86, directly in silicon. This is what everything uses now; the hypervisor's job becomes handling VM exits efficiently and keeping their frequency low.

## Memory virtualization

The guest OS maintains page tables mapping **guest virtual → guest physical**. But "guest physical" isn't real physical memory — the hypervisor has its own mapping of **guest physical → host physical**. Two levels of translation.

- **Shadow page tables** — the hypervisor maintains a hidden set of page tables that map guest virtual straight to host physical (the composition of both levels), and points the hardware MMU at those. Every guest page-table edit must be intercepted to keep the shadows in sync — expensive.
- **Nested paging** (Intel EPT, AMD RVI) — the hardware walks *both* levels itself. The guest freely edits its own page tables; the MMU translates guest-physical through the hypervisor's second-level table with no exits. A TLB miss costs more (a two-dimensional page walk) but page-table edits are free. This is the standard now.

Other memory tricks: **ballooning** (a guest driver that, on hypervisor request, allocates guest memory and pins it, letting the hypervisor reclaim the underlying host pages — the guest "feels" memory pressure and frees things), **page sharing** (deduplicate identical pages across guests, e.g. the same OS code), and **overcommit** (allocate guests more RAM in total than the host has, betting they won't all use it — swap covers the gap, at a cliff).

## Type 1 vs Type 2

- **Type 1 (bare metal)** — the hypervisor runs directly on hardware; guests run on it. VMware ESXi, Xen, Microsoft Hyper-V, KVM (which turns the Linux kernel itself into the hypervisor). What data centres and clouds run.
- **Type 2 (hosted)** — the hypervisor is an application on a normal OS. VirtualBox, VMware Workstation, QEMU standalone. Convenient on a laptop; an extra layer of overhead.

The line is blurry — KVM is "Type 1" but is a Linux module, and the Linux userspace (QEMU) handles device emulation.

## I/O

The hypervisor can **emulate** a device (present a virtual e1000 NIC; slow, universally compatible), use a **paravirtual** device (virtio; fast, needs a guest driver), or do **passthrough** — assign a real physical device (or a slice of one via **SR-IOV**, which lets one physical NIC present many virtual functions) directly to a guest, with an **IOMMU** (Intel VT-d) restricting the device's DMA to that guest's memory so it can't scribble on the host.

## Containers, and the middle ground

A **container** isn't a VM. It's a normal process on the host kernel, isolated by kernel features:

- **Namespaces** — give the process its own view of PIDs, mounts, network interfaces, users, hostnames, IPC.
- **cgroups** — cap and account its CPU, memory, I/O, and PIDs.
- **Union filesystems** (overlayfs) — layer a read-only image plus a writable top layer, so images are shareable and cheap.

No second kernel, no hardware virtualization, so containers start in milliseconds and add near-zero overhead. The trade is a weaker boundary: every container shares the host kernel, so a kernel exploit is a full escape, whereas a VM would still be contained. **microVMs** (AWS **Firecracker**) and **gVisor** (a user-space kernel that intercepts guest syscalls) split the difference — VM-grade isolation with container-grade startup and density, which is what serverless platforms run.

| | VM | microVM | Container |
|---|---|---|---|
| Kernel | own | own (minimal) | shared |
| Boundary | hardware | hardware | namespaces + seccomp |
| Start time | seconds | ~100 ms | ~10 ms |
| Overhead | few % | low | negligible |

## Live migration

A running VM can be moved to another host with sub-second downtime: pre-copy its memory pages while it keeps running, re-copy the pages that got dirtied, repeat until the dirty set is tiny, then pause for a moment to copy the last pages and the CPU state, and resume on the target. It's how clouds evacuate a host for maintenance without killing your instance.

## The one idea to keep

A hypervisor gives each guest OS the illusion of owning the machine by trapping every privileged instruction and emulating its effect — which x86 couldn't support until VT-x/AMD-V added a guest CPU mode that forces the exits. Memory needs a second translation layer, done cheaply in hardware by nested paging. Containers skip the whole second-kernel apparatus, getting most of the isolation from namespaces and cgroups at almost no cost, with microVMs and gVisor filling in when a shared kernel isn't a strong enough wall.
