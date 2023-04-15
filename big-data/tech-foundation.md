---
title: The Big Data Stack - Layers, Virtualisation, Cloud, and In-Memory
description: The layered architecture behind a big data platform from data sources up to visualisation, and the four technologies that make it work - virtualisation and containers, distributed and parallel computing, cloud infrastructure, and in-memory processing.
date: 2023-04-15
draft: false
slug: /big-data/tech-foundation
tags:
  - Big Data
  - Data Engineering
  - Distributed Systems
---

A big data platform is not one product. It is a stack of components — a file system here, a stream processor there, a scheduler, a query engine, a dashboard — each solving one slice of the problem of getting data from where it is born to where it produces a decision. This post walks that stack layer by layer, then covers the four foundational technologies that every layer leans on.

It assumes the [what and why of big data](/citadel/big-data/big-data); the [Hadoop ecosystem](/citadel/big-data/hadoop) and [storage models](/citadel/big-data/data-warehouse) go deeper on individual pieces.

## The layers

The exact products vary, but a big data architecture has a recognisable shape from the bottom up.

**1. Physical infrastructure.** Servers (usually commodity machines clustered together, sometimes specialised nodes), storage devices (HDDs for cheap bulk capacity, SSDs for speed), and fast interconnects (10/40/100 GbE, InfiniBand) so nodes can shuffle data quickly. Deployed on-premise or, increasingly, rented from a cloud provider.

**2. Data sources.** Everything that generates data: transactional databases, CRM and ERP systems, social media, IoT sensors and wearables, public web and APIs, server and application logs, clickstreams, multimedia. The defining trait is variety and velocity — structured, semi-structured, and unstructured, arriving at very different rates.

**3. Ingestion.** Getting data from those sources into the platform: collection, transport, routing, and light pre-processing. [Apache Kafka](/citadel/tech/kafka) for real-time streams, Apache Flume for log aggregation, Apache Sqoop for bulk transfer between Hadoop and relational databases, plus cloud services like AWS Kinesis and Google Cloud Pub/Sub. The hard parts are high throughput, mixed formats, reliable delivery, and the choice between batch and stream. See [message queues](/citadel/interview/message-queue) for the delivery-guarantee trade-offs.

**4. Storage.** Where data lands before and during processing. Distributed file systems (**HDFS**; cloud object stores like Amazon S3, Google Cloud Storage, Azure Blob), NoSQL databases (HBase, Cassandra, MongoDB), and the warehouse/lake family:

- **Data warehouse** — structured, cleaned data for reporting (Redshift, BigQuery, Snowflake).
- **Data lake** — raw data in native format, structured or not, usually on HDFS or object storage.
- **Data lakehouse** — a lake with warehouse features bolted on: ACID transactions, schema enforcement, time travel (Databricks [Delta Lake](/citadel/tech/databricks)).

**5. Platform management.** Cluster resource management and job scheduling: **Apache YARN** and Apache Mesos allocate CPU and memory across applications; [Kubernetes](/citadel/tech/k8s) orchestrates containerised workloads; Apache Airflow, Oozie, and Luigi schedule and monitor multi-step pipelines.

**6. Security.** Authentication (Kerberos, LDAP, OAuth), authorisation (Apache Ranger, Apache Sentry for fine-grained access), encryption at rest and in transit, auditing, and data governance for lineage and lifecycle. Compliance obligations (GDPR, HIPAA) cut across all of it.

**7. Monitoring.** Metrics collection (Prometheus, Ganglia), log aggregation and search (the ELK stack, Splunk), and dashboards (Grafana). Used for performance tuning, troubleshooting, and capacity planning.

**8. Processing.** Where raw data becomes results. Three paradigms:

- **Batch** — large volumes at rest. Hadoop MapReduce (the original), [Apache Spark](/citadel/tech/apache) (faster, general-purpose).
- **Stream** — data in motion. Spark Streaming (micro-batches), Apache Flink (true streaming, low latency), Apache Storm, Kafka Streams.
- **Interactive query** — ad-hoc SQL over large datasets. Apache Hive (SQL compiled to jobs), and the MPP engines [Trino/Presto](/citadel/tech/starbust), Impala, and Spark SQL.

