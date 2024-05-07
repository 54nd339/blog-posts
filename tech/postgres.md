---
title: RDBMS and PostgreSQL - The Relational Core and an Extensible Engine
description: "The relational model in one pass - tables, keys, ACID, normalization, joins, indexes - and then why PostgreSQL is the one people reach for: MVCC concurrency, five index types, and an extension system that pulls geospatial, time-series, vector search, and sharding into a single database."
date: 2024-05-07
draft: false
slug: /tech/postgres
tags:
  - Tools
  - Databases
  - PostgreSQL
---

The relational database has been the default place to put data for forty years, and the reason is boring in the best way: it makes strong promises and keeps them. Data goes in structured tables, relationships between tables are enforced by the engine, and transactions either complete fully or leave no trace.

This post does the relational model in one pass, then looks at PostgreSQL specifically — because among open-source relational databases it's the one that keeps winning the "most loved" surveys, and the reason is an extension system that lets one database do jobs you'd normally spin up three more for.

## The relational model

A **relational database** organises data into tables (formally *relations*) of rows and columns, with explicit links between tables.

- **Primary key** — a column, or set of columns, that uniquely identifies each row.
- **Foreign key** — a column in one table that points at another table's primary key. This is the mechanism that enforces relationships: the database rejects an order row whose `customer_id` doesn't exist.
- **SQL** — the language, in four parts: DDL to define structure (`CREATE TABLE`), DQL to read (`SELECT`), DML to change rows (`INSERT`, `UPDATE`, `DELETE`), and DCL for permissions (`GRANT`). The [five sublanguages and how a query runs](/citadel/interview/sql) is its own topic.
- **ACID** — **atomicity** (a transaction is all-or-nothing; a failure rolls the whole thing back), **consistency** (each transaction moves the database from one valid state to another, preserving every constraint), **isolation** (concurrent transactions produce the same result as if they'd run one after another — the [isolation levels](/citadel/interview/db-isolation) tune how strictly), and **durability** (once committed, a transaction survives a crash).
- **Normalization** — arranging columns and tables so each fact is stored once, which removes update anomalies at the cost of needing joins to reassemble data.
- **Joins** — combine rows from two tables on a shared column: `INNER` (only matching rows), `LEFT`/`RIGHT OUTER` (all rows from one side, matched or not), `FULL OUTER` (all rows from both).
- **Indexes** — auxiliary structures the engine uses to find rows without scanning the whole table, like the index at the back of a book.
- **Views and materialized views** — a *view* is a saved query that acts like a virtual table; a *materialized view* stores the query's results on disk and refreshes on demand, trading freshness for speed.

## PostgreSQL

Postgres is an *object-relational* database — the relational model plus features like custom types and inheritance — with a long record of tracking the SQL standard closely. What sets it apart in practice:

**Concurrency via MVCC.** Multi-Version Concurrency Control means a writer creates a new version of a row rather than locking it, so readers never block writers and writers never block readers. This is why Postgres handles mixed read/write load without the lock contention older designs suffer.

**Indexing that fits the data.** Five index types, each for a shape of query: **B-tree** for ordered data and ranges, **Hash** for equality, **GiST** for geometric and full-text, **GIN** for multi-valued columns like arrays and JSONB, **BRIN** for huge tables where rows are physically ordered by the indexed column (a time column, typically).

**Extensions.** This is the headline feature. An extension adds types, functions, index methods, or storage behaviour to a running database:

- **PostGIS** turns Postgres into a full geospatial database — geometry types, spatial indexes, distance and containment queries.
- **TimescaleDB** adds [time-series](/citadel/tech/timeseries-db) optimisations — automatic partitioning by time, compression, continuous aggregates.
- **pgvector** adds a vector type and similarity search, for embeddings and semantic search.
- **Foreign Data Wrappers** let you query a MySQL table, a MongoDB collection, or a CSV file as if it were a local table.
- **Citus** shards tables across nodes and parallelises queries, giving Postgres horizontal [scale-out](/citadel/interview/data-sharding).
- **pg_graphql**, **Apache AGE**, and full-text extensions like `pgroonga` cover [GraphQL](/citadel/interview/graphql), graph traversal, and search.

**Native semi-structured support.** `JSONB` stores and indexes JSON documents, and `hstore` stores key-value pairs — so a lot of what people reach for [NoSQL](/citadel/tech/mongo) to do can happen inside Postgres, keeping one system and one set of transactions.

The rest is table stakes done well: window functions, stored procedures, triggers, no licensing cost, and a scaling record that includes companies like Figma running it under very large load.

## Where relational still wins

Even with NoSQL everywhere, a relational database — Postgres especially — is the right default when the data is structured, the relationships between entities are load-bearing, you need real joins and complex queries, and you can't compromise on ACID. The [right database](/citadel/random-thoughts/right-db) for a job is often just Postgres with the right extension turned on.

## The takeaway

The relational model's value is enforced structure and transactions that keep their word. PostgreSQL takes that core and makes it extensible, so geospatial, time-series, vector search, and sharding become configuration rather than new infrastructure. Before adding a specialised datastore, it's worth checking whether an extension already covers it.
