---
title: Database Isolation Levels - Keeping Concurrent Transactions in Check
description: Dive into database isolation levels! Understand how they prevent issues like dirty reads, non-repeatable reads, and phantom reads using locks and MVCC, ensuring data consistency in a multi-user world.
date: 2024-04-15
draft: false
slug: /pensieve/database-isolation
tags:
  - interview
  - SDE
---

Hey everyone, and welcome back to the blog! If you've ever worked with databases, especially in environments where multiple users or processes are accessing and modifying data simultaneously, you've probably encountered the challenges of concurrency. How does a database ensure that one user's half-finished transaction doesn't mess up what another user is seeing or doing? The answer lies in **database isolation levels**.

Database isolation allows a transaction to execute as if there are no other concurrently running transactions. This is a critical concept for maintaining data integrity and consistency. Let's explore what these levels are, the problems they solve, and how databases achieve this isolation.

## The Chaos of Concurrency: What Can Go Wrong?

When multiple transactions run at the same time without proper isolation, several undesirable phenomena can occur, leading to inconsistent or incorrect data. The three main culprits are:

1. **Dirty Read:**
   * A transaction reads data that has been written by another concurrent transaction but has *not yet been committed*. If the first transaction rolls back, the data read by the second transaction becomes invalid ("dirty").
2. **Non-repeatable Read:**
   * A transaction reads the same data twice but finds that the data has been modified or deleted by another *committed* transaction in between the reads. So, the original read cannot be repeated to get the same result.
3. **Phantom Read:**
   * A transaction re-executes a query that returns a set of rows satisfying a certain search condition. It then finds that another *committed* transaction has inserted new rows (or deleted rows) that now satisfy that condition. These new "phantom" rows appear in the later read.

## The Four Standard Isolation Levels

To combat these issues, databases offer different isolation levels. These levels define the degree to which a transaction must be isolated from data modifications made by other concurrent transactions. There's generally a trade-off: higher isolation levels provide more consistency but can reduce concurrency and performance.

Here are the four standard levels, from lowest to highest:

### 1. Read Uncommitted

* **Description:** This is the lowest isolation level. In this mode, one transaction can see uncommitted changes made by other transactions.
* **Phenomena Allowed:** Dirty Reads, Non-repeatable Reads, and Phantom Reads are all possible (probably). Data modification can be read by other transactions before a transaction is committed.
* **Locking (Typical):** Generally uses no locks for reads, or very short-lived shared locks. Writes still acquire exclusive locks.

### 2. Read Committed

* **Description:** This level ensures that a transaction only reads data that has been committed.
* **Phenomena Allowed:**
  * Prevents Dirty Reads.
  * Non-repeatable Reads and Phantom Reads are still possible (probably).
  * Data modification can only be read after the transaction is committed.
* **Locking (Typical):** Writes acquire exclusive locks. Reads acquire shared locks but usually release them immediately after the read operation, or it might use MVCC.

### 3. Repeatable Read

* **Description:** This level guarantees that if a transaction reads a row multiple times, it will see the same data for that row each time.
* **Phenomena Allowed:**
  * Prevents Dirty Reads and Non-repeatable Reads.
  * Phantom Reads may still occur (probably).
  * Data read during the transaction stays the same as when the transaction starts.
* **Locking (Typical):** Reads acquire shared locks on all data they read and hold them until the transaction completes. Writes acquire exclusive locks. MVCC is also a common implementation.

### 4. Serializable

* **Description:** This is the highest and most strict isolation level. It aims to make concurrent transactions behave as if they were executed one after another (serially).
* **Phenomena Allowed:**
  * Prevents Dirty Reads, Non-repeatable Reads, and Phantom Reads.
  * Concurrent transactions are guaranteed to be executed in sequence.
* **Locking (Typical):** Often uses more extensive locking, such as range locks or predicate locks, to prevent new rows from being inserted into a range being read by a transaction, or relies on strict two-phase locking. Some databases use serializable MVCC.

The following diagram illustrates which phenomena are possible at each level:

