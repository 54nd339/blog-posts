---
title: The Hadoop Ecosystem - HDFS, MapReduce, YARN, and the Tools Around Them
description: "The three core pieces of Hadoop - HDFS for distributed storage, MapReduce for parallel processing, YARN for resource management - and the ecosystem built on top: HBase, Hive, Pig, Sqoop, ZooKeeper, Flume, and Oozie."
date: 2023-04-28
draft: false
slug: /big-data/hadoop
tags:
  - Big Data
  - Hadoop
  - Distributed Systems
---

Apache Hadoop is a framework for storing and processing very large datasets across clusters of ordinary, inexpensive machines. It is not a single program but a small core plus an ecosystem of projects that fill in the gaps. This post covers the core three — a distributed file system, a processing model, and a resource manager — and then the tools that sit on top: a NoSQL store, two query layers, a data-transfer utility, a coordination service, a log collector, and a workflow scheduler.

Where this fits: the [big data stack](/citadel/big-data/tech-foundation) post places Hadoop among the other options; newer engines like [Spark](/citadel/tech/apache) have taken over much of the processing, but HDFS and YARN remain foundational.

## HDFS: distributed storage

The **Hadoop Distributed File System** stores files across many machines and tolerates any of them failing. It uses a master/slave design:

- The **NameNode** (master) holds the filesystem metadata — names, directory tree, permissions, and which blocks make up each file and where those blocks live. It is the single point that knows the layout.
- The **DataNodes** (slaves, one per machine) store the actual data blocks and serve reads and writes.

A large file is split into fixed-size blocks (128 MB or 256 MB by default), and each block is replicated across several DataNodes — three copies is the default. If a node dies, the block is still available from its other replicas, and HDFS re-replicates to restore the count. The design is tuned for **high-throughput sequential access** to large files, not low-latency random reads: it streams gigabytes efficiently and scales to thousands of nodes and petabytes, at the cost of being a poor fit for many small random lookups.

## MapReduce: parallel processing

**MapReduce** is a programming model that hides the hard parts of distributed processing — parallel execution, fault tolerance, data movement — behind two functions you write:

1. **Map** takes an input record (a key-value pair) and emits zero or more intermediate key-value pairs. Map tasks run in parallel, each on one chunk of the input.
2. **Reduce** takes a key and all the intermediate values for that key, and emits the final output. Reduce tasks also run in parallel.

A job runs in stages:

1. **Input splitting** — the input (from HDFS) is divided into chunks, roughly one per block.
2. **Mapping** — each map task applies your map function to its chunk.
3. **Shuffle and sort** — the framework collects every intermediate pair, sorts by key, and groups the values for each key. This is the step that moves data across the network.
4. **Reducing** — each reduce task processes the values for its keys and writes the result back to HDFS.

This is [data parallelism](/citadel/parallel-computing/parallel-computing) in its purest form: the same operation on independent slices.

**Tuning levers:**

- **Combiners** — a local reduce that runs on each mapper's output before the shuffle, cutting the volume that crosses the network.
- **Partitioners** — control which reducer each key goes to; a custom one can balance an uneven load.
- **Split size** — smaller splits mean more parallelism but more overhead.
- **Compression** — compressing map output and final output trades CPU for less I/O and network traffic.
- **Writables** — Hadoop's own serialisable types, faster than generic serialisation.

MapReduce jobs handle log analysis, web indexing, ETL, data mining, and building recommendation models. Spark has displaced it for interactive and iterative work because MapReduce writes intermediate results to disk between every stage, but the model is what made large-scale cluster processing routine.

## YARN: resource management

**YARN** (Yet Another Resource Negotiator), introduced in Hadoop 2, separates cluster resource management from the processing engine, so frameworks other than MapReduce — Spark, Flink, Tez — can share one cluster.

- **ResourceManager** (one per cluster, master) allocates resources globally. Its **Scheduler** hands out resources by policy (capacity or fair scheduling) without tracking application status; its **ApplicationManager** accepts job submissions and launches each application's coordinator.
- **NodeManager** (one per worker) launches and monitors **containers** — bundles of CPU, memory, disk, and network on that node — and reports usage back.
- **ApplicationMaster** (one per application) negotiates containers from the Scheduler and works with NodeManagers to run and monitor the application's tasks. For a MapReduce job, it coordinates the map and reduce tasks.