Machine learning libraries (Spark MLlib, Apache Mahout) and distributed training with TensorFlow or PyTorch also live here.

**9. Visualisation.** Presenting results so people can act: BI tools (Tableau, Power BI, Qlik), web libraries (D3.js), and big-data-native dashboards (Apache Superset, Kibana).

## Virtualisation and containers

**Virtualisation** creates a software version of a physical resource — a server, a storage device, a network. A **hypervisor** creates and runs virtual machines, each with its own guest operating system, giving strong isolation between workloads on the same hardware. This lets one physical cluster host several big data jobs — production, dev, test — without them interfering, and lets you provision or tear down capacity to match demand.

**Containers** (Docker) are lighter: they share the host kernel and isolate only the user space, so they start in milliseconds and pack far more densely than VMs. [Kubernetes](/citadel/tech/k8s) is the standard for orchestrating containerised big data workloads. See [containers](/citadel/tech/docker) for the mechanics.

## Distributed and parallel computing

Big data problems do not fit on one machine, so the work is split.

- **Distributed computing** — multiple autonomous nodes cooperating over a network.
- **Parallel computing** — many computations running at once, usually by breaking one task into independent sub-tasks.

Two forms of parallelism matter here:

- **Data parallelism** — the same operation applied concurrently to different slices of the data. This is the dominant form in big data; the "map" phase of MapReduce is exactly this.
- **Task parallelism** — different operations running concurrently on the same or different data.

The costs are real: partitioning data so the work divides evenly, inter-node communication overhead, coordination and synchronisation, and fault tolerance when a node dies mid-job. The [parallel computing](/citadel/parallel-computing/parallel-computing) category covers the shared-nothing model and the speedup limits (Amdahl, Gustafson) in general.

## Cloud

Cloud platforms fit big data workloads well:

- **Elasticity.** Scale a cluster up for a nightly job and down afterward, paying only for what runs.
- **Economics.** No large upfront hardware purchase; a capital expense becomes an operating expense.
- **Managed services.** Amazon EMR, Google Dataproc, and Azure HDInsight run Hadoop and Spark clusters for you; managed NoSQL, serverless processing, and hosted data lakes cut operational load further.
- **Reach.** Deploy across regions for latency and disaster recovery.

**Cloud-native** big data means designing for these capabilities from the start — serverless processing, auto-scaling, direct integration with object storage — rather than lifting an on-premise cluster into a VM.

## In-memory processing

Classic big data processing reads and writes large amounts of data to disk, and disk I/O is orders of magnitude slower than RAM. **In-memory computing** keeps data in the main memory of a cluster instead, which transforms iterative work — most machine learning, interactive queries — where the same data is touched repeatedly.

- **Apache Spark** caches datasets (RDDs, DataFrames) in cluster memory for fast repeated access; this is the main reason it outruns MapReduce on iterative jobs.
- **In-memory databases** (SAP HANA, VoltDB, MemSQL) hold all or most data in RAM.
- **In-memory data grids** (Apache Ignite, Hazelcast, GridGain) pool RAM across many machines into one addressable store.

The trade-offs: RAM costs more per byte than disk, and durability needs a mechanism — periodic snapshots, transaction logs, or replication to nodes with persistent storage — because memory is volatile. The [memory hierarchy](/citadel/computer-architecture/hierarchical-memory) explains why the speed gap exists.

## The one idea to keep

The stack looks complicated because every layer is a different product, but the shape is simple: sources feed ingestion, ingestion feeds storage, storage feeds processing, processing feeds visualisation, and management, security, and monitoring wrap all of it. The four foundations — virtualisation for isolation, parallelism for throughput, cloud for elasticity, and memory for speed on repeated work — are the levers you pull when one layer becomes the bottleneck.
