---
title: Advanced Databases - Query Optimization, Tuning, and Warehousing
description: What a database engine does beyond storing rows - the join algorithms and cost-based optimizer that pick an execution plan, the physical-design and tuning levers, the shared-nothing architecture behind parallel databases, and OLAP, star schemas, and data mining for analytics.
date: 2022-06-11
draft: false
slug: /dbms/advanced
tags:
  - Databases
  - Query Languages
  - Data Warehousing
---

A `SELECT` statement says *what* rows you want, not *how* to get them. Between the SQL and the disk sits the part of a database that earns its reputation: an optimizer that considers dozens of ways to run the query and picks one, a physical layer of indexes and file layouts you tune to the workload, and — when one machine isn't enough — a parallel architecture that splits the work.

This post covers those layers, plus the analytics side: how databases are reshaped into data warehouses for OLAP, and the mining techniques that pull patterns out of them. It assumes the [query languages](/citadel/dbms/relational-query-lang) and [indexing basics](/citadel/dbms/relational-data-model).

## Query evaluation

The optimizer has a menu of algorithms for each relational operator. For a join:

- **Nested loop join** — for every row of the outer relation, scan the inner relation for matches. Simple; quadratic, so only good when one side is tiny or indexed.
- **Hash join** — build a hash table on the join attribute of the smaller relation, then probe it with rows from the larger. The default for equi-joins on unsorted data.
- **Sort-merge join** — sort both relations on the join attribute, then merge in one pass. Wins when the inputs are already sorted (or need sorting anyway).

The choice turns on relation sizes, available indexes, and data distribution.

## Query optimization

A single SQL query maps to many logically equivalent relational-algebra expressions, and they can differ in cost by orders of magnitude. The optimizer:

1. **Parses** the SQL into an algebra expression tree.
2. **Rewrites** it with equivalence rules — push selections and projections down so less data flows up, reorder joins, turn subqueries into joins (**decorrelation** converts a correlated subquery, which re-runs per outer row, into a single join).
3. **Estimates cost** for candidate plans using **database statistics** — row counts, distinct-value counts, histograms of value distribution, index metadata — with cost measured in estimated I/O and CPU.
4. **Enumerates plans** over a search space it can't fully explore.

The pioneering **System R optimizer** is cost-based, uses dynamic programming to find a good **join order** for multi-table queries, considers each join method and access path (index versus full scan), and builds plans bottom-up, pruning sub-plans it can prove suboptimal. It's the same "many equivalent forms, pick the cheapest" idea a [compiler's optimizer](/citadel/compiler-design/code-optimisation) uses.

## Physical design and tuning

Physical database design decides how the logical schema sits on disk: [file organisation](/citadel/dbms/relational-data-model) (heap, sorted, hashed) and index structures.

**Index types** beyond B+-tree and hash:

- **Bitmap index** — for low-cardinality columns (a few distinct values). One bitmap per value, with a bit per row indicating membership. Bitmaps for several such columns can be `AND`ed cheaply, which is powerful for analytical filters.
- **Covering index** — an index that contains every column a query needs, so the query is answered from the index alone without touching the table.
- **Clustered index** — the table rows are physically stored in the index key's order. One per table; retrieval by that key is very fast.

Indexes speed up `SELECT` and `WHERE` and slow down `INSERT`/`UPDATE`/`DELETE`, because the index must be maintained too. Choosing them means analysing the query workload, indexing the columns that appear in `WHERE`, `JOIN`, and `ORDER BY`, and stopping there.

**Tuning** is iterative bottleneck removal across the application, DBMS configuration, OS, hardware, and schema. At the schema level, the main lever is **normalization versus denormalization**: normalization removes redundancy and protects integrity but adds joins; **denormalization** reintroduces redundancy selectively to make specific read patterns fast, at the cost of harder updates. At the query level: rewrite inefficient SQL, check execution plans to confirm the right indexes are used, use query hints sparingly, and precompute expensive views as **materialized views**.

## Parallel databases

Three architectures for using many processors and disks:

| Architecture | Memory | Disk | Note |
| --- | --- | --- | --- |
| Shared memory | shared | shared | simple to program; the shared resources bottleneck |
| Shared disk | private | shared | needs a distributed lock manager |
| Shared nothing | private | private | processors pass messages; most scalable — used by MPP databases |

Parallelism within a query comes as **intra-operator** (one operator — a sort, a join — split across processors) and **inter-operator** (different operators of the plan running concurrently, pipelined). Effective parallel processing depends on **data partitioning** — hash, range, or round-robin — so the work divides evenly. The optimizer must weigh parallel strategies, partitioning, and inter-node communication cost. The [parallel-computing](/citadel/parallel-computing/parallel-computing) category covers the shared-nothing model in general.

## Data warehousing and OLAP

Transactional databases are tuned for many small writes; analysis needs something else. A **decision support system** rests on a **data warehouse** — a collection of data that is **subject-oriented** (organised around customers, products, sales), **integrated** (reconciled from many sources), **time-variant** (historical, not just current), and **non-volatile** (loaded periodically, not updated by users).

**OLAP** (online analytical processing) gives fast interactive analysis over a **multidimensional** model — a cube with dimensions like time, product, region. The operations:

- **Slice** — fix one dimension to a single value.
- **Dice** — fix several dimensions to value ranges, getting a sub-cube.
- **Drill-down** — move to finer detail (yearly → quarterly sales).
- **Roll-up** — aggregate up a hierarchy (city → country sales).
- **Pivot** — rotate the axes for a different view.

SQL supports these with `GROUP BY CUBE` and `GROUP BY ROLLUP`. The schema is a **star schema** — a central fact table surrounded by dimension tables — or a **snowflake schema** with the dimension tables further normalized. Frequently-run aggregations are stored as **materialized views** and refreshed as the underlying data changes.

## Data mining

Finding patterns in the warehouse:

- **Association rules** — "if A then B" relationships (market-basket analysis: buyers of bread also buy milk), scored by **support** (how often the itemset appears) and **confidence** (how often B follows A).
- **Decision trees** — classification by recursively partitioning on attribute values; leaves are class labels, and the tree is easy to read. Covered in [machine learning](/citadel/artificial-intelligence/ml).
- **Clustering** — unsupervised grouping of similar records (k-means, hierarchical), covered in [unsupervised learning](/citadel/artificial-intelligence/unspervised-learning).

## The one idea to keep

The optimizer is the reason SQL is declarative: because many plans produce the same rows, you can state the result and let a cost model choose among them. Physical tuning is choosing which of those plans are *cheap* to run — the indexes and layouts that turn a scan into a seek. And the warehouse is the acknowledgement that the storage design serving transactions is the wrong one for analysis, so you build a second copy shaped for the questions analysts ask.
