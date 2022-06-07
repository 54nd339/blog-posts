---
title: Transaction Processing Systems - Isolation, Distribution, and Security
description: Transactions past the single node - flat and nested transaction structures, the SQL isolation levels and the read anomalies they permit, multiversion concurrency control, two-phase and three-phase commit for distributed atomicity, the Saga alternative, and the security services that protect transactions in e-commerce.
date: 2022-06-07
draft: false
slug: /dbms/transaction-processing
tags:
  - Databases
  - Transactions
  - Distributed Databases
---

The [transactions post](/citadel/dbms/transaction-proc) covers a single database: ACID, serializability, locking, logging. Real transaction processing systems — the ones behind ATMs, airline booking, e-commerce checkout — add three more layers. Transactions get *structure* (savepoints, nesting, and the SQL isolation levels that trade consistency for concurrency). Transactions *span machines*, needing a commit protocol to stay atomic across them. And transactions *carry value*, needing authentication, encryption, and payment-specific protocols.

This post is those three layers, on top of the single-node foundation.

## Transaction structure

A **flat transaction** is the default: one begin, one end (commit or abort), everything in between atomic. Two ways to add structure:

- **Savepoints** — mark a point mid-transaction; on a later error, roll back to that savepoint instead of aborting the whole thing.
- **Multiple transactions** — a long business process (book flight, then hotel, then car) is split into several short flat transactions, because holding locks across the whole thing is impractical. Consistency *across* them becomes the application's problem, typically handled by [Sagas](#sagas).
- **Nested transactions** — a transaction composed of child sub-transactions. A child's commit is provisional: its changes become permanent only if the top-level parent commits. Children run concurrently, and a child's failure doesn't necessarily abort the parent, which can handle it.

For abstract objects, operations can be **semantically commutative** even when they touch the same bytes — two `increment()` calls on a counter commute — allowing more concurrency than raw read/write locking sees. When a physical rollback is impractical (a committed sub-step of a long workflow), a **compensating operation** logically reverses it.

## Concurrency, revisited

A **schedule** is serializable if its outcome matches some serial order; **conflict serializability** is the practical form (see [transactions](/citadel/dbms/transaction-proc)). Two more properties matter for systems:

- **Recoverability** — a transaction commits only after every transaction whose data it read has committed. Otherwise it could commit on data that later rolls back.
- **Strictness** — no transaction may read or write an item until the last transaction that wrote it has committed or aborted. Strict schedules avoid cascading aborts and make recovery simpler.

Two philosophies:

- **Pessimistic** — assume conflicts are likely; lock before accessing, wait if blocked. A **lock manager** grants and releases locks from a lock table; **granularity** (database / table / page / row) trades concurrency against locking overhead; **deadlocks** are found with a waits-for graph and broken by aborting a transaction.
- **Optimistic** — assume conflicts are rare; run without locks and validate at commit time, rolling back if another transaction modified something you read.

### SQL isolation levels

The standard defines four, trading isolation for concurrency by permitting successively fewer read anomalies:

| Level | Dirty read | Non-repeatable read | Phantom read |
| --- | --- | --- | --- |
| READ UNCOMMITTED | possible | possible | possible |
| READ COMMITTED | no | possible | possible |
| REPEATABLE READ | no | no | possible |
| SERIALIZABLE | no | no | no |

A **dirty read** sees another transaction's uncommitted write. A **non-repeatable read** gets different values reading the same row twice. A **phantom read** gets different *rows* re-running the same query, because another transaction inserted matching rows. READ COMMITTED is a common default.

### Granular and index locking

**Intention locks** make hierarchical locking efficient: before an S or X lock on a row, a transaction takes an intention lock (`IS`, `IX`, `SIX`) on the ancestor table and page, signalling its intent so conflicting coarse-grained locks are blocked without scanning every row. **Index (key-range) locking** locks parts of an index to stop phantoms at SERIALIZABLE.

### MVCC

**Multiversion concurrency control** keeps several versions of each item. A reader sees a consistent snapshot as of a point in time (usually its start); writers create new versions rather than overwriting. Readers never block writers and writers never block readers — a large concurrency win. It's how [PostgreSQL](/citadel/tech/postgres) and Oracle work.

To tune concurrency, developers keep transactions short, access data in a consistent order across transactions (fewer deadlocks), and pick the lowest isolation level the task tolerates.

## Recovery in a TPS

Same foundation — [write-ahead logging](/citadel/dbms/transaction-proc), with UNDO/REDO for immediate modification and REDO-only for deferred. The widely used industrial algorithm is **ARIES**. For **media failure** (the disk is gone), recovery restores the most recent **archive backup**, then rolls forward by redoing every committed transaction since the backup, using the archived and online logs.

## Distributed transactions