| Isolation Level  | Dirty Read | Non-repeatable Read | Phantom Read |
| ---------------- | ---------- | ------------------- | ------------ |
| Serializable     | Impossible | Impossible          | Impossible   |
| Repeatable Read  | Impossible | Impossible          | Probably     |
| Read Committed   | Impossible | Probably            | Probably     |
| Read Uncommitted | Probably   | Probably            | Probably     |

## How is Isolation Achieved? Mechanisms Behind the Magic

Databases typically use two main mechanisms, often in combination, to implement these isolation levels: **Locking** and **Multi-Version Concurrency Control (MVCC)**.

### 1. Locking

Locks are used to control access to shared resources (like rows, tables, or pages).

* **S Lock (Shared Lock):**
  * If transaction A holds an S lock on some data, other transactions (like B) can also acquire an S lock to read the same data.
  * However, no transaction can acquire an X lock to modify the data until all S locks are released.
* **X Lock (Exclusive Lock):**
  * If transaction A holds an X lock on some data (meaning it intends to write), no other transaction can acquire any lock (neither S nor X) on that data. This prevents other transactions from reading or changing the data.

Different isolation levels use these locks with varying granularity and duration to prevent conflicts. For example, `Serializable` might hold locks for longer periods and on broader ranges of data.

### 2. MVCC (Multi-Version Concurrency Control)

MVCC is a more sophisticated technique that allows multiple transactions to access the same data concurrently without necessarily blocking each other. Instead of locking data, MVCC maintains multiple versions of data items. When a transaction reads data, it sees a snapshot of the data as it existed at a particular point in time (often the start of the transaction).

Let's illustrate how MVCC works for **Repeatable Read** with an example:

Imagine a `users` table with `user_id`, `balance`, and two hidden columns for each row: `transaction_id` (the ID of the transaction that created this version of the row) and `roll_pointer` (pointing to the previous version of the row).

1. **Transaction A (TxA) starts.** The database creates a "Read View" for TxA, noting that TxA started when, say, the latest committed `transaction_id` in the system was `200`. Let's assume TxA itself gets `transaction_id = 201`.
2. **Transaction B (TxB) starts shortly after.** It gets its own Read View, perhaps when the latest committed `transaction_id` is still `200`, and TxB gets `transaction_id = 202`.
3. **TxA modifies the balance for `user_id = 123` from 100 to 200.** The database creates a *new version* of the row for `user_id = 123` with `balance = 200` and `transaction_id = 201`. The `roll_pointer` of this new version points to the old row version (where `balance = 100` and `transaction_id = 200`).
4. **TxB reads the balance for `user_id = 123` (before TxA commits).** TxB's Read View was created when `transaction_id = 200` was the latest committed version. When TxB tries to read the row for `user_id = 123`, it sees the version created by TxA (`transaction_id = 201`). Since TxA (`201`) has not yet committed and is not part of TxB's Read View of visible transactions, TxB follows the `roll_pointer` to the previous version of the row—the one with `balance = 100` and `transaction_id = 200`.
5. **TxA commits.** The changes made by TxA (balance = 200) are now officially part of the database.
6. **TxB reads the balance for `user_id = 123` again.** Even though TxA has committed, TxB *still* uses its original Read View created when TxB started. So, TxB continues to see the version of the data with `balance = 100` because that was the state of the data visible to it when it began. This ensures that for TxB, the read is repeatable.

This MVCC mechanism allows readers not to block writers, and writers not to block readers, significantly improving concurrency.

## Key Takeaways

* Database isolation levels are crucial for managing concurrent transactions and preventing data anomalies like dirty reads, non-repeatable reads, and phantom reads.
* The four standard levels—Read Uncommitted, Read Committed, Repeatable Read, and Serializable—offer different trade-offs between consistency and performance.
* Isolation is primarily achieved through locking mechanisms (Shared and Exclusive locks) and Multi-Version Concurrency Control (MVCC).
* Choosing the right isolation level depends on the specific needs of your application, considering factors like data integrity requirements and acceptable performance overhead.

Understanding these concepts is vital for any developer or database administrator aiming to build robust and reliable applications.
