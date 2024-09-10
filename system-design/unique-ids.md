---
title: Globally Unique IDs - UUIDs, Ticket Servers, and Snowflake
description: Assigning every post, user, and transaction an ID that is unique across every server at once, ideally sortable by time and cheap to index. The requirements, five common approaches, and why large systems keep landing on Snowflake.
date: 2024-09-10
draft: false
slug: /system-design/unique-ids
tags:
  - System Design
  - Interview Prep
  - Distributed Systems
---

A single database can hand out IDs by incrementing a counter. Spread the work across a hundred servers taking thousands of requests a second each, and that counter becomes a bottleneck and a single point of failure. The problem is generating identifiers that never collide across the whole fleet, without every server stopping to ask a central authority.

This covers what large systems actually want from an ID, the five approaches in common use, and the trade-offs that make Twitter's Snowflake the default answer.

![The five IDs of a Snowflake-style layout side by side: a bar split into a sign bit, a 41-bit timestamp, a 10-bit machine ID, and a 12-bit per-millisecond sequence number](../images/uniqueId.png "A 64-bit Snowflake ID: sign bit, timestamp, machine ID, sequence number.")

## What the ID has to do

"Unique" is the floor, not the whole spec. A large distributed system usually wants:

- **Global uniqueness.** No collisions anywhere in the system, ever — two rows with the same ID means corrupted references.
- **Rough time ordering.** If IDs increase with creation time, you can sort posts or events chronologically straight from the primary key, with no separate timestamp column to index and sort on.
- **Numeric, and 64-bit.** Numbers index more compactly than strings and sort naturally when they carry a timestamp. 32 bits caps out near 4 billion values — not enough at scale. 128 bits (a full UUID) sorts and stores worse for no benefit here. 64 bits is the balance.
- **Scalable and low-latency.** Millions of IDs per second, with minimal delay on the request path.

## The five approaches

### UUID

A 128-bit value, usually written as 36 hex characters like `123e4567-e89b-12d3-a456-426614174000`. Any server generates one locally with no coordination, and the collision probability is negligible.

The costs: 128 bits is bulky to store and index, the randomness makes it a poor clustered-index key, standard versions carry no time ordering, and it isn't numeric.

### Database auto-increment

Let a relational database's auto-increment primary key assign IDs. Trivial to set up, guaranteed unique within the table, and the database's transaction handling sorts out concurrent requests.

It doesn't survive going distributed: multiple write nodes need coordination to avoid clashes, every ID costs a round trip to the database, and sequential IDs leak business data (a competitor reads your user count off a signup URL).

### Database ticket / segment servers

A scaling patch on auto-increment. An ID service asks the database for a *block* — "give me 1,000 IDs" — and serves that block from memory, going back only when it runs low.

This cuts database traffic by three orders of magnitude and reuses a well-understood mechanism. But a central database still reserves the ranges, a service instance that crashes loses the unused IDs in its current block unless it persisted them, and IDs from different instances interleave in time rather than sorting cleanly.

### Redis `INCR`

Use Redis's atomic `INCR` / `INCRBY` as the counter. It avoids a relational database and runs in memory, so it's faster than disk-backed options.

The trade-offs: a Redis deployment to operate and scale, IDs that don't sort across instances if you shard the counter (odds on one node, evens on another), and a persistence configuration that must be right or a restart rewinds the counter.

### Twitter Snowflake

An algorithm for 64-bit, roughly time-sortable IDs generated locally on each machine, open-sourced by Twitter and used by Discord among others. The 64 bits are laid out as:

- **1 sign bit** — left 0 so the number is always positive.
- **41 bits of timestamp** — milliseconds since a custom epoch, giving rough time ordering and about 69 years of range.
- **10 bits of machine ID** — often 5 for the data centre and 5 for the worker, keeping IDs distinct across generators.
- **12 bits of sequence** — incremented per ID within the same millisecond on the same machine, so $2^{12} = 4096$ IDs per machine per millisecond.

It's numeric, compact, roughly ordered, and needs no network call per ID. The costs: it depends on machine clocks — a backward clock jump can stall generation or, in the worst case, risk collisions — every worker needs a distinct machine ID assigned and managed, and the bit split is a fixed three-way trade between lifespan, machine count, and IDs per millisecond.

## Side by side

| Property | UUID | DB auto-increment | Ticket server | Redis `INCR` | Snowflake |
| --- | --- | --- | --- | --- | --- |
| Time-ordered | No | Within one table | Within a block | Within one instance | Roughly, by timestamp |
| Generated in-process | Yes | No (DB call) | No (service call) | No (service call) | Yes (local) |
| Single point of failure | No | Yes (primary DB) | Yes (DB / server) | Yes (Redis) | No |
| Index-friendly | Poor (large, random) | Good | Good | Good | Good |
| Uniqueness | Probabilistic | Per table | Global if ranges managed | Global if shards managed | Global if machine IDs unique |

## Picking one

Auto-increment is fine until you have more than one writer. UUIDs win when a server must generate an ID completely offline and ordering doesn't matter. Everything else — high volume, time ordering, numeric keys, no central dependency — points at Snowflake, which is why it keeps getting reimplemented. The choice is a ranking of uniqueness, sortability, latency, and operational cost for your system, not a single best answer. [Discord's message IDs](/citadel/system-design/discord) and [Twitter's tweet IDs](/citadel/system-design/twitter) are both Snowflake variants.
