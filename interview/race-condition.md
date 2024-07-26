---
title: Race Conditions - Optimistic Locking and the Alternatives
description: When two operations touch shared data and the result depends on which ran first, you have a race. A worked example, optimistic locking with version numbers in detail, and the other tools — pessimistic locks, atomics, immutability, serialized access.
date: 2024-07-26
draft: false
slug: /interview/race-condition
tags:
  - System Design
  - Interview Prep
  - Concurrency
---

A **race condition** is when two or more threads or processes touch the same shared resource concurrently and the final result depends on the order their operations happen to interleave. When that order is not the one you assumed, you get corruption or an inconsistent state — and because it depends on timing, it often will not reproduce on demand.

## A worked example

An inventory counter starts at 10.

1. User A's request reads the counter — sees 10.
2. User B's request reads the counter — also sees 10.
3. A computes 10 − 1 = 9.
4. B computes 10 − 1 = 9.
5. A writes 9.
6. B writes 9.

Two items sold, inventory dropped by one. The correct value is 8. The read-modify-write of A and B interleaved, and B's read happened before A's write landed.

## Optimistic locking

**Optimistic concurrency control** assumes conflicts are rare. Rather than lock the row up front and block everyone, it lets all writers proceed and checks for a conflict at commit. The usual implementation uses a version number:

1. **Add a `version` column** to the table, incremented on every successful update.
2. **Read with the version.** User A reads their profile; `version = 5`.
3. **Do the work.** A edits the email.
4. **Write with a version check:**
   ```sql
   UPDATE user_profiles
   SET email = 'new@example.com', version = 6
   WHERE user_id = 123 AND version = 5;
   ```
5. **Commit or abort.** If `version` is still 5, the row updates and becomes version 6 — nobody changed it in between. If someone else committed first, `version` is now 6, the `WHERE` matches zero rows, and the update is a no-op.
6. **Handle the miss.** Zero rows means a conflict: tell the user to refresh and retry, or re-read the current data, re-apply the change, and try again.

Version numbers are preferred over timestamps because server clocks drift.

**Trade-off.** In low-conflict workloads this beats locking — no lock is held during the application's "thinking time", so throughput is higher. Under **high contention** it collapses: many clients racing for the last hotel room means one succeeds and the rest all fail the version check and retry, sometimes repeatedly, producing a retry storm and a stream of "try again" errors. It also pushes retry logic into the application.

## The other tools

- **Pessimistic locking** — assume conflicts are likely; take a row or table lock *before* using the data, and make others wait. Better under high contention; lower concurrency because locks are held longer. This is what [database isolation levels](/citadel/interview/db-isolation) enforce.
- **Atomic operations** — a single indivisible step that cannot interleave: Redis `INCR`, a compare-and-swap instruction. The counter example is fixed outright by `UPDATE inventory SET count = count - 1 WHERE id = X AND count > 0`.
- **Mutexes, semaphores, locks in code** — guard a critical section so only one thread executes it at a time.
- **Immutable data** — if shared state cannot be mutated after creation, there is nothing to race on; updates create new instances.
- **Serialized access** — funnel every write to a resource through one queue or one dedicated worker, so the operations simply cannot overlap.

## The fix is deciding how contended the data is

Every option here is the same trade at a different point: how much concurrency you give up to guarantee correctness. Optimistic locking gives up almost none and bets contention is low; pessimistic locking and single-worker serialization give up a lot and are safe under any load; atomic operations give up nothing but only cover operations the database or language makes atomic for you. Pick by measuring the contention on the specific resource — the [retry-with-backoff](/citadel/interview/data-systems) loop that makes optimistic locking usable is the same pattern used for transient distributed failures.