When a transaction updates data at several autonomous sites, network latency and partial failure make coordination the hard problem. A **TP monitor** (historically CICS, Tuxedo; today folded into application servers and message brokers) provides process management, transaction coordination, inter-process communication, name services, and security.

### Two-phase commit

The standard protocol for **global atomicity** — all sites commit or all abort — run by a coordinator (**TM**):

1. **Prepare phase** — the coordinator sends `PREPARE` to every participant. Each decides whether it can durably commit its part; if yes, it force-writes a `PREPARE` log record (entering the *prepared* state) and replies `VOTE_COMMIT`; if no, it replies `VOTE_ABORT` and aborts locally.
2. **Decision phase** — if all votes are `VOTE_COMMIT`, the coordinator force-writes `GLOBAL_COMMIT` and sends `COMMIT`; on any `VOTE_ABORT` or timeout, it force-writes `GLOBAL_ABORT` and sends `ABORT`. Participants force-write the outcome, act, and acknowledge.

2PC is **blocking**: if the coordinator fails after starting phase 2, participants that voted `VOTE_COMMIT` are stuck in the prepared state — they can't unilaterally commit or abort — until it recovers. **Three-phase commit** inserts a *pre-commit* phase to make this non-blocking, but only under assumptions (no network partitions, bounded failures) that limit its real-world use.

Communication between sites uses synchronous **RPC**, peer-to-peer, or asynchronous **message queuing** (which decouples components and tolerates outages, at the cost of transactional-messaging complexity). Data is **partitioned**, **replicated**, or **federated** across sites. Distributed deadlocks — cycles spanning sites — are found with a global waits-for graph or timeouts; global serialization uses distributed 2PL or timestamp ordering. The [distributed databases post](/citadel/dbms/distributed-database) covers fragmentation, allocation, and distributed query processing.

### Sagas

When strict 2PC is too slow or infeasible (microservices, third-party services outside your control), a **Saga** replaces it: a sequence of local transactions, each committing independently, with a **compensating transaction** for each. If step *k* fails, the compensations for steps 1…*k*−1 run in reverse to undo them. The result is **eventual consistency**, not strict atomicity — the same trade [event sourcing](/citadel/interview/event-sourcing) and [CAP-bound systems](/citadel/interview/latency-consistency) make.

Replication forces a similar choice: **eager (synchronous)** replication propagates updates within the committing transaction (strong consistency, slow writes); **lazy (asynchronous)** replication propagates afterward (fast writes, stale [replica reads](/citadel/interview/read-replica-pattern), eventual consistency); **quorum** protocols require reads and writes to reach a majority of replicas.

## Security in transactions

Three base services: **authentication** (who are you — passwords, MFA, tokens, biometrics, certificates), **authorization** (what may you do — ACLs, [RBAC](/citadel/interview/identity-management), ABAC), and **encryption** (confidentiality of data at rest and in transit — AES).

Cryptographic tools: [**digital signatures**](/citadel/cyber-security/digital-signatures) give authenticity, integrity, and non-repudiation (signed with the sender's private key, verified with the [public key](/citadel/cyber-security/public-key)); a **PKI** of certificate authorities binds public keys to identities; **Kerberos** authenticates via symmetric keys and a trusted key distribution centre.

For e-commerce specifically:

- [**SSL/TLS**](/citadel/interview/https) secures web traffic (HTTPS) — encryption, server authentication via a CA-issued certificate, and message integrity.
- **Single sign-on** — Microsoft Passport was an early attempt; today it's **SAML**, **OAuth 2.0** (delegated authorization), and **OpenID Connect** (authentication on top of OAuth).
- **Card data** — **PCI DSS** governs handling; **tokenization** replaces the card number with a non-reversible token while a processor holds the real value; end-to-end encryption protects it in flight.
- The **SET protocol** (Visa/Mastercard) was a comprehensive scheme whose **dual signature** linked the order info (for the merchant) and payment instructions (for the bank) so each party saw only its half while both could verify the link. Too complex to adopt; superseded by TLS plus 3-D Secure.
- **Goods atomicity** (buyer gets goods iff merchant gets paid), **certified delivery**, and **escrow** address trust beyond payment.
- **E-cash** aims at digital cash with anonymity; David Chaum's **blind signatures** let a bank sign a digital coin without seeing its serial number, so the coin is anonymous yet verifiable and protected against double-spending.

## The one idea to keep

Every layer here is the single-node ACID guarantee stretched until something has to give. Isolation levels trade consistency for concurrency on purpose. 2PC preserves atomicity across machines but blocks on coordinator failure. Sagas drop atomicity for availability and rebuild correctness with compensation. The security layer is orthogonal but non-negotiable: a transaction that's perfectly atomic and isolated is still worthless if an attacker can forge or read it.
