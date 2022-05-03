---
title: Distributed Databases - Fragmentation, Transparency, and Queries
description: One logical database spread across many sites - the transparencies that hide the distribution, how a global relation is fragmented and allocated, how a query is decomposed and optimized to minimize network traffic with semi-joins, and how transactions and locking work across sites.
date: 2022-05-03
draft: false
slug: /dbms/distributed-database
tags:
  - Databases
  - Distributed Databases
  - Replication
---

A **distributed database** is one logical database physically spread across multiple sites on a network, with a **DDBMS** that makes the spreading invisible — users query it as if it were on a single machine. You'd do this for availability (a site failing doesn't stop the system), performance (keep data near the users who read it, run queries in parallel), and scaling (add commodity nodes instead of a bigger mainframe).

The price is complexity everywhere: a query now has to worry about where data lives and how much crosses the network; a transaction has to stay atomic across sites that can fail independently. This post covers how data is carved up and placed, how queries are optimized around network cost, and how transactions hold together.

## What you gain and pay

**Advantages**: reliability and availability (other sites carry on; a replica can take over), performance (locality of reference, parallel execution), elastic scalability, local autonomy over each site's data, and economics (many small machines beating one large one).

**Disadvantages**: design and operational complexity, a larger attack surface, the difficulty of keeping replicated data consistent, higher software and staffing cost, and less mature standards than centralized systems.

## Transparency

The DDBMS's job is to hide the distribution behind a set of **transparencies**:

- **Distribution transparency** — *location* (query without knowing which site holds the data) and *naming* (one global name per object regardless of location).
- **Fragmentation transparency** — a table may be split into pieces; users see the whole table.
- **Replication transparency** — copies exist for availability and speed; users don't know.
- **Performance transparency** — the DDBMS optimizes queries for the distributed layout.
- **Transaction transparency** — ACID for transactions that span sites.
- **Failure transparency** — keep running (perhaps degraded) through site and link failures, and recover.
- **Heterogeneity transparency** — hide differences in hardware, OS, or local DBMS.

Supporting them is a **global directory** (the distributed catalog: schemas, fragment locations, replica placement), which is itself **centralized** (bottleneck, single point of failure), **distributed** (consistency is hard), or **replicated** (fast reads, synchronised updates).

## Fragmentation and allocation

Designing a distributed database is **top-down** (new system: global schema → fragment → allocate) or **bottom-up** (integrate existing heterogeneous databases under a harmonising global schema).

**Fragmentation** splits a global relation into pieces:

- **Horizontal** — subsets of *rows* by a condition (`Customers` split by `Region`).
- **Vertical** — subsets of *columns*, each fragment carrying the primary key (frequently-read columns in one fragment, a large `Photo`/`Bio` in another).
- **Mixed** — horizontal then vertical, or vice versa.

Three rules keep it safe: **completeness** (every value maps to some fragment), **reconstruction** (the original relation is recoverable from the fragments), **disjointness** (rows don't overlap for unreplicated horizontal fragments; attributes don't overlap, PK aside, for vertical fragments).

**Allocation** places the fragments:

| Scheme | Meaning |
| --- | --- |
| Centralized | everything at one site (not really distributed) |
| Partitioned | no replication; each fragment at exactly one site |
| Fully replicated | the whole database at every site — max availability and read speed, very slow writes |
| Partially replicated | some fragments replicated at some sites — the usual choice |

Decisions weigh locality of reference, availability, query performance, storage cost, and communication cost.

## Distributed query processing

A query is now expensive in a new way: **network communication cost** usually dominates CPU and I/O, because shipping data between sites is slow. The processor turns a high-level SQL query into an efficient plan in four layers:

1. **Query decomposition** — parse the SQL into relational algebra over the *global* schema; normalize, semantically and type check, simplify redundant predicates, and restructure (push selections and projections down the tree).
2. **Data localization** — rewrite the global query into **fragment queries** using the fragmentation info in the directory.
3. **Global query optimization** — choose the execution strategy across sites: operation ordering, which site does each join, and data-shipping tactics. This is the hard part.
4. **Local query optimization** — each fragment query is then optimized by its local DBMS with ordinary [single-node techniques](/citadel/dbms/advanced).

The key network-saving tactic is the **semi-join**. To join relation `A` (at site 1) with `B` (at site 2), don't ship all of `A`: project `A` onto the join attributes, ship that small set to site 2, use it to filter `B` down to the matching rows, and ship only those back. Classic optimization algorithms include **System R\*** (dynamic programming over total cost = CPU + I/O + communication) and hill-climbing heuristics for large queries.

## Distributed object management

When the data is objects (encapsulated data plus methods, each with an identity independent of its location), a few extra problems appear: naming and locating objects that can move, persistence, and object-level concurrency and recovery. Two architectures: an **object server** stores objects and runs their methods, with clients invoking remotely; a **page server** ships pages and lets clients assemble objects locally. Middleware — **CORBA**, **DCOM**, **Java RMI** — provides the plumbing. Performance techniques mirror the relational ones: **object caching** (with a cache-consistency problem), **object clustering** (co-locate objects used together), and **object migration** (move an object toward its users or for load balancing).

## Distributed transactions

A **global transaction** touches data at several sites and must be ACID across all of them. Concurrency control extends the single-node methods:

- **Locking** — *centralized 2PL* (one lock manager for the whole system), *primary-copy 2PL* (lock the designated primary copy of a replicated item), or *distributed 2PL* (each site manages locks for its own data). A global transaction acquires locks at every site it touches.
- **Timestamp ordering** — one global timestamp per transaction, orders conflicting operations everywhere.
- **Optimistic** — proceed without locks, validate at commit.

**Distributed deadlock** can form a cycle spanning sites (`T1` at site A waits for `T2`, `T2` at site B waits for `T1`), detected with a global waits-for graph, a distributed cycle search, or timeouts, and broken by aborting a transaction.

**Distributed commit** ensures global atomicity through **two-phase commit** (a voting phase then a decision phase — blocking if the coordinator fails after voting) or **three-phase commit** (an extra pre-commit phase, non-blocking under limited failure assumptions). The full protocols, the blocking problem, and the Saga alternative are in [transaction-processing systems](/citadel/dbms/transaction-processing). Each site keeps its own recovery log; the commit protocol guarantees all sites eventually reach the same decision (all commit or all abort).

## The one idea to keep

Distributed databases are a bet that the benefits of spreading data out — availability, locality, scale — outweigh the coordination costs, and every technique here is about keeping those costs down. Fragmentation and allocation put data where it's used. Semi-joins keep queries from shipping whole tables. Two-phase commit keeps transactions atomic across sites, and Sagas step in when that guarantee is too expensive to keep. The transparencies are the promise that, done well, none of this leaks to the person writing the query.
