---
title: Inside Uber - Matching Riders to Drivers in Real Time
description: The geospatial index, matching, and surge-pricing logic behind a ride request, plus a tour of the stack Uber runs it on — H3, DocStore, Cadence, a gRPC service mesh, a Bazel monorepo, and the M3 metrics database.
date: 2025-03-03
draft: false
slug: /system-design/uber
tags:
  - System Design
  - Case Study
  - Geospatial
---

A tap on "Request" starts a real-time logistics problem: find the drivers near a moving point, pick the best one against traffic and direction, hold the trip through half a dozen states, price it against current supply and demand, and settle payment across currencies — for millions of concurrent trips. This walks through the core matching logic and then the stack Uber has described for running it.

## The features

### Geospatial indexing and matching

Millions of riders and drivers stream GPS coordinates continuously. To query "who is near here" fast, Uber built and open-sourced **H3**, a hierarchical hexagonal grid: the globe is divided into hexagonal cells at multiple resolutions, so finding nearby entities is a lookup on cell IDs rather than a distance scan. Matching then picks the *optimal* driver, not just the closest — weighing proximity, availability, traffic, the direction the driver is already heading, rating, and vehicle type — and dispatches the request to that driver.

### Surge pricing

The system tracks the ratio of available drivers to open requests per area, continuously. When demand outruns supply, the price multiplier rises: it pulls more drivers into that area and damps demand until the ratio recovers. That's a real-time aggregation over a lot of moving data.

### Ride lifecycle

A trip moves through states — requested, driver en route, picked up, en route to destination, completed, paid. Each ride is a **state machine**, events driving transitions. Slow or non-urgent steps — driver notifications, post-ride payment — run as asynchronous workflows, orchestrated by **Cadence**, Uber's open-source workflow engine for long-running async processes.

### Payments and ETAs

Payments integrate many gateways and wallets, multiple currencies, fraud detection, and fare calculation including surge, tolls, and fees. ETAs come from predictive models over historical and live traffic plus driver speed, updated continuously against the driver's location.

## The stack

**Mobile and web.** The rider and driver apps use **RIBs** (Router, Interactor, Builder), Uber's cross-platform architecture for modular, testable apps — native Swift on iOS, Java on Android. Web uses **Fusion.js** (a React-based universal framework) and **visualization.js** for geospatial rendering.

**Backend and service mesh.** Microservices in **Java with Spring Boot**. **Uber Gateway**, a dynamic config layer over **NGINX**, is the entry point; services talk over **gRPC** and **QUIC**, with **Apache Thrift** for API definitions. **Flipr** (later UCDP) pushes configuration changes without redeploying. **Cadence** handles workflow orchestration; **uAct** backs the event-driven pieces.

**Databases and storage.**
- **DocStore** — Uber's in-house strongly-consistent distributed database for OLTP, built on top of **MySQL** and **PostgreSQL** with a **RocksDB** storage engine.
- **H3** for geospatial indexing.
- **Big data** — the **Hadoop** ecosystem, **Apache Hudi** and **Parquet** columnar formats for the data lake, **Alluxio** as a caching layer, **Hive** and **Spark** for processing, **Marmaray** for ingestion and dispersal between stores.
- **Time-series and analytics** — **Apache Pinot** (real-time OLAP) and **AresDB** (GPU-accelerated real-time analytics).
- **Streaming** — **Apache Kafka** for pipelines and event streaming, **Apache Flink** for stream processing.

**Build and deploy.** A **monorepo** built with **Bazel**; services containerised with **uBuild** (on Buildkite); `devpod` for consistent dev environments. Custom code-quality tools: **NullAway** (compile-time null checks for Java), **NEAL** (linting), **Piranha** (stale feature-flag removal). Testing: **SLATE** (on-demand environments), **Shadower** (replays production traffic for load tests), **Ballast**, and an experiment platform built on deep learning, parts of which — the **Pyro** probabilistic programming language — are open-sourced. Continuous delivery runs on **Spinnaker**.

**Observability.** **uMetric** (metrics emission, backed by Cassandra), **uMonitor** (alerting on thresholds), **M3** (Uber's distributed time-series database for metrics at scale), an on-call dashboard for incident response. **Peloton** schedules cluster resources across batch and stateful services; **Crane** optimises multi-cloud cost.

## What to take from it

Two threads run through the stack. First, Uber builds infrastructure when the scale breaks existing tools and open-sources it — H3, Cadence, Marmaray, M3, Peloton. Second, almost every choice is about keeping a real-time decision fast: the hexagonal index so matching is a lookup, DocStore for consistent trip state, gRPC and QUIC to shave the service hop, M3 so the metrics that drive surge don't lag. See [gRPC](/citadel/interview/grpc) for the service protocol, [the monorepo trade-off](/citadel/interview/repo), and the sibling post on [map rendering and nearby search](/citadel/system-design/map-rendering).
