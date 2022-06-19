---
title: Operating Systems - The Layer Between Hardware and Programs
description: What an operating system is and how it is put together - the kernel and user mode, the bootstrap sequence, system calls as the interface to kernel services, the evolution from batch monitors to multitasking, and the monolithic, layered, and microkernel ways of structuring it.
date: 2022-06-19
draft: false
slug: /operating-system/os
tags:
  - Operating Systems
  - Processes
  - Memory Management
---

Your program never touches the disk, the network card, or another program's memory directly. Between the hardware and every application sits the **operating system**: the software that shares the machine's resources among competing programs and hides the hardware's specifics behind a uniform interface. Two views of the same thing — to a user it is what makes the computer convenient; to the machine it is the resource allocator and the control program that keeps user code from breaking things.

This post is the map of the OS: how it is layered, how it starts, how programs ask it for services, how the design evolved, and the main ways its internals are organised. The [process](/citadel/operating-system/process-thread), [scheduling](/citadel/operating-system/process-scheduling), [memory](/citadel/operating-system/memory-management), [file system](/citadel/operating-system/file-system), [I/O](/citadel/operating-system/io-management), [deadlock](/citadel/operating-system/process-synchronisation), and [protection](/citadel/operating-system/protection) subsystems each get their own post.

## The kernel and the rest

A computer system stacks four layers: **hardware** (CPU, memory, devices), the **operating system**, **application programs**, and **users**. The OS is the intermediary.

The **kernel** is the part that is always running. It executes in a privileged **kernel mode**; applications run in **user mode**, and a hardware **mode bit** enforces the split — privileged instructions (halt the CPU, do raw I/O, change the memory map) trap if attempted from user mode. Everything shipped with the OS that is *not* the kernel — shells, `cp` and `ps`, compilers, daemons — is a **system program** running in user mode, usually as a thin wrapper over kernel services. A complete OS is the kernel plus these.

## Booting

Turning the machine on runs a fixed sequence:

1. **Firmware** (ROM / UEFI) executes from a fixed address, runs the power-on self-test, and initialises basic hardware.
2. It loads a small first-stage **bootstrap loader** from a known location (MBR, EFI system partition).
3. That loads a larger second-stage bootloader (GRUB, Windows Boot Manager) which understands file systems, can show a boot menu, and locates the OS **kernel** image on disk.
4. The bootloader loads the kernel into memory and jumps to it.
5. The kernel initialises its subsystems and drivers, mounts the root file system, starts the first process (`init` / `systemd`), and brings the system to a login prompt.

## Kernel data structures

The kernel manages huge dynamic collections — ready processes, free memory blocks, pending I/O — with the standard [data structures](/citadel/data-structures/data-structures):

- **Linked lists** for queues that change constantly (ready queues, free lists, device request queues).
- **Stacks** for kernel function calls.
- **Queues** for FIFO waiting (I/O requests, processes blocked on a semaphore).
- **Balanced trees** ([red-black](/citadel/data-structures/red-black-tree), [B-trees](/citadel/data-structures/b-tree)) for sorted data — memory regions, scheduler run queues, file-system indices.
- **Hash tables** for fast keyed lookup — virtual-to-physical address maps, buffer caches.
- **Bitmaps** for tracking many fixed-size resources — free disk blocks, free memory frames.

## System calls

A user program requests a kernel service — open a file, create a process — through a **system call**. It is the only way into privileged code.

Programmers rarely invoke one directly; they call an **API** function (the C standard library, POSIX, Win32, the Java API) that wraps it. `printf()` eventually calls the `write()` system call. The mechanism: the library puts arguments in place, executes a **trap** instruction that switches to kernel mode and jumps to the system-call handler, which uses a **system-call number** to index a table of kernel routines. On return it switches back to user mode.

Parameters reach the kernel in one of three ways: in **registers** (fast, limited count), in a **memory block** whose address is passed in a register (Linux, Solaris — no size limit), or pushed on the **stack**.

System calls group into six categories: **process control** (`fork`, `exec`, `wait`, `exit`), **file management** (`open`, `read`, `write`, `close`), **device management** (`ioctl`, request/release), **information maintenance** (`getpid`, `time`), **communication** (`pipe`, `shmget`, send/receive), and **protection** (`chmod`, `chown`).

## How the design evolved

Each stage answered a limitation of the last:

1. **Batch processing.** Jobs on punch cards, grouped into batches, run one at a time by a resident **monitor**. The CPU idled through every I/O operation. This era introduced hardware **memory protection** (guard the monitor) and a **timer** (stop a job hogging the CPU).
2. **Multiprogramming.** Keep several jobs in memory; when the running one blocks for I/O, switch the CPU to another that is ready. CPU utilisation and throughput jump. But it is not interactive.
3. **Time-sharing (multitasking).** Switch between jobs many times a second on fixed **time slices**, so each user gets sub-second **response time** and the illusion of a private machine. Each running program is a **process**. If they do not all fit in RAM, use **swapping** or **virtual memory**. Multics and UNIX.
4. **Real-time systems.** Correctness now depends on *when* a result is produced. An RTOS guarantees critical tasks meet their deadlines — industrial control, avionics.
5. **Multithreading.** A **thread** is a unit of CPU execution within a process; threads share the process's code, data, and open files but each has its own stack and registers. Cheaper than processes for concurrent work within one program, and they exploit multiple cores.

## Structuring the kernel

How the OS's functions are organised internally decides how maintainable and how fast it is.

- **Monolithic.** Nearly everything — scheduling, memory, file systems, drivers — in one kernel address space. Component calls are direct function calls, so it is fast. But everything is interdependent: hard to modify, and a bug anywhere can crash the system. MS-DOS, classic UNIX, Linux (modular but monolithic).
- **Layered.** Stack the OS in levels, each using only the level below. Modular and easy to verify, but choosing the layers is hard and every request pays overhead crossing them.
- **Microkernel.** Keep only the essentials in the kernel — IPC, minimal memory management, low-level scheduling — and run file systems, drivers, and networking as user-space **server** processes that communicate by message passing. Extensible, portable, and reliable (a crashing server does not take the kernel down), but message passing is slower than direct calls. Mach, QNX.

## OS services

The services that make the machine usable:

- **User interface** — CLI, GUI, or batch.
- **Program execution** — load, run, and terminate programs.
- **I/O operations** — mediate access to files and devices.
- **File-system manipulation** — create, delete, read, write, and manage permissions.
- **Communication** — [IPC](/citadel/operating-system/process-thread) via shared memory or message passing, across a network too.
- **Error detection** — catch CPU, memory, device, and program errors and respond.
- **Resource allocation** — divide CPU, memory, storage, and devices among concurrent processes.
- **Accounting** — track resource usage.
- **Protection and security** — [internal access control](/citadel/operating-system/protection) and defence against external threats via authentication.

## The one idea to keep

An operating system is two jobs at once: an abstraction layer that gives every program the same clean view of files, memory, and devices, and a referee that shares the real hardware fairly and safely among programs that do not trust each other. The kernel/user-mode split enforced by hardware is what makes the referee possible; the system-call interface is where the two jobs meet.
