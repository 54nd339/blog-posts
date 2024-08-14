---
title: SQL - The Five Sublanguages and How a Query Runs
description: SQL splits into DQL, DML, DDL, DCL, and TCL — asking, changing, shaping, permitting, and transacting. The core clauses and JOIN types, plus the parse-optimize-execute path a query takes inside the engine.
date: 2024-08-14
draft: false
slug: /interview/sql
tags:
  - System Design
  - Interview Prep
  - SQL
---

**SQL** is the domain-specific language for relational databases, standardised since 1986 and still the way most of the world's structured data is queried. It does five jobs — define the structure, query it, change the data, control access, and manage transactions — and the language splits into five sublanguages along those lines. This covers each, the concepts that matter, and what happens when you press run.

## The five sublanguages

**DQL — asking questions.** The `SELECT` statement and its clauses, evaluated roughly in this order:

- `FROM` / `JOIN` — the tables and how they combine.
- `WHERE` — filter rows (`AND`, `OR`, `NOT`, comparisons).
- `GROUP BY` — collapse rows sharing a value into summary rows.
- `HAVING` — filter those groups (like `WHERE`, but post-grouping).
- `SELECT` — the columns to return.
- `ORDER BY` — sort (`ASC` / `DESC`).
- `LIMIT` / `TOP` — cap the row count.

**DML — changing data.** `INSERT INTO`, `UPDATE`, `DELETE FROM`.

**DDL — shaping the schema.** `CREATE` (`DATABASE`, `TABLE`, `INDEX`, `VIEW`), `ALTER` (add/drop a column), `DROP` (remove an object).

**DCL — permissions.** `GRANT` and `REVOKE` privileges (`SELECT`, `INSERT`, …) on objects to users or roles.

**TCL — transactions.** `BEGIN`/`START TRANSACTION`, `COMMIT` (make changes permanent), `ROLLBACK` (undo to the start), `SAVEPOINT` (a rollback target inside a transaction). This is how the [ACID](/citadel/interview/db-isolation) properties are made available to the application.

## Concepts to have

**Relational structure.** Tables (relations) of rows (tuples) and columns (attributes). **Keys**: a **primary key** uniquely identifies each row; a **foreign key** references another table's primary key to link them; a **surrogate key** is a generated stand-in (auto-increment integer, UUID) when no natural key fits. Relationships are one-to-one, one-to-many, or many-to-many.

**JOINs** — combine rows from two tables on a related column:

| JOIN | Returns |
| --- | --- |
| `INNER` | rows with a match in both tables |
| `LEFT [OUTER]` | all left rows; right columns `NULL` where no match |
| `RIGHT [OUTER]` | all right rows; left columns `NULL` where no match |
| `FULL OUTER` | all rows from either side; `NULL` on the unmatched side |
| `CROSS` | every combination of rows (Cartesian product) |

**The rest:**

- **Subqueries** — a query nested in a `WHERE`, `SELECT`, or `FROM`.
- **Aggregate functions** — `COUNT`, `SUM`, `AVG`, `MAX`, `MIN`, usually with `GROUP BY`.
- **Indexes** — speed up reads by avoiding full scans; slow down `INSERT`/`UPDATE`/`DELETE` and cost storage.
- **Views** — a virtual table defined by a stored query, for abstraction or a security boundary. A **materialized view** stores the results physically and must be refreshed.
- **Stored procedures and functions** — named, pre-compiled blocks of SQL for encapsulating logic.
- **Data types** — numeric (`INT`, `DECIMAL`), string (`VARCHAR`, `TEXT`), temporal (`DATE`, `TIMESTAMP`), boolean, and often `JSON`.

## How a query runs

Submitting SQL is submitting a *request*, not a plan. The engine:

1. **Parses and validates** — checks syntax, then that the tables and columns exist and the user may access them.
2. **Transforms** — rewrites the query into an internal form (relational algebra, an execution tree).
3. **Optimizes** — the query optimizer enumerates ways to get the data — which index to use, which order to join tables — and picks one using table statistics and a cost model. This step is why two logically identical queries can perform very differently, and why stale statistics cause bad plans.
4. **Executes** — the chosen plan runs: read from disk or cache, join, filter, sort, aggregate.
5. **Returns** — for `SELECT`, formats the rows and sends them back.

## SQL endures because the guarantees do

NoSQL took the workloads where a rigid schema and ACID were a cost rather than a benefit, but for anything that needs enforced structure, referential integrity, and transactions that either fully happen or fully do not, SQL over a [relational database](/citadel/interview/db-models) is still the default — and the declarative model means you say *what* you want and the optimizer, not you, works out the *how*. The [isolation levels](/citadel/interview/db-isolation) post covers what "a transaction" actually guarantees under concurrency.
