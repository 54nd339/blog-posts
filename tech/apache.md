---
title: Hadoop, Spark, and Hive - The Original Big Data Stack
description: "Hadoop gave big data three things: HDFS for storage, MapReduce for batch processing, and YARN for scheduling. Spark replaced MapReduce with an in-memory engine, and Hive put SQL on top. How the three fit together and where each one still earns its place."
date: 2024-05-13
draft: false
slug: /tech/apache
tags:
  - Tools
  - Big Data
  - Data Engineering
---

When a dataset stops fitting on one machine, every part of working with it changes — storage, processing, even asking a simple question. The Apache stack that grew out of Google's early-2000s papers on the Google File System and MapReduce was the first widely used answer, and three of its projects still anchor most big data platforms: **Hadoop** for storage and scheduling, **Spark** for processing, and **Hive** for SQL access.

This post covers what each one does, and how a pipeline uses all three together.

## Hadoop: storage and scheduling

Hadoop is a framework for distributing storage and computation across a cluster of commodity machines. Three components:

**HDFS (Hadoop Distributed File System)** stores very large files — terabytes to petabytes — by splitting each into blocks and spreading the blocks across DataNodes. A single **NameNode** holds the filesystem namespace and tracks which blocks live where; the **DataNodes** hold the actual data. Fault tolerance comes from replicating every block (three copies by default) onto different nodes, so losing a machine loses nothing. That 3× storage overhead is exactly the problem [erasure coding](/citadel/interview/erasure-coding) later solved for colder data.

**MapReduce** is the original processing model. A job runs in two phases: the **map** phase splits the input into independent chunks, processes them in parallel, and emits intermediate key-value pairs; the **reduce** phase shuffles and sorts those pairs by key and aggregates them into the final output. It's a powerful model but a blunt one — non-trivial logic becomes a chain of jobs, each writing its output back to HDFS before the next reads it.

**YARN (Yet Another Resource Negotiator)**, added in Hadoop 2, is the cluster's scheduler. It separates resource management from MapReduce, so the cluster can run Spark, Hive, and other frameworks too. A **ResourceManager** allocates cluster-wide capacity; a **NodeManager** on each machine manages that node's share.

Hadoop scales to thousands of nodes on cheap hardware and tolerates their failure. Its niche is large-scale batch processing, archiving, and log crunching — and being the storage layer everything else sits on.

## Spark: the in-memory engine

Spark is a general-purpose cluster compute engine, and it's fast for one main reason: it keeps intermediate results in memory instead of writing them to disk between steps, which is where MapReduce spends much of its time. On iterative work — training a model, running graph algorithms — that's often an order of magnitude difference.

Its core abstraction is the **RDD (Resilient Distributed Dataset)**: an immutable, partitioned collection that Spark can rebuild from its lineage if a partition is lost. On top of RDDs sits the **DataFrame/Dataset API**, a structured interface that lets the **Catalyst optimizer** rewrite queries for efficiency. Around that core:

- **Spark SQL** — SQL and DataFrame queries, able to read Hive tables directly.
- **Spark Streaming** — stream processing in small batches.
- **MLlib** — distributed machine learning: classification, regression, clustering, collaborative filtering.
- **GraphX** — graph-parallel computation.

Spark runs standalone or on YARN, and reads and writes HDFS (and S3, Cassandra, and more), so it slots into a Hadoop cluster without replacing it.

## Hive: SQL over files

Hive, built at Facebook, is a data warehouse layer that lets you query files in HDFS with **HiveQL**, a SQL dialect — so analysts don't have to write MapReduce by hand. Its pieces:

- **Metastore** — the schemas, column types, and file locations for Hive tables, kept in a relational database like MySQL or Postgres.
- **Driver, compiler, optimizer** — parse a HiveQL query and turn it into an execution plan.
- **Execution engine** — runs that plan. Originally always MapReduce; now usually **Apache Tez** or **Spark**, which cut query latency substantially.

Hive is **schema-on-read**: the data sits in HDFS in whatever form it landed, and the schema is applied when you query it, not when you write it. That makes it well suited to warehousing, ad-hoc BI queries, reporting, and ETL, and it extends through user-defined functions.

## How they work together

A typical pipeline uses all three:

1. **Storage** — raw and processed data lands in HDFS (or cloud object storage).
2. **Scheduling** — YARN allocates cluster resources to whatever's running.
3. **Warehousing** — Hive defines tables over the HDFS data and runs batch ETL and BI queries, executed by Tez or Spark.
4. **Advanced processing** — Spark handles the work that needs speed or that MapReduce can't express well: interactive analysis, iterative ML, [stream processing](/citadel/tech/kafka) — reading from HDFS or Hive tables and keeping intermediate state in memory.

The modern lineage of this stack runs through [data lakes and the Lakehouse](/citadel/tech/databricks) and [federated query engines](/citadel/tech/starbust), and the pipeline-building concerns are the subject of [data pipelining](/citadel/interview/data-pipelining) and [data systems](/citadel/interview/data-systems).

## The takeaway

Hadoop's lasting contribution is the split it made: storage (HDFS), processing (a pluggable engine), and scheduling (YARN) as separate concerns. MapReduce, the processing engine it shipped with, is the part that aged out — Spark does the same job without the disk round-trips. Hive is still the reason an analyst can query a petabyte of files in SQL. New platforms rearrange these pieces; they rarely discard them.
