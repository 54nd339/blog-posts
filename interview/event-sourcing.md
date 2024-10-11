---
title: Event Sourcing - Storing Changes Instead of State
description: Keep the log of what happened, not just the current row. How state is rebuilt by replaying events, what that buys in auditability and flexible read models, and the versioning and consistency costs it brings.
date: 2024-10-11
draft: false
slug: /interview/event-sourcing
tags:
  - System Design
  - Interview Prep
  - Event-Driven
---

The usual way to persist data keeps only the present. A customer changes their address, you `UPDATE` the row, and the old address is gone unless a separate audit table happened to catch it. **Event sourcing** inverts this: the durable record is the ordered list of every change that ever happened, and the current state is something you compute from it.

This covers how state is reconstructed from events, the worked contrast with CRUD, what the pattern is good for, and the costs — event versioning, eventual consistency, snapshotting — that come with it.

## The event store

The source of truth is an **append-only log** of immutable events, each a business fact — `OrderPlaced`, `QuantityChanged`, `PaymentReceived`. Once written, an event is never updated or deleted. Events are **sequenced** (incrementing IDs or precise timestamps) so their order is unambiguous, which gives the system **determinism**: the same events replayed from the same start always produce the same state.

![Two designs side by side. In the CRUD system, Bob's order goes through create, modify quantity, and pay, each an UPDATE that overwrites the Orders row; the database holds state but not the source of the changes. In the event-sourced system, the same actions append NewOrderEvent, ModifyOrderEvent, and OrderPaymentEvent to a sequenced event log, and a separate Order View is built by replaying those events.](../images/event-sourcing.png "CRUD overwrites the current row; event sourcing appends a fact per change and derives the current state by replay. The read-model view can always be rebuilt from the log.")

## CRUD versus events, worked

Bob orders 5 units, changes it to 6, then pays \$60.

**CRUD.** `INSERT` a row (`OrderID 1, Quantity 5, Status Created`). `UPDATE ... SET Quantity = 6` — the 5 is overwritten. `UPDATE ... SET Status = Paid`. A `SELECT` returns the latest row and nothing about how it got there.

**Event-sourced.** Append `NewOrderEvent {orderID: 1, quantity: 5}`, then `ModifyOrderEvent {orderID: 1, newQuantity: 6}`, then `OrderPaymentEvent {orderID: 1, amountPaid: 60.00}`. Nothing is overwritten; each fact is different in shape and all three remain.

## Rebuilding state: projections

Since the log only says what happened, the current state is derived by **replaying** an entity's events from the start (or from a snapshot) and applying each in turn. That derived state is materialised as a **read model** or **projection** — an `OrderView` kept in memory, a cache, or a database shaped for the queries it serves. It subscribes to new events and updates itself.

The projection does **not** need the event store's durability guarantees: if it is lost or corrupted, or you want a new one shaped differently, you rebuild it by replaying the log. Queries hit the projection, not the raw events.

## What it buys

- **A complete audit trail.** Every change is in the log, immutably and in order — you know not just the current state but exactly how it was reached, for compliance and for debugging.
- **Temporal queries.** Replay events up to any past moment to see the state as it was then.
- **Debugging by replay.** Reproduce a bug by replaying the exact event sequence into a test environment.
- **Many read models from one log.** Each projection is tuned for a different query need without touching the write path — which is exactly **CQRS** (command query responsibility segregation): the write model produces events, separate read models answer queries.
- **A natural fit for event-driven architecture.** The domain events can be published to a [message broker](/citadel/interview/message-queue) so other services react to them, decoupled.

## Where it is used

- **Content systems.** The New York Times stores every article, image, and byline published since 1851 as events, and builds projections — including denormalised content fed to Elasticsearch — from that log.
- **[Change data capture](/citadel/interview/data-systems).** Database inserts, updates, and deletes are read from the transaction log, turned into events, and pushed through Kafka to downstream sinks. The log becomes the channel, sometimes the store.
- **Microservice integration.** A `ShoppingCartService` emits `ItemAddedToCart`, `OrderCheckoutStarted`; a `FraudDetectionService`, `BillingService`, and `EmailService` each consume the stream and build their own view of just the data they need — loose coupling, with the events as the shared truth.

## The costs

- **Mindset and setup.** Modelling "sequence of changes" instead of "current state" is a real shift, and more code up front.
- **Event versioning.** Events are immutable, but their schema evolves. Renaming a field or splitting an event type needs a strategy — *upcasting* old versions to the current shape on read, or consumers that understand several versions.
- **Querying current state.** Replaying a long history on every read is slow, which is why projections are mandatory, not optional — and rebuilding a projection over a long history takes time.
- **Eventual consistency.** If projections update asynchronously, there is a lag between an event being stored and every read model reflecting it. That window has to be acceptable for the use case.
- **Snapshotting.** For an entity with thousands of events, periodically save its full state as a snapshot; reconstruction then loads the latest snapshot and replays only the events after it.
- **Idempotent handlers.** Consumers must tolerate an event being delivered more than once without double-applying its effect.

## It is worth it when history is part of the requirement

Event sourcing pays off for systems where "what happened" is itself valuable — strong auditability, historical analysis, complex state transitions, multiple divergent views of the same data. For a service that only ever needs the current value of a row, it is a large amount of machinery for a problem you do not have; plain [CRUD](/citadel/interview/db-models) with an audit column is the honest default.
