---
title: Distributed OS, UNIX, and Linux - Past the Single Machine
description: A short tour past the single machine - what a distributed OS aims for and why it is hard, the design decisions that made UNIX influential (hierarchical files, fork/exec, the shell, portability in C), and how Linux inherited them as an open-source kernel with its distributions and core commands.
date: 2022-08-11
draft: false
slug: /operating-system/distributed-os
tags:
  - Operating Systems
  - File Systems
  - Processes
---

Everything so far has assumed one machine. Connect many and try to make them act as one, and you get a **distributed operating system** — a hard problem that is still mostly a research frontier. The systems that actually run the world's computers are the descendants of one 1970s design, so this post pairs a glimpse of the distributed goal with a look at the [OS](/citadel/operating-system/os) lineage that won: UNIX and Linux.

## Distributed operating systems

A **distributed OS** manages a collection of autonomous, networked computers and tries to make them look like a single system. That is more than a network OS, where users know they are logging into distinct machines; a true distributed OS aims for **transparency** — you should not be able to tell how many machines are involved.

The goals:

- **Resource sharing** — files, printers, and CPUs available across the network.
- **Computation speedup** — spread work over multiple machines (load sharing).
- **Reliability** — if one machine fails, others carry on.
- **Communication** — processes on different machines exchange messages.

The hard parts are the ones a single machine never faces: keeping a consistent view across machines, synchronising processes without shared memory, and handling **partial failure** — where some nodes are down and you cannot always tell which. It is a field of its own; the treatment here stops at the outline.

## UNIX: the design that stuck

**UNIX**, begun at Bell Labs in the late 1960s, set the conventions almost every later system follows. Its influential choices:

- **Time-sharing from the start** — multi-user, multitasking, interactive, when batch processing was still standard.
- **A hierarchical file system** — the tree of directories under `/` that everyone now takes for granted.
- **`fork()` then `exec()`** — create a near-copy of the current [process](/citadel/operating-system/process-thread), then replace its memory with a new program. Paired with **pipes** for inter-process communication, this became the standard process model.
- **The shell** — a command-line interpreter that is also a scripting language, letting users compose programs.
- **Small tools, composed** — many single-purpose utilities piped together rather than a few large ones.
- **Portability** — written in C (developed alongside it), so it moved to new hardware far more easily than assembly-language systems.

## Linux

**Linux** is a UNIX-like OS kernel, started by Linus Torvalds in 1991, developed independently and sharing no original UNIX code. It implements **POSIX** (the standard for UNIX-like interfaces) and inherits UNIX's design philosophy and command set.

- **"Linux" is the kernel** — the core managing hardware, processes, and memory. A usable system needs more.
- **Open source** — the source is freely available to read, modify, and redistribute under the GPL. The collaborative model drove rapid growth; the kernel now runs on phones (Android), the majority of servers, and nearly all supercomputers.
- **Distributions** package the kernel with system utilities (the GNU toolchain), a desktop environment, applications, and a package manager. Common ones: **Ubuntu** (user-friendly, Debian-based), **Fedora** (fast-moving, Red Hat-sponsored), **Debian** (stability, the base for many others), **Linux Mint** (traditional desktop, Ubuntu-based), **Arch** (minimal, build-it-yourself, rolling release), and the **RHEL family** (AlmaLinux, Rocky Linux — enterprise, long support). The choice comes down to stability versus recency, hardware support, and taste.

## Core commands

The command line, via a **terminal**, is how you drive a Linux system. The essentials:

```bash
pwd                 # print the current directory's full path
ls -la /etc         # list contents (long format, including hidden files)
cd ~                # change directory (~ is home; .. is parent; / is root)
mkdir project       # create a directory
rmdir empty         # remove an empty directory
touch notes.txt     # create an empty file or update its timestamp
cp -r src/ dst/     # copy (recursively for directories)
mv old.txt new.txt  # move or rename
rm -r build/        # delete (recursively); no undo
cat file.txt        # print a file
less big.log        # page through a file (space to scroll, q to quit)
echo "hello"        # print text
man ls              # show a command's manual page
sudo apt update     # run a command with administrator privileges
```

The [Linux tooling post](/citadel/tech/linux) goes further into day-to-day use.

## The one idea to keep

The distributed OS is the ambitious version of the problem — one system image over many failure-prone machines — and it is genuinely unsolved in the general case. What actually runs everywhere is the modest version: UNIX's choices from the 1970s (a file tree, `fork`/`exec`, a scripting shell, portable C) proved good enough to keep, and Linux is those choices rebuilt in the open.
