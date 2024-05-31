---
title: Database Isolation Levels - The Anomalies They Trade Away
description: An isolation level is a contract about which concurrency anomalies a transaction may see. The three anomalies, the four standard levels ordered by strictness, and how locking and MVCC actually enforce them.
date: 2024-05-31
draft: false
slug: /interview/db-isolation
tags:
  - System Design
  - Interview Prep
  - Databases
---

Two transactions run at the same time. One is halfway through updating a balance; the other reads it. What should the reader see — the old value, the new one, or an error? The database's answer depends on its **isolation level**: a contract stating which anomalies a transaction is allowed to observe, and by implication how much concurrency the database will give up to prevent them.

This covers the three anomalies the standard levels are defined against, the four levels from loosest to strictest, and the two mechanisms — locking and MVCC — that implement them.

## The three anomalies

- **Dirty read** — a transaction reads a row another transaction has written but not committed. If that writer rolls back, the reader acted on a value that never existed.
- **Non-repeatable read** — a transaction reads a row, and a second read of the *same row* returns different data because another transaction committed an update or delete in between.
- **Phantom read** — a transaction runs a query with a `WHERE` condition twice, and the second run returns extra rows because another transaction committed inserts that match the condition.

## The four levels

Higher isolation prevents more anomalies and permits less concurrency.

| Level | Dirty read | Non-repeatable read | Phantom read |
| --- | --- | --- | --- |
| Read Uncommitted | possible | possible | possible |
| Read Committed | prevented | possible | possible |
| Repeatable Read | prevented | prevented | possible |
| Serializable | prevented | prevented | prevented |

- **Read Uncommitted** — reads take no lock (or a fleeting one); writes still take exclusive locks. You can see uncommitted data.
- **Read Committed** — a read only returns committed data. Typically the read lock is released immediately after the read, or the level is implemented with MVCC. The default in PostgreSQL, Oracle, and SQL Server.
- **Repeatable Read** — read locks are held to the end of the transaction, or a consistent snapshot is used, so any row read once reads the same again. New rows matching a range query can still appear. MySQL's InnoDB default.
- **Serializable** — the result must equal *some* serial order of the transactions. Enforced with range/predicate locks that stop inserts into a scanned range, strict two-phase locking, or a serializable variant of MVCC.

## Locking

Locks gate access to a row, page, or table in one of two modes:

- **Shared (S)** — held for reading. Several transactions can hold S on the same data at once, but nobody can take an X lock until they all release.
- **Exclusive (X)** — held for writing. One holder, and it blocks every other lock, S or X, so no one else reads or writes that data.

Levels differ in how broadly and how long they hold these. Serializable holds them longer and over ranges; Read Committed holds read locks for an instant.

## MVCC

Multi-version concurrency control avoids blocking by keeping several versions of each row. A reader sees a **snapshot** as of a point in time — usually its transaction's start — so readers never block writers and writers never block readers.

Each row version carries two hidden fields: the `transaction_id` that created it, and a `roll_pointer` to the previous version. Walk through Repeatable Read:

1. **TxA starts.** It gets a *read view* fixing which transactions are visible to it; say the latest committed ID is `200`, and TxA is `201`.
2. **TxB starts** just after, with its own read view — latest committed still `200`, TxB is `202`.
3. **TxA updates** `user_id = 123`, balance `100 → 200`. The database writes a new version of the row with `balance = 200`, `transaction_id = 201`, its `roll_pointer` aimed at the old version (`balance = 100`, `transaction_id = 200`).
4. **TxB reads** row `123`, before TxA commits. The newest version was made by `201`, which is not in TxB's read view, so TxB follows `roll_pointer` back to the `200` version and sees `balance = 100`.
5. **TxA commits.**
6. **TxB reads** row `123` again. It still uses its original read view, so it still resolves to the `200` version — `balance = 100`. The read repeated.

## Picking a level is picking what you can tolerate

The default (Read Committed almost everywhere, Repeatable Read on MySQL) is right for most work. Raise to Serializable only where a wrong interleaving corrupts something that matters — a double-spend, a broken invariant across rows — and be ready for more aborts under contention, because the database now refuses interleavings it cannot prove serial. Where you need finer control than a level gives, reach for explicit [row locks or optimistic concurrency](/citadel/interview/race-condition); the [lock-mode taxonomy](/citadel/interview/data-systems) behind these levels — update locks, key-range locks — is its own topic.
