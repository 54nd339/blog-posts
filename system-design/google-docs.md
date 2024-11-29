---
title: Designing Google Docs - Real-Time Collaboration with OT and CRDTs
description: "How multiple people edit one document at once without it descending into chaos: a WebSocket layer, a message queue for durability, a collaboration engine, and the two families of algorithm — Operational Transformation and CRDTs — that keep every copy consistent."
date: 2024-11-29
draft: false
slug: /system-design/google-docs
tags:
  - System Design
  - Interview Prep
  - Distributed Systems
---

The hard problem in a collaborative editor is that two people can change the same sentence at the same time, and both changes must survive, apply in any order, and leave every screen showing the same text. This is a design for that — the moving parts, and the conflict-resolution algorithms at the centre.

## The architecture

- **Clients** — browsers and mobile apps where the document is viewed and edited.
- **WebSocket server** — clients send editing **operations** (insert a character, delete a range, apply formatting) over a persistent bidirectional [WebSocket](/citadel/interview/socket-program), which is also how updates come back. A request/response protocol can't push edits as they happen.
- **Message queue (Kafka, RabbitMQ)** — incoming operations are pushed onto a queue: a durable buffer that decouples fast edit ingestion from slower processing and guarantees no operation is lost if a downstream service restarts.
- **File operation server** — the collaboration engine. It consumes operations from the queue, applies them to the authoritative document, runs the collaboration algorithm to produce **transformed operations** that stay consistent across clients, and broadcasts the result back through the WebSocket server.
- **Storage**, in three parts:
  - **Metadata store** — title, owner, timestamps, permissions, version-history pointers.
  - **Content store** — the document body, as periodic snapshots or a form rebuildable from operations.
  - **Operations log** — every operation ever applied. It powers version history, replay, and auditing, and the collaboration algorithms are built on it.

## Conflict resolution

Three approaches to merging concurrent edits without data loss:

**Operational Transformation (OT).** Concurrent operations are *transformed* against each other so they can be applied in any order on any client and converge to the same state. When an operation reaches the server, it's transformed against the concurrent operations already processed, then applied and broadcast. Google Docs uses OT. It is hard to implement correctly once the operation set is rich — text, formatting, comments, tables all interacting.

**Differential Synchronization.** Rather than transforming operations, clients and server periodically exchange **diffs** of their document versions and merge them. Simpler in some ways, but less common for rich text than OT or CRDTs.

**Conflict-free Replicated Data Types (CRDTs).** Data structures designed so concurrent updates to different replicas **merge automatically and mathematically**, with a guarantee that all replicas converge — no server-side conflict-resolution logic needed. Increasingly used for collaborative editing, and a natural fit for decentralised or offline-capable systems.

## The other requirements

- **Scale** — every tier (WebSocket servers, queue, operation servers, databases) scales horizontally to millions of concurrent documents.
- **Latency** — edits must appear near-instantly, which constrains network and processing design.
- **Durability** — no lost edits through server failure, via the queue plus replicated databases.
- **Version history** — free, given the operations log.
- **Presence** — showing who else is in the document and where their cursors are, over the same real-time channel.

## The takeaway

The architecture is standard real-time-system plumbing — WebSockets in, a queue for durability, workers, replicated storage. What makes a collaborative editor its own problem is the merge: OT keeps the server authoritative and transforms operations as they arrive; CRDTs push the merge into the data structure itself so any replica can do it. The [operations log](/citadel/interview/event-sourcing) underneath is the same idea as event sourcing — the document is a fold over its edits.
