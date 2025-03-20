---
title: Data Systems - CDC, Retries with Backoff, and Lock Modes
description: Three techniques that keep a data tier correct under load. Change data capture propagates every write, retries with exponential backoff and jitter survive transient failures, and database lock modes are what concurrency control is built from.
date: 2025-03-20
draft: false
slug: /interview/data-systems
tags:
  - System Design
  - Interview Prep
  - Databases
---

Three problems show up in every data-intensive system: getting every change out of a database and into the systems that need it, retrying a failed call without making things worse, and letting many transactions touch the same rows without corrupting them. This covers the standard answer to each — change data capture, retries with backoff, and the database's lock modes.

## Change data capture

**Change data capture (CDC)** propagates every insert, update, and delete from a source database to downstream systems in near real time — the deltas, not periodic full copies. It feeds real-time analytics, keeps warehouses and search indexes and caches in sync, lets one microservice react to another's data changes, and produces an audit trail of every modification.

Three implementations:

- **Timestamp-based** — poll for rows where `last_updated` moved. Simple, but misses deletes and adds query load to the source.
- **Trigger-based** — database triggers write change rows to a side table. Catches everything, at the cost of a trigger firing on every write.
- **Log-based** (the usual choice) — read the database's own transaction log: MySQL's binlog, Postgres's write-ahead log. A tool like Debezium parses the log and emits an ordered event stream. Near-zero load on the source and every committed change captured in commit order; the cost is a database-specific log format and non-trivial parsing.

The common architecture pipes those events through [Kafka](/citadel/interview/message-queue): the connector publishes changes, and databases, caches, and indexes each consume the stream independently. It is the collect stage of a [data pipeline](/citadel/interview/data-pipelining).

## Retries with backoff

Transient failures — a brief network blip, a momentarily overloaded service — often succeed on a second attempt. Two rules keep retries from causing harm:

- **Idempotency first.** A retried operation must be safe to apply more than once. Non-idempotent writes need a dedup key or an idempotency token so the second attempt is a no-op.
- **Back off, with jitter.** Retrying immediately, from many clients at once, turns a small stumble into a stampede. Exponential backoff spaces attempts out — 1s, 2s, 4s. Adding jitter breaks up the synchronised retry wave:

```
delay = min(cap, base * 2 ** attempt) + random_jitter
```

Also bound the number of attempts and the maximum delay, and stop retrying on errors that are clearly permanent (a `400`, a validation failure) rather than transient. Amazon's engineering guidance treats "timeouts, retries, and backoff with jitter" as a single required pattern. The [optimistic locking](/citadel/interview/race-condition) retry loop is the same idea applied to write conflicts.

## Scaling the data tier

The database is the usual bottleneck. The strategies, each with its own post where it goes deep:

| Strategy | Buys | Costs |
| --- | --- | --- |
| Indexing | fast reads, no full scans | slower writes, storage |
| Materialized views | fast complex/repeated queries | staleness until refresh |
| Vertical scaling | simplicity | a hard ceiling, cost, one box |
| [Read replicas](/citadel/interview/read-replica-pattern) | read throughput | replication lag, read/write routing |
| [Sharding](/citadel/interview/data-sharding) | read *and* write scale | cross-shard joins, re-sharding pain |
| [Caching](/citadel/interview/caching) | read latency and load | invalidation, [failure modes](/citadel/interview/cache-pitfalls) |
| Denormalization | fewer joins on hot reads | redundancy, inconsistency risk |

The [scalability](/citadel/interview/scalability) post sequences these — indexing and caching before replicas, replicas before sharding.

## Database lock modes

When transactions contend for the same data, locks enforce the chosen [isolation level](/citadel/interview/db-isolation). Two axes: how much is locked, and in what mode.

**Granularity** trades concurrency against bookkeeping:

- **Table** — one transaction on the table at a time. Cheap to manage, minimal concurrency.
- **Page** — a disk page (many rows). Fewer locks; unrelated rows on the same page still contend.
- **Row** — only the touched rows. Maximum concurrency; the most locks to track.

**Modes** govern who else can hold a lock on the same resource:

- **Shared (S)** — held for reads. Many transactions can hold S on one resource at once; blocks any X.
- **Exclusive (X)** — held for writes (`INSERT`/`UPDATE`/`DELETE`). One holder, and it blocks all S and X.
- **Update (U)** — taken when a transaction will read then possibly write. Compatible with S but not with another U or X, so two "read-then-update" transactions cannot both hold it — which prevents the classic deadlock of both trying to upgrade S to X. Promotes to X if the write happens.
- **Schema stability (Sch-S) / schema modification (Sch-M)** — Sch-S is held by anything depending on a stable schema (query compilation) and blocks DDL; Sch-M is the exclusive lock a DDL statement (`ALTER TABLE`, `CREATE INDEX`) takes, blocking everything else on the object.
- **Bulk update (BU)** — used by bulk loads to allow parallel insert threads into an unindexed table while excluding ordinary transactions for the duration.
- **Key-range (predicate)** — under serializable isolation, locks a *range* of index values matching a `WHERE` predicate, not just existing rows, so no other transaction can insert a row that would become a **phantom**.

**Deadlocks** are the price: two transactions each waiting on a lock the other holds. The database detects the cycle, picks a victim, and rolls it back — so any transaction can fail with a deadlock error and must be retryable.

## The three share one theme

CDC turns writes into an ordered, replayable stream; backoff spaces retries so a recovering system is not re-buried; lock modes serialise just enough of two transactions to keep them correct. Each is a way of imposing order on concurrent change without forcing everything through a single point — which is the core problem of every data tier that has outgrown one machine.
