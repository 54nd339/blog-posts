---
title: Big Data and Starburst - Querying Data Where It Lives
description: "The \"V\" characteristics that make big data hard, and the ecosystem that handles it - distributed storage, batch and stream processing, query engines. Then Starburst and Trino, which take the federated approach: one SQL query joining a data lake, a warehouse, and a Postgres database without moving any of it."
date: 2024-05-08
draft: false
slug: /tech/starbust
tags:
  - Tools
  - Big Data
  - Data Engineering
---

Most approaches to big data start by moving it: extract from the source systems, transform it, load it into one central place you can query. That works, but the copying is expensive, the copies go stale, and by the time the data is centralised the question that prompted the move has often changed.

**Starburst**, built on the open-source **Trino** engine, takes the other route: leave the data where it is — in the lake, in the warehouse, in an operational database — and send the query to it. This post covers what makes big data hard in the first place, the ecosystem that addresses it, and how federated query fits in.

## What makes it "big"

Big data is usually described by a set of characteristics Gartner started naming in 2001. The first three:

- **Volume** — the raw size, in terabytes to exabytes.
- **Velocity** — how fast it arrives and how fast you need to act on it, often in real time.
- **Variety** — structured rows, semi-structured JSON and sensor data, unstructured text and images, from many sources at once.

Later additions:

- **Veracity** — how much you can trust it; big data is often noisy and incomplete.
- **Value** — the point of the exercise: extracting insight worth more than the cost of storing and processing.
- **Variability** — the meaning of a field, and the way it's collected, drift over time.

The payoff for handling all this is the usual list — better insight into trends and behaviour, data-driven decisions, operational efficiency, new products — but the characteristics are what break traditional tools.

## The ecosystem

**Storage.** Distributed file systems like [HDFS](/citadel/tech/apache), and cloud object storage (S3, Azure Blob, GCS) used as [data lakes](/citadel/tech/databricks).

**Processing.** *Batch* engines — Hadoop MapReduce, Spark — for large volumes processed on a schedule. *Stream* engines — Apache Flink for stateful computation, [Kafka](/citadel/tech/kafka) as the transport, Apache Storm — for acting on data as it arrives.

**Query and warehousing.** Hive for SQL over files; data warehouses like Snowflake and Redshift for structured, processed data; and SQL-on-lake query engines — Trino and Starburst — that query the lake directly.

Most of this traces back to three Google papers: the **Google File System** (distributed storage), **MapReduce** (parallel processing), and **BigTable** (distributed structured storage), which seeded Hadoop and Spark and shaped how both transactional and analytical systems evolved. See [data pipelining](/citadel/interview/data-pipelining) and [data systems](/citadel/interview/data-systems) for how these pieces get assembled.

## Starburst and Trino

Trino is a distributed SQL query engine. Starburst packages it with enterprise features, connectors, and support. The core idea is **data federation** — query data where it lives, without first moving it into one repository.

**Decoupled compute and storage.** Trino stores nothing itself. It reaches data through **connectors**, and there are connectors for most things:

- Data lakes via open table formats — Hive, Apache Iceberg, Hudi, Delta Lake — on S3 or HDFS.
- Warehouses — Snowflake, Redshift, BigQuery.
- Relational databases — [PostgreSQL](/citadel/tech/postgres), MySQL.
- NoSQL — [MongoDB](/citadel/tech/mongo), Cassandra.
- Streaming — Kafka.

**Massively parallel processing.** A Trino cluster has one **coordinator** — it parses the SQL, plans and optimises the query, and hands work to the workers — and many **worker** nodes that execute tasks in parallel over the data. Scaling out is adding workers.

**Federated queries.** Because every source is a connector, one SQL statement can join a table in the lake to a table in Postgres to a topic in Kafka, and Trino executes it as a single distributed query. No ETL job, no copy.

**Data lake analytics.** Fast interactive SQL straight against lake files in open formats — Starburst's "Icehouse" architecture pairs Trino with Iceberg specifically for this.

**Governance and performance.** Role-based access control, encryption, query auditing, and acceleration features (Starburst calls its caching layer "Warp Speed").

Starburst doesn't replace the lake or the warehouse — it's a query layer over them. It lets analysts use SQL instead of writing Spark jobs, supports a data-mesh model where teams own their data but everyone queries through one interface, and cuts the number of pipelines whose only job is to copy data somewhere queryable.

## The takeaway

The centralise-then-query model and the federated model solve the same problem from opposite ends. Centralising gives you one optimised store and predictable performance, at the cost of pipelines and staleness. Federation gives you fresh data and no copying, at the cost of query performance bounded by the slowest source. Starburst makes the federated side practical enough to be the default for a lot of ad-hoc and BI work.