The payoff is **multi-tenancy** (several frameworks on one cluster), better utilisation (resources allocated dynamically), and better scaling than the Hadoop 1 architecture, where the JobTracker was both scheduler and monitor and became a bottleneck.

## HBase: random access on top of HDFS

**HBase** is a distributed, column-family NoSQL database modelled on Google's Bigtable. It provides the low-latency random reads and writes that HDFS alone does not.

- **Data model** — tables have column families defined up front, but columns can be added to a family per row at any time. Tables can hold billions of rows and millions of columns, and store sparse data efficiently (empty cells cost nothing). Reads and writes are strongly consistent for a single row. Cells are versioned by timestamp.
- **Sharding** — tables split automatically into **regions**, distributed across **RegionServers**.
- **Storage** — HBase writes its data files (HFiles) and write-ahead logs to HDFS, inheriting its durability and scalability.

It fits applications that need real-time random access to very large datasets — user profiles, message histories, live dashboards — and can be a source or sink for MapReduce and Spark jobs.

## Hive: SQL over Hadoop

**Apache Hive** gives a SQL interface to data in HDFS or S3. You write **HiveQL**, close to standard SQL, and Hive compiles it into MapReduce, Tez, or Spark jobs. Table schemas — columns, types, partitions — live in a **metastore**, usually a relational database like MySQL or PostgreSQL.

Features that matter for performance: columnar file formats (ORC, Parquet), **partitioning** (physically splitting a table by a column so queries skip irrelevant data), **bucketing** (hashing rows into a fixed number of files), and user-defined functions for logic SQL cannot express.

## Pig: a data-flow language

**Apache Pig** offers a procedural alternative to raw MapReduce code. You write **Pig Latin**, a data-flow language — a sequence of transformations (`LOAD`, `FILTER`, `GROUP`, `JOIN`, `ORDER BY`, `STORE`) — and the Pig engine translates it into jobs. It suits rapid prototyping of complex analysis over structured or unstructured data, and extends via UDFs. Like Hive, its direct use has declined as Spark took over, but the pattern — a high-level language compiled to cluster jobs — is the same one Hive and Spark SQL use.

## Data movement and coordination

**Sqoop** (SQL-to-Hadoop) transfers bulk data between Hadoop and relational databases. Imports run as MapReduce jobs that pull from the database in parallel and write to HDFS, Hive, or HBase; exports read from HDFS and insert into database tables. It supports connectors for common databases and **incremental imports** that load only new or changed rows.

**ZooKeeper** is a coordination service for distributed systems. It exposes a small hierarchical namespace of data nodes called **znodes**; clients create, read, and delete them, and set **watches** to be notified of changes. Operations are atomic and ordered, kept consistent across a server ensemble by a consensus protocol (Zab). The ecosystem uses it for HBase master election and RegionServer tracking, YARN ResourceManager high availability, Kafka broker metadata, and generic tasks like leader election, distributed locks, and configuration management.

**Flume** collects, aggregates, and moves large volumes of log or event data into HDFS or HBase. Its pieces:

- **Event** — one unit of data, such as a log line.
- **Source** — consumes events from outside (web server logs, message queues).
- **Channel** — buffers events between source and sink; memory-based (fast, not durable) or file-based (durable).
- **Sink** — removes events from the channel and forwards them to the next hop.

A Flume **agent** is a JVM process hosting sources, channels, and sinks.

**Oozie** schedules Hadoop jobs as a directed acyclic graph of actions. Three job types: **workflow jobs** (a sequence of actions with control flow and decision points), **coordinator jobs** (recurring workflows triggered by time or by data appearing in HDFS), and **bundle jobs** (a package of coordinators and workflows). It handles dependencies between jobs and provides monitoring and error handling. Airflow now fills the same role in many pipelines; see [data pipelining](/citadel/interview/data-pipelining).

## The one idea to keep

Hadoop's core answers three separate questions — where does the data live (HDFS), how is it processed (MapReduce), who gets the CPU and memory (YARN) — and keeping them separate is what let the ecosystem grow. HBase, Hive, Pig, Sqoop, Flume, and Oozie are each one missing capability filled in: random access, SQL, scripting, database transfer, log collection, and scheduling. Newer engines replaced the processing layer, but the storage and resource layers underneath are still doing the job.
