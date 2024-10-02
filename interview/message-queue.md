---
title: Message Queues - Decoupling, Delivery Semantics, and Patterns
description: A queue between two services lets the sender return immediately and the receiver work at its own pace. The core concepts, the at-most / at-least / exactly-once delivery guarantees, the brokers from IBM MQ to Kafka, and the patterns built on them.
date: 2024-10-02
draft: false
slug: /interview/message-queue
tags:
  - System Design
  - Interview Prep
  - Messaging
---

A service needs to send a notification, or absorb a spike of orders, without blocking on the work or overwhelming the thing that does it. Put a **message queue** between producer and consumer: the producer writes a message and moves on, the queue holds it, and the consumer takes it when ready. This covers the concepts, the delivery guarantees, how the technology evolved, and the patterns that queues enable.

## Why a queue

- **Decoupling** — the producer does not need to know where consumers are, whether they are up, or how they process. Components develop, deploy, and scale independently.
- **Asynchrony** — the producer returns immediately instead of waiting for the work to finish.
- **Buffering and persistence** — the queue absorbs bursts when producers outrun consumers, and durable queues keep messages on disk so a broker or consumer crash does not lose them.
- **Load levelling** — a spike becomes a backlog the consumers drain at a steady rate, instead of a thundering load.
- **Independent scaling** — add consumers to raise throughput; add brokers to raise capacity.

## Core concepts

- **Message** — payload plus headers (metadata).
- **Producer** / **consumer** — writes messages / reads and processes them.
- **Broker** — the software managing receipt, storage, routing, and delivery.
- **Queue (point-to-point)** — each message goes to *one* consumer from a pool.
- **Topic (publish-subscribe)** — each message goes to *every* subscriber; used for broadcast.
- **Exchange** (RabbitMQ) — sits before the queues and routes messages to them by rules and routing keys, per its type (direct, topic, fanout).

## Delivery semantics

The guarantee on whether a message can be lost or duplicated:

- **At-most-once** — never redelivered, may be lost on failure. Fine for metrics and non-critical logs where a few missing points do not matter.
- **At-least-once** — never lost, may be delivered more than once (a failed acknowledgement triggers a resend). Workable when consumers are **idempotent** — a unique key lets them reject the duplicate on write. The common default.
- **Exactly-once** — delivered and processed precisely once. The most desirable and the hardest, with a real cost in throughput and complexity. Reserved for payments, trading, accounting, where duplicates are unacceptable and downstream systems are not idempotent.

## How brokers evolved

- **IBM MQ (1993)** — one of the first commercial messaging middlewares, entrenched in finance for reliable enterprise messaging.
- **RabbitMQ** — an AMQP broker with the exchange model: producers publish to an exchange, which routes to bound queues by type and routing key.
- **Apache Kafka (2011)** — reframed the problem as a **distributed event streaming platform**. A topic is a partitioned, replicated, append-only **log**; consumers in a **group** pull at their own pace, and messages are retained by time rather than deleted on read — which is what makes replay possible. Optimised for write throughput.
- **Apache Pulsar** — a two-layer design: stateless broker (serving) layer over a separate storage layer (Apache BookKeeper), plus native **tiered storage** offloading old messages to object storage. More cloud-native scaling and multi-tenancy than early Kafka.

## Patterns

![Six messaging patterns as diagrams. Async request-reply: client POSTs, gets HTTP 202, then polls a status endpoint until 200. Publisher-subscriber: a publisher sends through a broker to several subscribers. Claim check: a large payload goes to a store, only a reference travels the queue. Priority queue: higher-priority messages are consumed first. Saga: a chain of local transactions each with a compensating undo. Competing consumers: several consumers pull from one queue for throughput.](../images/message-queue.png "Patterns queues enable. Saga coordinates a distributed transaction with compensating steps; competing consumers trade message ordering for parallel throughput. Source: Microsoft Learn.")

- **Async request-reply** — client calls an API, which enqueues the work and returns `202 Accepted`; the client polls a status endpoint or gets a callback for the result. For long backend tasks.
- **Publish-subscribe** — publishers and subscribers know only the topic, not each other.
- **Claim check** — a large payload goes into external storage (S3, a database); only a reference travels the queue, so the broker is not strained.
- **Priority queue** — higher-priority messages are consumed ahead of lower.
- **Saga** — a distributed transaction as a sequence of local transactions, each with a **compensating transaction** to undo it if a later step fails. Queues carry the coordination.
- **Competing consumers** — many consumer instances pull from one queue in parallel for throughput, giving up ordering across consumers.

## The queue makes failure a backlog, not an outage

The core value is turning tight coupling into a buffer: when the consumer is slow or down, messages pile up instead of errors propagating back to the user. That only holds if the consumer is idempotent (because at-least-once *will* redeliver) and the backlog is bounded (an unbounded queue just defers the collapse). The [event sourcing](/citadel/interview/event-sourcing) and [data pipeline](/citadel/interview/data-pipelining) posts build directly on a retained log; the [saga](/citadel/interview/orchestration) coordination is its own topic.
