---
title: Time-Series Databases - Built for Timestamped, Write-Heavy Data
description: "Metrics, sensor readings, and traces arrive constantly, timestamped, and get queried in time windows - a pattern general-purpose databases handle badly. How TSDBs specialise for it: high write throughput, delta compression, time-and-tag indexing, and built-in retention policies."
date: 2024-05-15
draft: false
slug: /tech/timeseries-db
tags:
  - Tools
  - Databases
  - Observability
---

A lot of the data a running system produces is the same shape: a timestamp, a number, and some labels. CPU usage at 10:05:01 on `web01`. A temperature reading from a sensor. A stock price. A request latency. It arrives continuously, it's almost never updated after the fact, and you query it in time windows — "average over the last hour", "p99 for this service today".

General-purpose databases can store this, but they're not built for it, and it shows at scale. **Time-series databases** are built for exactly this pattern. This post covers why the pattern is hard for other databases and what a TSDB does differently.

## The data and the access pattern

A **time-series data point** is a timestamp, one or more values, and usually a set of metadata **tags** — `metric_name=cpu`, `host=web01`, `region=asia-south1`. Time is the primary axis.

The workload has a distinctive shape:

- **Writes are heavy and constant** — a large fleet can emit millions of points per second, and they only ever append.
- **Reads are spiky and time-scoped** — dashboards querying the last few minutes, analytics aggregating over windows, alert rules checking thresholds.

### Why general-purpose databases struggle

**Relational (MySQL, Postgres).** A wide table with a column per metric becomes unmanageable as metrics proliferate; a narrow table of `(metric, value, timestamp, tags)` grows into billions of rows where both writes and queries slow down. Time-series operations — moving averages, downsampling, windowed aggregation — are awkward and slow in standard SQL.

**Wide-column NoSQL (Cassandra, Bigtable).** Better at write-heavy, horizontally scaled load, but still not optimised for windowed aggregation, and missing the built-in **data lifecycle management** — retention and downsampling policies — that time-series data needs because it grows forever.

## What a TSDB does differently

- **High write throughput.** Ingest is the design centre. InfluxDB benchmarks show over 250,000 writes per second on a modest 8-core, 32 GB machine.
- **Compression tuned for the data.** Consecutive readings from one series barely differ, so TSDBs use delta encoding, delta-of-delta, run-length encoding, and Facebook's Gorilla compression to shrink storage dramatically.
- **Time-windowed queries as first-class operations.** Query languages like PromQL and InfluxQL make filtering by time range, grouping into buckets (per minute, per hour), aggregating (sum, avg, percentiles), and downsampling concise and fast.
- **Retention and downsampling policies.** Rules like "drop raw data after 30 days" or "roll minute data up to hourly after a week, keep the hourly for a year" are built in — the main lever on storage cost.
- **Indexing on time and tags.** Efficient indexes on the time axis and on the metadata tags, so "average CPU for the `payment-service` in `asia-south1`" is fast. The catch: tags should be **low-cardinality** — a tag with millions of distinct values (a user ID, a request ID) blows up the index.
- **Hot/cold tiering.** Recent data is kept in memory for fast dashboard queries; older data moves to compressed on-disk storage.

## The common TSDBs

- **InfluxDB** — Go, open source, fast, easy to start with. Infrastructure and application metrics, IoT.
- **Prometheus** — a monitoring system with a TSDB inside. It *pulls* metrics by scraping targets, queries with PromQL, and alerts via Alertmanager. The default in cloud-native stacks, usually paired with Grafana. See [metrics and alerting](/citadel/system-design/metric-alerts).
- **TimescaleDB** — a [PostgreSQL](/citadel/tech/postgres) extension, so you get time-series optimisations while querying in ordinary SQL.
- **Amazon Timestream** — AWS's serverless managed option.
- **OpenTSDB** — an early one, built on Hadoop and HBase; scales far but is heavy to operate.
- **In-house** — [Netflix's Atlas](/citadel/system-design/netflix), Twitter's MetricsDB, and [Uber's use of Apache Pinot and AresDB](/citadel/system-design/uber) for real-time time-series analytics.

## The surrounding stack

A TSDB is one stage in an observability pipeline:

- **Collection** — agents and client libraries push metrics, or Prometheus scrapes them on a schedule.
- **Optional stream processing** — raw data can flow through [Kafka](/citadel/tech/kafka) into Flink or Spark Streaming for enrichment or anomaly detection before it's stored.
- **Visualisation** — Grafana queries the TSDB and renders dashboards.
- **Alerting** — Alertmanager (or Grafana) evaluates rules against the stored data and sends notifications when a threshold trips.

## The takeaway

Time-series data is write-heavy, append-only, and queried in windows — a pattern where the general-purpose databases pay for flexibility they don't need. A TSDB trades that flexibility for ingest speed, compression, windowed-query performance, and automatic aging-out of old data. The one design constraint to remember is tag cardinality: keep the label values to a small, bounded set, or the index that makes it fast becomes the thing that makes it slow.
