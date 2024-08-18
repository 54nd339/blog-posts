---
title: Data Pipelines - Collect, Ingest, Store, Compute, Consume
description: Raw data from many sources becomes something a dashboard or a model can use by passing through five stages. What each stage does, the technologies that fill it, and why batch and stream processing gave rise to the Lambda and Kappa architectures.
date: 2024-08-18
draft: false
slug: /interview/data-pipelining
tags:
  - System Design
  - Interview Prep
  - Data Engineering
---

Raw operational data — rows in a transactional database, clickstream events, sensor readings — is not in a shape a dashboard, a report, or a model can use. A **data pipeline** is the sequence of stages that gets it there: gather it from its sources, move it into a processing system, land it somewhere durable, transform it, and hand it to whatever consumes it.

The stages have standard names — **collect, ingest, store, compute, consume** — and standard technology at each. This walks through them and the design choices that separate one pipeline from another.

## Collect

Acquire data from where it already lives: relational and NoSQL databases, application event logs, IoT devices, website clickstreams, third-party SaaS APIs — usually spread across many systems and locations.

For existing databases, the key technique is **change data capture (CDC)**: rather than repeatedly re-scanning a table, a CDC tool such as Debezium tails the database's transaction log — MySQL's binlog, Postgres's logical decoding stream — and emits every insert, update, and delete as an event. Downstream systems stay in near-real-time sync without polling. CDC is also covered from the database side in [data systems](/citadel/interview/data-systems).

## Ingest

Move the collected data into the pipeline's processing environment and buffer it for what comes next.

- **Batch load** — files (CSV, Parquet, Avro) or periodic database dumps, loaded on a schedule.
- **Event queue** — a [message broker](/citadel/interview/message-queue), almost always Apache Kafka, for streaming data. Kafka decouples producers from consumers, sustains high throughput, and retains messages on disk until they age out — which is what lets a consumer fall behind and catch up, or replay from the start.

## Store

Land the data, raw or refined, in storage matched to how it will be queried.

- **Data lake** — a central repository holding data in its native form, structured to unstructured, on cheap object storage (S3, GCS, Azure Blob). Schema-on-read: you impose structure when you query, not when you write. Good for keeping everything cheaply for later.
- **Data warehouse** — cleansed, structured, schema-on-write data tuned for BI and OLAP queries.
- **Data lakehouse** — table formats like Delta Lake, Apache Iceberg, and Apache Hudi that add ACID transactions, schema enforcement, and warehouse-grade performance on top of a data lake, aiming to remove the need for a separate warehouse.

The lineage runs back to the Google File System, which inspired HDFS and the whole idea of scaling storage across commodity disks.

## Compute

Turn raw data into something useful — aggregate, cleanse, enrich, validate, normalise, reformat, partition. Two processing models:

- **Batch** (MapReduce, Spark; Hive for a SQL interface over Hadoop) — processes a fixed, bounded dataset on a schedule. Fault tolerance is easy: rerun the failed job on the same input. Suited to nightly summaries and bulk recomputation.
- **Stream** (Flink, Spark Streaming, Storm, Kafka Streams) — processes an unbounded stream continuously, feeding live dashboards, alerting, and continuously updated indexes. Fault tolerance is harder and handled by micro-batching (Spark: process the stream in small time slices) or distributed checkpointing (Flink: periodically snapshot computation state and recover from it). Google's Dataflow model, realised in Flink, unified the two under one abstraction.

Combining them historically produced two architectures:

- **Lambda** — a batch layer recomputing everything for accuracy, a speed layer handling recent data for low latency, and a serving layer merging the two. Accurate but doubles the code.
- **Kappa** — one stream engine for both live and historical data; "reprocessing" is just replaying the durable event log (Kafka) through the same code. Simpler, and the reason a retained log matters.

## Consume

Deliver the refined data to what acts on it: model training and prediction for data science and ML; BI tools (Tableau, Power BI) and self-service analytics for analysts; metric dashboards (Grafana); automated decision engines for fraud scoring or real-time bidding; and back into user-facing applications as recommendations or personalised content.

## The pipeline is a chain of durability decisions

Each stage is really a question of what to keep and for how long: CDC turns a mutable table into an append-only change log, Kafka retains that log so consumers can replay it, the lake keeps raw data indefinitely because storage is cheap and future questions are unknown, and Kappa exists because a durable log makes "reprocess history" the same operation as "process now". Get the retention right at each hop and reprocessing is routine; get it wrong and a bad transform means data you cannot reconstruct.
