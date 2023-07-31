---
title: Distributed Algorithms - Models, Consensus, Time, and Fault Tolerance
description: How independent computers coordinate with no shared clock, no global state, and components that fail. The synchronous and asynchronous models, leader election and consensus, why consensus is impossible in the asynchronous model with one failure, logical time with Lamport and vector clocks, consistent global snapshots, and the techniques that make systems fault-tolerant.
date: 2023-07-31
draft: false
slug: /miscs/distributed-algorithms
tags:
  - Distributed Systems
  - Algorithms
---

A distributed algorithm runs across many interconnected processors that coordinate by passing messages. The difficulty is everything the single-machine case takes for granted: processes run concurrently, some fail while others continue, message delays are unpredictable, and no process ever has an instantaneous view of the whole system. This post works through the standard models, the core problems (leader election, consensus), how to reason about time without a global clock, and how to keep a system working when parts of it break.

## The synchronous model

The idealised starting point. Its assumptions:

- **Lock-step rounds.** Computation proceeds in synchronised rounds; each round, every process sends messages, receives that round's messages, and computes locally.
- **Bounded message delay** — a known upper limit on delivery time.
- **Bounded processing time** — local computation per round is also bounded.

Timing is predictable, which makes algorithms easier to design and analyse.

### Leader election in a ring

**Problem:** processes in a ring topology need to autonomously agree on one unique leader.

**LCR (LeLann–Chang–Roberts):** each process sends its unique ID clockwise. On receiving an ID, a process forwards it if it is larger than its own, discards it if smaller. A process that receives *its own* ID back declares itself leader — its ID must have circled the whole ring unbeaten, so it is the maximum. Cost is measured in rounds and total messages.

### Other synchronous algorithms

- **Breadth-first spanning tree** — processes cooperate to build a BFS tree rooted at a source or the elected leader, layer by layer over successive rounds. Used for efficient broadcast and aggregation.
- **Broadcasting** — spreading one process's value to all others, done efficiently along a spanning tree.

### Consensus

**Consensus** is getting all correct (non-faulty) processes to agree on a single value, despite failures.

- **With link failures** (messages can be lost, but delays are bounded): use timeouts. An expected message that misses its bound marks the link or sender as suspect for that round; retransmit or reroute. Solvable while enough connectivity remains.
- **With crash failures** (a process stops): still non-trivial, but consensus **is** solvable in the synchronous model provided the number of faulty processes stays below a threshold. Algorithms run multiple rounds of value exchange with decision rules (majority vote, or a default when too few responses arrive). This is the key contrast with the asynchronous model, where crash-tolerant deterministic consensus is impossible.

## The asynchronous model

Drop the timing guarantees: no bound on message delay, no bound on process speed, no global clock. You cannot use a timeout to reliably tell a slow process from a dead one, which is what makes this hard.

### Shared memory

Processes communicate through shared registers.

- **Atomic operations** — `read` and `write` execute indivisibly (a read sees either the old or the fully-written new value). Stronger primitives: `test-and-set`, `fetch-and-add`, `compare-and-swap` (CAS).
- **Mutual exclusion** — only one process in a critical section at a time. Software: **Peterson's algorithm** (two processes), **Lamport's bakery algorithm** (N processes, "take a number"). Hardware: atomic instructions like test-and-set or CAS give simpler, faster solutions. The single-machine version of this is in [process synchronisation](/citadel/operating-system/process-thread).
- **Consensus in shared memory** — with only atomic read/write registers, no deterministic algorithm solves consensus for N processes if even one may crash (the shared-memory analogue of FLP). With **CAS**, consensus becomes solvable — indeed **wait-free** (each process finishes in a bounded number of its own steps). CAS has *consensus number infinity*: it solves consensus for any number of processes.
- **Linearizability** — the strong correctness condition for concurrent objects: each operation appears to take effect instantaneously at one point between its call and its return, so you can reason about the object as if operations were sequential.

### Message passing

