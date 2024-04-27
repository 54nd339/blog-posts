---
title: NoSQL and MongoDB - The Four Data Models and the BASE Trade-off
description: "NoSQL covers four distinct data models - document, key-value, wide-column, and graph - that trade SQL's joins and strict schema for horizontal scale and flexibility, often under BASE rather than ACID. Then a closer look at MongoDB: BSON documents, sharding, replica sets, and when to embed versus reference."
date: 2024-04-27
draft: false
slug: /tech/mongo
tags:
  - Tools
  - Databases
  - NoSQL
---

For decades the answer to "where does the data go" was a relational database, and it was a good answer. NoSQL didn't arrive because relational databases stopped working — it arrived because some workloads wanted a different set of trade-offs: a schema that changes without a migration, storage that scales by adding cheap machines rather than a bigger one, and access patterns that don't need joins.

"NoSQL" ("not only SQL") isn't one thing. It's four fairly different data models with one shared instinct: give up some of what relational databases guarantee to get scale and flexibility back. This post walks the four models, the consistency shift they usually come with, and then MongoDB in particular.

## ACID versus BASE

Relational databases are built around **ACID** transactions — atomicity, consistency, isolation, durability — which together promise that a transaction either fully happens or doesn't, and that concurrent transactions don't corrupt each other.

Many NoSQL systems relax that in favour of **BASE**:

- **Basically available** — the system answers every request, even if the answer is an error or slightly stale data.
- **Soft state** — the data can change on its own between reads, because replicas are still converging.
- **Eventual consistency** — given no new writes, all replicas converge to the same value in time.

This is the [CAP trade-off](/citadel/interview/latency-consistency) made concrete: when the network partitions, these systems keep serving (availability) at the cost of every node agreeing right now (strong consistency). Whether that's acceptable depends entirely on the data — a shopping cart, fine; a bank balance, not.

## The four models

**Document** — stores self-contained documents (JSON, BSON, or XML), each free to have its own shape. Good for content management, product catalogs, user profiles, mobile app backends. MongoDB is the leading example; [Discord stored its early messages](/citadel/system-design/discord) in MongoDB before outgrowing it. Managed options: Azure Cosmos DB, AWS DocumentDB.

**Key-value** — the simplest: a unique key maps to an opaque value. Caching, session storage, user preferences, real-time lookups. [Redis](/citadel/tech/redis) and DynamoDB.

**Wide-column** — tables of rows, but each row can have a different set of columns, grouped into column families. Built for huge datasets with heavy write throughput: event logs, time-series, recommendation feeds. Cassandra, Google Bigtable, HBase.

**Graph** — nodes for entities, edges for relationships, properties on both. When the relationships *are* the query: social graphs, fraud rings, recommendations, knowledge graphs. Neo4j, JanusGraph, AWS Neptune.

Picking among these — and against SQL — is the whole subject of [choosing the right database](/citadel/random-thoughts/right-db) and the broader survey of [data systems](/citadel/interview/data-systems).

## MongoDB

MongoDB is an open-source document database that stores data as **BSON** — binary JSON, extended with types JSON lacks like dates and raw binary, and laid out for fast traversal. A *collection* is its equivalent of a table, a *document* its equivalent of a row.

- **Dynamic schema.** Two documents in the same collection can have entirely different fields. The data model evolves as the application does, with no schema-migration step.
- **Sharding.** MongoDB scales horizontally by [partitioning a collection](/citadel/interview/data-sharding) across shards on a shard key, so both data volume and throughput grow with the cluster.
- **Replica sets.** A replica set is a group of `mongod` processes holding the same data: one *primary* takes all writes, *secondaries* copy it and can serve reads, and if the primary fails the set elects a new one automatically — the [read-replica pattern](/citadel/interview/read-replica-pattern) with built-in failover.
- **Indexing.** Single-field, compound, geospatial, text, and hashed indexes on any field.
- **Aggregation pipeline.** A sequence of stages — match, group, project, sort — that processes documents inside the database, so analytical work doesn't have to pull everything to the client.
- **GridFS.** A convention for storing files larger than the 16 MB document limit by splitting them into chunks across two collections.

### Embed or reference

The document model gives you a modelling choice relational schemas don't:

- **Embed** related data inside one document — an order with its line items nested in it. One read gets everything; no join. The cost is duplication and a document that can grow unbounded.
- **Reference** related data by storing its `_id` in another document, like a foreign key. Use it for many-to-many relationships, or when embedding would make documents huge or force you to update the same fact in many places.

The right call depends on how the application reads the data. Read the order and its items together every time? Embed. Do items get queried and updated on their own? Reference.

## SQL or NoSQL

Reach for a **relational database** when the data is well-structured, the relationships matter, you need real joins, and ACID guarantees are non-negotiable. Reach for **NoSQL** when the data is loosely structured or fast-changing, you need horizontal scale and high availability, and your reads and writes follow a known, narrow set of access patterns you can design the storage around.

Most large systems end up doing both — **polyglot persistence**, a relational store for the money and the core entities, a document or key-value store for the parts that fit those models better.

## The takeaway

NoSQL is a family, not a database. The document, key-value, wide-column, and graph models each drop joins and rigid schema for a different payoff, and most of them hand you eventual consistency in the bargain. MongoDB's version of that deal is flexible documents, automatic sharding, and self-healing replica sets — worth taking when your access patterns are known and narrow, and worth questioning when you find yourself wishing for a join.
