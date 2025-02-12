---
title: Heartbeats - Failure Detection in Distributed Systems
description: A node that has crashed sends nothing, which looks exactly like a node that is slow. Heartbeats turn that ambiguity into a timeout decision, and tuning the timeout is a trade between fast detection and false positives.
date: 2025-02-12
draft: false
slug: /interview/heartbeat-mechanism
tags:
  - System Design
  - Interview Prep
  - Distributed Systems
---

A worker node stops responding. Has it crashed, or is it just slow, or is the network between you and it down? From the outside these look identical — nothing arrives. A **heartbeat** is how a distributed system forces a decision: each node periodically sends an "I'm alive" signal, and a monitor that stops hearing them, past a deadline, declares the node failed and acts.

This covers the mechanism, the one hard tuning decision, and what still bites after you have tuned it.

## The mechanism

In a typical master-worker setup:

1. **Each worker sends** a small heartbeat message to the master (or a monitoring service) at a fixed **interval** — every 5 seconds, every 30 seconds.
2. **The master tracks** the last heartbeat time for every worker, and holds a **timeout** per worker: the longest it will wait before treating silence as a problem.
3. **On a missed deadline** — no heartbeat within the timeout, often after a few consecutive misses — the master marks the worker failed.
4. **The master acts**: reassign the failed worker's tasks to healthy nodes, remove it from the pool so the [load balancer](/citadel/interview/load-balancing) stops routing to it, alert an on-call engineer, and trigger a restart or a replacement instance.

## The one hard decision: interval and timeout

Both knobs trade the same way, against different costs.

- **Interval.** Shorter → failures detected sooner. Longer → less network and processing overhead from fewer messages.
- **Timeout.** Shorter → faster detection, but a sharply higher rate of **false positives**: a healthy node whose heartbeat was merely delayed by a congestion spike or a GC pause gets evicted and its work needlessly reassigned. Longer → few false positives, but a real failure goes unnoticed for longer, which directly worsens that component's [recovery time](/citadel/interview/cloud-disaster-recovery).

Setting them means estimating the inputs: the typical and worst-case network latency between worker and master, and the delays that CPU or I/O load can add on either end. A timeout shorter than the plausible worst-case latency guarantees false positives. And the cost of a false positive itself matters — if evicting a node and rebalancing its load is cheap, lean aggressive; if it is disruptive, lean lenient and add a second check before acting.

## What still bites

- **Push versus pull.** The classic heartbeat is **push** — the worker sends. The **pull** form is a health check — the monitor (or load balancer) sends a request and expects a timely reply. Both detect failure; pull also confirms the node can serve requests, not just that its heartbeat thread runs.
- **Payload.** A heartbeat can be a bare liveness ping, or carry a little status — current load, free memory — so the master can make smarter placement and routing decisions.
- **Network partitions.** Heartbeats detect an unreachable node, not just a dead one. A node alive but partitioned from the master looks failed *from the master's view* — and may still be serving clients on its side of the split, which is where consistency problems start.
- **Who monitors the monitor?** If the master dies, detection dies with it. The fixes add machinery: a standby master, **leader election** via a consensus protocol (Raft, Paxos) among candidate masters, or peer-to-peer heartbeating where nodes watch each other directly.

## It is a deadline, not a diagnosis

A heartbeat never tells you *why* a node went quiet — crash, overload, or partition all present the same. It converts an unanswerable question into a timeout you chose in advance, and every parameter is you deciding how long to wait before being wrong in one direction or the other. Systems that need to be sure pair heartbeats with a second signal — a health check, a quorum, a human — before doing anything destructive with the verdict.