- **Flooding** — a process receiving a message for the first time forwards it to all neighbours except the sender. Simple, but redundant.
- **Echo algorithms** — an initiator sends a wave to its neighbours; leaves echo back; the initiator collects echoes to detect completion or gather data. Used for spanning trees and termination detection.
- **Synchronizers** — let an algorithm written for the synchronous model run on an asynchronous network by simulating rounds: a process waits for "enough" messages before advancing. Cost: each simulated round runs at the speed of its slowest participant. Variants trade message overhead against latency — **alpha** (acknowledge every message, simple), **beta** (lower overhead), **gamma** (optimised per topology).

## Time without a clock

To order events across an asynchronous system, use **logical time**.

- **Lamport timestamps** — each process keeps a counter. Increment before each local event; attach the value to every message sent; on receipt, set the counter to `max(local, received) + 1`. Guarantees: if event *A* causally precedes *B*, then `timestamp(A) < timestamp(B)`. The converse does not hold — a smaller timestamp does not imply causal precedence; the events may be concurrent.
- **Vector clocks** — each process *Pᵢ* keeps a vector `VCᵢ` of length N; `VCᵢ[j]` is its estimate of *Pⱼ*'s logical time. Increment `VCᵢ[i]` per local event; send the whole vector; on receipt take the elementwise `max`, then increment your own entry. Now `A → B` **if and only if** `VC(A) < VC(B)` componentwise; incomparable vectors mean the events are concurrent.

### Global snapshots

The **Chandy–Lamport snapshot algorithm** captures a consistent global state of an asynchronous system:

1. An initiator records its own state and sends a **marker** on every outgoing channel.
2. When a process first receives a marker on a channel, it records its own state, records that channel's state as empty, starts recording messages on its *other* incoming channels, and sends markers on all its outgoing channels.
3. A later marker on an already-recorded channel stops recording for that channel; its state is the messages captured since.

The collected local and channel states form a consistent snapshot. Used to detect **stable properties** — properties that stay true once true (termination, deadlock): if the property holds in the snapshot, it genuinely held at some point in the real execution.

### Partial synchrony

Between the two extremes: bounds on delay and speed exist but are unknown, or hold only *eventually* after an initial asynchronous period ("eventual synchrony"). This is realistic enough for real systems and strong enough to make consensus solvable, unlike full asynchrony with failures.

## Fault tolerance

Fault tolerance — correct service despite crashes, link failures, or corruption — is a main reason to build distributed. See also [distributed databases](/citadel/dbms/distributed-database) and [latency and consistency](/citadel/interview/latency-consistency).

- **Basic techniques** — redundancy (multiple copies of hardware, software, or data), replication (copies of data or services on different nodes), diversity (different implementations of redundant components).
- **Quorum systems** — an operation needs permission from a subset (quorum) of servers, chosen so any two write quorums, and any read and write quorum, overlap — which keeps reads and writes consistent.
- **Replication strategies:**
  - **Active (state-machine) replication** — every replica executes every request. Needs deterministic operations and reliable ordered broadcast.
  - **Passive (primary-backup)** — the primary handles requests and ships updates to backups; a backup takes over on primary failure.
- **FLP impossibility** — the Fischer–Lynch–Paterson result: no deterministic algorithm solves consensus in an asynchronous message-passing system with even one crash failure. Circumvented by:
  - **Randomisation** — reach consensus with high probability.
  - **Failure detectors** — an oracle giving (possibly wrong) suspicions about crashes.
  - **Partial synchrony** — if the system is eventually synchronous, consensus is solvable.
- **Failure detectors** — classified by **completeness** (every crashed process is eventually permanently suspected by every correct process) and **accuracy** (how often correct processes are wrongly suspected). A **perfect detector (P)** never suspects a correct process; an **eventually perfect detector (◇P)** stops doing so after some point. The Chandra–Toueg algorithm solves consensus with ◇P in an asynchronous system with crash failures.
- **Self-stabilisation** — the strongest form: from *any* arbitrary initial state, including one corrupted by a transient fault, the system is guaranteed to converge to a legitimate state and stay there, with no outside intervention. Hard to design, extremely resilient.

## The one idea to keep

The whole field turns on what you assume about timing. In the synchronous model, timeouts distinguish slow from dead, and consensus is solvable despite crashes. In the asynchronous model that distinction is impossible, so deterministic crash-tolerant consensus is impossible (FLP), and every practical system escapes that by adding an assumption back — randomness, a failure detector, or eventual synchrony.
