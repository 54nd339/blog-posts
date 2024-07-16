---
title: Orchestration vs Choreography - How Microservices Collaborate
description: A business process spanning several services can be run by a central coordinator or by services reacting to each other's events. What each style costs in coupling, visibility, and failure handling, and when to use which.
date: 2024-07-16
draft: false
slug: /interview/orchestration
tags:
  - System Design
  - Interview Prep
  - Microservices
---

In a monolith, the steps of "process an order" are function calls in one process. Split those steps across [independent services](/citadel/interview/cloud-native) and something has to sequence them: check inventory, take payment, arrange shipping, in that order, handling failure at each step. There are two ways to do it — a central coordinator, or services that react to each other — and they trade off differently.

## Orchestration: a central coordinator

An **orchestrator** service owns the workflow. It calls each participant in turn and holds the logic for ordering, transaction management, and error handling. An `OrderOrchestrator` calls `InventoryService`, then `PaymentService`, then `ShippingService`, and decides what to do if `PaymentService` fails.

- **Reliable and visible** — the whole workflow is in one place to read, monitor, and change; compensating logic for a failed step lives there too.
- **Easy to extend** — adding a step often means editing only the orchestrator.
- **A bottleneck and a single point of failure** — every workflow runs through the orchestrator, so its capacity caps throughput and its outage halts the process. It must be made highly available.
- **Risk of a "god service"** — the orchestrator accumulates knowledge of every participant, which is tighter coupling than microservices are supposed to have.

Netflix Conductor is a well-known orchestrator.

## Choreography: services react to events

No coordinator. Each service publishes events and subscribes to the ones it cares about. `OrderService` emits `OrderCreated`; `PaymentService` consumes it, charges the card, emits `PaymentProcessed`; `ShippingService` consumes *that* and ships. The workflow is an emergent property of who listens to what.

- **Loose coupling** — a service knows only its events, not the other services. Strong autonomy.
- **Independent scaling and fault isolation** — one service failing does not necessarily stop unrelated parts.
- **The end-to-end flow is nowhere** — no single place describes the whole process; you reconstruct it by tracing which service listens to which event.
- **Harder to debug and to make transactionally correct** — following a request across event-driven hops needs distributed tracing, and compensating transactions across a choreographed flow are fiddly.

## The protocols underneath

Either style still needs services to communicate:

- **Synchronous** — [REST](/citadel/interview/rest-api) from a gateway to a service, or [gRPC](/citadel/interview/grpc) for performance-critical internal calls.
- **Asynchronous** — a [message queue or event streaming platform](/citadel/interview/message-queue) (Kafka, RabbitMQ). This is the backbone of choreography: publish an event, interested services consume it, and no two services need to be up at the same moment.

## Choosing

- **Complex multi-step workflows with strict ordering and transactional integrity** → orchestration, for the control and the single place to reason about it.
- **Loose coupling and service autonomy as the priority** → choreography.
- **Latency-sensitive interactions** → choreography avoids the central hop; orchestration adds one.

Large systems commonly do both: orchestration for the critical, order-sensitive workflows (checkout), choreography for decoupled reactions (send a receipt email, update a recommendation model). The [saga pattern](/citadel/interview/message-queue) — a sequence of local transactions each with a compensating undo — can be run either way, and is how distributed transactions are handled without a global lock.

## The choice is where you put the workflow knowledge

Orchestration concentrates the "what happens next" logic in one service you can read top to bottom, and pays for it with a central dependency. Choreography spreads that logic across the participants as event subscriptions, and pays for it with a process that exists only in the aggregate. Neither removes the coordination problem — they relocate it, to a place that is either a bottleneck or invisible.
