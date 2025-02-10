---
title: Database Models - From Flat Files to NoSQL
description: A database model fixes how data relates, what constraints hold, and which queries are cheap. The path from flat, hierarchical, and network models through the relational standard to the four NoSQL families.
date: 2025-02-10
draft: false
slug: /interview/db-models
tags:
  - System Design
  - Interview Prep
  - Databases
---

A database model is the logical structure underneath a database: how data elements relate, what constraints apply, which operations are possible, and — the part you feel daily — which access patterns are fast and which are expensive. Pick the wrong one and every query fights the structure.

This traces the models in the order they appeared: the flat, hierarchical, and network precursors; the relational model that has held the centre since 1970; and the NoSQL families that carved out the cases relational handles poorly.

## The precursors

**Flat model.** One two-dimensional table — rows as records, columns as attributes — like a CSV. Trivial to read and write, but with no relationships it repeats data across rows, which causes update anomalies (change one copy, miss another) and deletion anomalies (drop a row, lose the only record of something). Fine for a config file or a small list; nothing more.

**Hierarchical model.** A tree: every child record has exactly one parent, a parent can have many children. Natural for genuinely hierarchical data — an org chart, a bill of materials — and fast along the predefined paths. But it cannot represent many-to-many relationships without duplicating data, and queries that cut across the hierarchy are awkward. IBM's IMS was the archetype; a filesystem is the shape you still see.

**Network model.** A generalisation of the hierarchy to a graph — a record can have many parents and many children, linked by pointers, with named "sets" defining relationships. More expressive, less redundant, but querying meant navigational programming (chase the pointers by hand), and the schema was hard to change once set. CODASYL databases like IDMS.

## The relational model

E. F. Codd, 1970. Data lives in **tables** (relations) of **rows** (tuples) and **columns** (attributes); relationships are expressed by **keys** — a primary key identifying each row, foreign keys referencing primary keys elsewhere. What it brought:

- **SQL** — one declarative language to define, query, and modify data.
- **Constraints** — primary/foreign keys, uniqueness, type checks, enforced by the database.
- **[ACID transactions](/citadel/interview/db-isolation)** — atomic, consistent, isolated, durable.
- **Normalization** — split data across related tables so each fact is stored once.
- **Schema-on-write** — the structure is fixed before data goes in.

The costs: a rigid schema is friction for fast-changing or highly varied data; scaling writes horizontally needs [sharding](/citadel/interview/data-sharding); and mapping application objects onto tables is the perennial "impedance mismatch". Still the default for OLTP, finance, ERP, CRM — anything wanting strong consistency over structured data. MySQL, PostgreSQL, Oracle, SQL Server.

### Star and snowflake schemas

Analytics uses the relational model with a particular shape. A **star schema** has a central **fact table** of measures (sales amount, quantity) and foreign keys, surrounded by **dimension tables** of descriptive context (time, product, customer) kept denormalized for few-join queries. A **snowflake schema** normalizes those dimensions into sub-tables — less redundancy, more joins, generally slower. Star for query speed, snowflake for dimensional integrity.

## The NoSQL families

As web-scale volumes and schema churn strained the relational fit, four families emerged, generally trading strict ACID for horizontal scale and availability (the BASE posture — basically available, soft state, eventual consistency):

- **Document** (MongoDB) — self-describing JSON/BSON documents. Semi-structured data, evolving schemas.
- **Key-value** (Redis, DynamoDB) — the minimal model, a value per key. Caching, sessions, fast lookups.
- **Wide-column** (Cassandra, HBase) — rows with flexible column families, tuned for sparse data, high write throughput, and spreading across many nodes.
- **Graph** (Neo4j, Neptune) — nodes and edges as first-class citizens, for data whose value is in the relationships: social graphs, recommendations, fraud rings.

**Object storage** (S3) is not a database — no queries, no transactions — but it is the storage layer under many modern systems: opaque objects plus metadata in a flat namespace, reached over HTTP, backing data lakes and unstructured blobs. It is covered in [storage systems](/citadel/interview/storage-systems).

## The model follows the access pattern

There is no ranking of these, only a fit to a workload: how structured the data is, whether its value lives in relationships, whether you need strong consistency or can take eventual, whether queries are key lookups or multi-table joins or aggregations, and how often the schema moves. Large systems increasingly go **polyglot** — a relational store for orders, a key-value store for sessions, a graph for the social layer — choosing per service rather than forcing one model to cover everything. The [SQL post](/citadel/interview/sql) goes deeper on the relational half.
