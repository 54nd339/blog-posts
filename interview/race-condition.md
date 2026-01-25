---
title: The Concurrency Tango - Understanding and Taming Race Conditions
description: Ever had your software produce unexpected results under load? Race conditions might be the culprit! Let's explore what they are and dive deep into solutions like optimistic locking to keep your concurrent operations in harmony.
date: 2024-05-06
draft: false
slug: /pensieve/race-condition
tags:
  - interview
  - SDE
---

Hey everyone, and welcome back! Imagine you and a friend spot the last delicious cookie on a plate at the exact same moment. You both reach for it... who gets it? Or worse, do you both end up with half a cookie, or a crumbled mess? This, in a very simplified way, is the kind of scenario we can encounter in software systems with **race conditions**.

Race conditions are those sneaky, often hard-to-reproduce bugs that can wreak havoc in concurrent applications, leading to data corruption, inconsistent states, and a lot of head-scratching for developers. Today, I want to share some thoughts on what they are and take a closer look at a common strategy for dealing with them: optimistic locking.

## Understanding the "Race" in Race Condition

A race condition occurs when two or more threads or processes try to access and manipulate a shared resource (like a variable, a file, or a database record) at the same time, and the final outcome depends on the particular order in which their operations are executed. If that "lucky" order isn't the one you designed for, you get unexpected and incorrect results.

Let's consider a classic, simple example: updating a shared counter.

Imagine a counter in a database representing available inventory for a product, starting at 10.
* **User A** wants to buy 1 item. Their process reads the counter (sees 10).
* Simultaneously, **User B** wants to buy 1 item. Their process also reads the counter (sees 10).
* User A's process calculates the new count (10 - 1 = 9).
* User B's process also calculates the new count (10 - 1 = 9).
* User A's process writes 9 back to the database.
* User B's process writes 9 back to the database.

Oops! We sold 2 items, but the inventory only decremented by 1. The final inventory should be 8, not 9. This happened because the read-modify-write operations of User A and User B were interleaved in an unfavorable way.

## The Challenge: Ensuring Order in a Concurrent World

Concurrency—having multiple operations happen seemingly at the same time—is essential for modern applications. It improves performance, responsiveness, and resource utilization. However, with concurrency comes the challenge of managing access to shared resources to prevent these race conditions.

## Solution Spotlight: Optimistic Locking - "Hope for the Best, Prepare for a Retry"

One popular strategy for handling concurrent updates to shared resources, especially in database systems, is **optimistic locking** (also known as optimistic concurrency control). The core idea is that it allows multiple concurrent users to *attempt* to update the same resource, assuming that conflicts (where multiple users try to change the same data simultaneously) are relatively rare.

Instead of locking a resource upfront (which can block other users), optimistic locking lets everyone proceed with their changes and then checks for conflicts just before committing the update.

### How Optimistic Locking Works (Typically with Version Numbers)

Here's how it's commonly implemented using a version number:

1.  **Add a Version Column:** A new column, often named `"version"` or `"revision"`, is added to the database table for the resource you want to manage. This column will store a number that gets incremented with each successful update to that specific row.
2.  **Read Data (Including Version):** When an application needs to modify a resource (e.g., a user wants to update their profile), it first reads the current state of the resource from the database, *including its current version number*. Let's say User A reads their profile data, and the version is `5`.
3.  **Perform Business Logic & Prepare Update:** The application then performs the necessary business logic. User A changes their email address, for example.
4.  **Attempt to Write (with Version Check):** When the application is ready to save the changes, it tries to update the row in the database. Crucially, the `UPDATE` statement includes a condition that checks if the version number in the database *still matches the version number it originally read*. The application will also try to increment the version number by 1 as part of the update.
    So, User A's update might look something like:
    `UPDATE user_profiles SET email = 'new_email@example.com', version = 6 WHERE user_id = 123 AND version = 5;`
5.  **Database Validation & Commit/Abort:**
    * **If the versions match:** The database executes the update. The `WHERE` clause (`version = 5`) is true. The email is updated, and the version becomes `6`. This means no other transaction modified this user's profile since User A read it. The transaction commits successfully.
    * **If the versions *don't* match:** This means another transaction (say, from User B, or even User A from another session) managed to update the profile *after* User A read it but *before* User A tried to write their changes. The `WHERE` clause (`version = 5`) will now be false (because the version in the database might now be `6` or higher). The update will affect zero rows. The transaction typically aborts. The database might also have a specific validation check ensuring that the next version number correctly increments the current one.
6.  **Handle Failure (Retry or Inform User):** If the transaction aborts due to a version mismatch, the application knows a conflict occurred. It then needs to handle this, usually by:
    * Informing the user that the data was modified by someone else and they need to refresh and try again.
    * Or, programmatically re-reading the latest data (with the new version number), re-applying the intended changes if possible, and attempting the update again.

While version numbers are common, timestamps can also be used for optimistic locking, but version numbers are generally preferred because server clocks can be unreliable or drift over time.

### Pros and Cons of Optimistic Locking

**Pros:**

* **Higher Concurrency (Often):** It's usually faster than pessimistic locking (which locks resources *before* use) because it doesn't hold database locks for extended periods during the "thinking time" of the application or user. This can lead to better overall system throughput if conflicts are infrequent.

**Cons:**

* **Performance Degradation with High Contention:** The performance of optimistic locking drops dramatically when concurrency is high and conflicts are frequent.
    * Imagine many clients trying to book the last available hotel room simultaneously. Only one will succeed in its update attempt. All others will get a version check failure and will have to retry. This can lead to repeated retries and a very unpleasant user experience as users keep seeing "try again" messages.
* **Retry Logic Complexity:** The application needs to implement logic to handle update failures and potentially retry operations, which can add complexity.

## Other Strategies to Tackle Race Conditions

Optimistic locking is a powerful tool, but it's not the only one. Here are a few other common approaches:

* **Pessimistic Locking:** This strategy assumes conflicts are likely. It involves explicitly locking a resource before using it (e.g., database row locks, table locks). Other transactions must wait for the lock to be released. Good for high-contention scenarios but can reduce concurrency.
* **Atomic Operations:** Many databases and programming languages provide atomic operations (e.g., `INCR` in Redis, `compare-and-swap` instructions) that are guaranteed to execute as a single, indivisible unit, preventing interleaving.
* **Mutexes, Semaphores, Locks (In Code):** These are synchronization primitives used within application code to protect "critical sections" where shared variables or resources are accessed, ensuring only one thread can enter at a time.
* **Immutable Data Structures:** If data structures are designed to be immutable (their state cannot change after creation), race conditions related to modifying shared state are eliminated. Updates involve creating new instances.
* **Queues / Serialized Access:** Forcing all operations that modify a specific shared resource to go through a single queue or be handled by a single dedicated worker thread can serialize access and prevent races.

## Key Takeaways

* Race conditions are a common and often subtle problem in concurrent systems, leading to unpredictable behavior and data corruption.
* Optimistic locking, typically using version numbers, is a valuable strategy for managing concurrent updates by allowing operations to proceed and then checking for conflicts at commit time.
* While optimistic locking can offer better performance in low-conflict scenarios, it can struggle under high contention.
* A variety of techniques, from pessimistic locking and atomic operations to careful code-level synchronization, can be used to prevent or mitigate race conditions.

Choosing the right approach depends heavily on the specific characteristics of your application, the nature of the shared resources, and the expected level of concurrency and contention.
