---
title: Program, Process, Thread - And Concurrency vs Parallelism
description: A program is instructions on disk, a process is that program running with its own memory, a thread is a path of execution inside it. Then the two words those units enable — concurrency (managing many tasks) and parallelism (running them at once).
date: 2024-07-09
draft: false
slug: /interview/process-thread
tags:
  - System Design
  - Interview Prep
  - Concurrency
---

"Program", "process", and "thread" get used loosely, but they are three precise things stacked on each other: a program becomes a process when it runs, and a process runs its work on one or more threads. Get the distinctions right and two more terms — concurrency and parallelism — stop being interchangeable. This walks all five.

## Program

An **executable file** on disk — `notepad.exe`, `/bin/ls`. A static list of instructions, doing nothing. A recipe, not the cooking.

## Process

A **program in execution**. The OS loads the instructions and data from disk into RAM and gives them resources: a **dedicated address space**, file handles, registers, a program counter, a stack. It is an active entity.

One program can spawn many processes. Chrome runs each tab as its own process, so a crash in one tab takes down that process and leaves the rest of the browser standing — the isolation between address spaces is the safety mechanism.

## Thread

The **smallest unit of execution, inside a process** — a lightweight path through the code. A process has one thread (single-threaded) or several (multi-threaded), each doing part of the work. In a word processor, one thread takes your keystrokes while another spell-checks and a third auto-saves.

The key fact: threads in one process **share that process's memory** — code, globals, heap — but each has its own program counter, registers, and stack. Sharing memory is what makes threads cheap to create and fast to communicate between; it is also what makes them dangerous, because two threads writing the same variable is a bug waiting to interleave.

## Process vs thread

| | Process | Thread |
| --- | --- | --- |
| Isolation | independent; one crash spares the others | share a process; if it dies, all its threads die |
| Memory | own address space | shared code/data/heap, own stack |
| Creation cost | heavyweight | lightweight |
| Context switch | expensive — swap memory maps | cheap — memory is shared |
| Communication | IPC: pipes, sockets, shared-memory segments | shared memory directly, but needs synchronisation |

## Concurrency vs parallelism

**Concurrency** is *managing* many tasks over overlapping time. They start, run, and finish in interleaved periods, not necessarily at the same instant. On a single core, the OS **time-slices** — switching rapidly between tasks so each makes progress and none blocks the rest. A chef working several dishes, one pot at a time.

**Parallelism** is *executing* many tasks at the same physical instant. It needs hardware with multiple processing units — multiple cores, multiple CPUs, multiple machines. Several chefs, each on their own dish.

The distinction in one line: **concurrency is dealing with many things at once; parallelism is doing many things at once.** You can have concurrency without parallelism (single core, time-sliced). Parallelism is one way to *implement* concurrency when you have the cores for it. Threads give a process concurrency; on a multi-core machine those threads can also run in parallel, one per core.

## The cost of sharing

Concurrency's hard part is shared mutable state. When two threads or processes touch the same data, you get:

- **Race conditions** — the result depends on which operation happened to run first ([their own topic](/citadel/interview/race-condition)).
- **Deadlocks** — two tasks each hold a resource the other needs, and both wait forever.
- **The need for synchronisation** — locks, mutexes, semaphores, or channels to serialise access to the shared thing.

## The units are the same; the discipline changes

Program to process to thread is one idea getting more concrete: instructions, then instructions running with resources, then a path of execution through them. Concurrency and parallelism are about *how many* of those paths advance and whether truly simultaneously. The moment more than one path shares data, correctness stops being free — which is why [race conditions](/citadel/interview/race-condition) and the [isolation levels](/citadel/interview/db-isolation) that fence them off are their own subjects.
