---
title: Database Management Systems - Architecture and Core Components
description: What a DBMS is and how it's built - why file-processing systems failed, the data models from hierarchical to relational, the three-schema architecture that gives data independence, and the storage manager and query processor that make a database engine work.
date: 2022-04-17
draft: false
slug: /dbms/dbms
tags:
  - Databases
  - Relational Model
  - Architecture
---

Before databases, an organisation's data lived in separate files owned by separate departments — Personnel had employee files, Payroll had its own, Accounting had a third. The same address sat in three places, changed in one, and quietly disagreed forever after. A **database** replaces that with one integrated, shared store, and a **database management system (DBMS)** is the software that stands between users and the raw bytes, enforcing structure, constraints, and access control.

This post is the shape of a DBMS: the data models it can be built on, the layered schema architecture that lets you retune storage without rewriting applications, and the two engines inside — the storage manager and the query processor. The rest of the series then goes deep: [the relational and E/R models plus indexing](/citadel/dbms/relational-data-model), [normalization](/citadel/dbms/rdb-design), [query languages](/citadel/dbms/relational-query-lang), [transactions](/citadel/dbms/transaction-proc) and [transaction-processing systems](/citadel/dbms/transaction-processing), [distributed databases](/citadel/dbms/distributed-database), [advanced topics](/citadel/dbms/advanced), and [worked calculations](/citadel/dbms/calcs).

## Why not just files

File-processing systems failed in predictable ways:

- **Redundancy** — the same fact stored in multiple files, wasting space.
- **Inconsistency** — update one copy, miss another, and the data now contradicts itself.
- **Poor integration** — answering "everything about this customer" meant stitching together incompatible file formats.
- **Hard access** — every new question needed a new program written against a specific file layout.

A database fixes this by holding **interrelated data** in one place, storing both the **end-user data** (the facts) and the **metadata** (data about the data — types, sizes, relationships) that describes and constrains it.

## Data models

A **data model** is the set of conceptual tools for describing data, the relationships between data, its meaning, and the consistency constraints on it. Its building blocks: an **entity** (a real-world thing you store data about), an **entity set** (all entities of one type), an **attribute** (a property of an entity), a **relationship** (an association between entities, with cardinality **1:1**, **1:M**, or **M:N**), and a **constraint** (a rule the data must satisfy).

The models, roughly in order of appearance:

| Model | Structure | Notes |
| --- | --- | --- |
| Hierarchical (1960s) | tree of segments | strictly 1:M parent-child; awkward for M:N |
| Network | graph; a record can have many parents | handles M:N; navigational pointers, rigid |
| Relational (Codd, 1970) | tables (relations) of rows and columns, linked by key values | conceptually simple, powerful query language; the standard |
| Entity-Relationship (Chen, 1976) | diagrams: entities, attributes, relationships | a *conceptual design* tool, later mapped to relational |
| Object-oriented | data + methods encapsulated in objects, with inheritance | good for complex objects; heavier |
| Object-relational | relational tables plus object features | hybrid |
| Semi-structured | flexible per-record schema | XML, JSON |

The [relational model](/citadel/dbms/relational-data-model) and [normalization](/citadel/dbms/rdb-design) get their own posts. The [interview view](/citadel/interview/db-models) compares the modern families (relational, document, key-value, graph, columnar).

## The three-schema architecture

The ANSI/SPARC architecture defines three levels of description so that changes at one level don't ripple to the others — **data independence**:

- **External level** — one or more *user views*, each showing only the part of the database a user group needs.
- **Conceptual level** — a single logical description of the *whole* database: entities, attributes, relationships, constraints, with no storage detail.
- **Internal level** — a single description of the *physical* storage: record layouts, file organisation, indexes, compression.

Two mappings connect them, and each buys an independence:

- **External/conceptual mapping** → **logical data independence**: add a column to a conceptual table without breaking applications that don't use it.
- **Conceptual/internal mapping** → **physical data independence**: change file organisation or add an index without touching the conceptual schema or any application. This is what makes performance tuning possible.

## Who uses a database

- **Naive users** — interact only through prewritten application screens (a bank teller, someone booking a flight).
- **Application programmers** — write those applications, embedding SQL or calling APIs.
- **Sophisticated users** — write ad-hoc queries directly (data analysts).
- **Specialised users** — build applications outside the standard data-processing mould (CAD, knowledge bases).

## What makes a DBMS hard to build

Managing **size and complexity** (the software itself is large); delivering **performance** on complex queries and thousands of concurrent transactions; guaranteeing **atomicity and durability** through crashes; enforcing **consistency** constraints across huge datasets; providing fine-grained **security**; and **recovering** correctly from transaction, system, and disk failures.

## Core components

A DBMS is two engines over a set of on-disk structures.

**Storage manager** — the physical layer:

- **Authorization and integrity manager** — checks permissions and constraint satisfaction on every access.
- **Transaction manager** — enforces the [ACID properties](/citadel/dbms/transaction-proc) and manages concurrent execution.
- **File manager** — talks to the OS file system, allocates disk space, manages page and file structures.
- **Buffer manager** — moves pages between disk and the in-memory buffer cache and decides what to keep and what to flush; the single biggest lever on performance (the same [caching](/citadel/computer-architecture/hierarchical-memory) idea the hardware uses).

Its on-disk structures: **data files** (the actual content), the **data dictionary** / system catalog (all the metadata — schemas, constraints, users, index details, statistics), and **indices** (usually [B+-trees](/citadel/data-structures/b-tree)) for fast access paths.

**Query processor** — the access layer:

- **DDL interpreter** — parses `CREATE TABLE` and friends, records the resulting schema in the data dictionary.
- **DML compiler** — parses a `SELECT`/`INSERT`/`UPDATE`/`DELETE`, validates it against the catalog, runs **query optimization** to pick an efficient plan, and emits low-level operations.
- **Query evaluation engine** — executes those operations, calling the storage manager to fetch pages.

## The DBA

The **database administrator** has central control: defining the schema and storage structures, modifying them as needs change, granting user authority, specifying integrity constraints, running backups and recovery, and monitoring and tuning performance.

## How applications connect

- **2-tier** — the client machine holds the UI and application logic and issues SQL (via ODBC/JDBC) straight to the DBMS server. Simple; heavy on the client, awkward to deploy at scale.
- **3-tier** — a thin client, an **application server** holding the business logic, and a **database server** reached only by the app server. More scalable, and more secure — clients never touch the database directly.

## The one idea to keep

A DBMS earns its keep by decoupling. The three-schema architecture decouples applications from storage, so the physical design can change underneath a running system. The storage manager and query processor decouple *what* data you want from *how* it's fetched, so the optimizer can pick a good plan without the application knowing or caring. Every later topic — indexing, normalization, transactions, distribution — is a way of managing one of those layers without disturbing the others.
