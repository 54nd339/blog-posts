---
title: Under the Hood of Uber - The Complex Tech Powering Your Ride
description: Ever wondered how Uber matches riders and drivers in real-time, calculates ETAs, and handles millions of trips? We dive into Uber's tech stack - from its RIBs mobile architecture and gRPC service mesh to DocStore databases, Kafka/Flink streaming, and Spinnaker deployments.
date: 2024-06-24
draft: false
slug: /pensieve/uber
tags:
  - system-design
  - SDE
---

Hey everyone, and welcome back to the blog! Here in Bengaluru, and in countless cities worldwide, hailing an Uber has become second nature for many of us. That seemingly simple tap on your phone unleashes a cascade of complex operations: finding nearby drivers, calculating ETAs, dynamic pricing, navigating the best route, processing payments, and so much more, all in real-time.

But what kind of intricate technological marvel powers this global, real-time marketplace for rides and, increasingly, deliveries? Uber operates at a massive scale, dealing with immense volumes of geospatial data, concurrent users, and the need for ultra-high availability. Today, let's take a peek under the hood and explore some of the key components, architectural decisions, and technologies that make Uber work, based on insights from their engineering blogs and publicly shared information.

## The Core Mission: Connecting Riders and Drivers, Instantly

At its heart, Uber solves a complex logistical problem: efficiently connecting individuals needing a ride (riders) with those providing one (drivers) in real-time. This involves a delicate dance between several key components:

* **Rider App:** The interface for users to request rides, track driver ETAs, make payments, and rate their experience.
* **Driver App:** The tool for drivers to accept ride requests, navigate to pickups and destinations, manage their earnings, and receive new ride opportunities.
* **Backend Platform:** The brains of the operation, encompassing a vast array of microservices responsible for everything from user management and geospatial processing to payment handling and fraud detection.

## Dissecting Uber's Key Features: A System Design Lens
Let's break down some of Uber's core functionalities and the system design thinking that likely powers them:

### 1. Geospatial Indexing & Rider-Driver Matching

This is the magic that finds you a ride.
* **Location Updates:** Millions of riders and drivers are constantly sending their GPS coordinates. The system needs to ingest, store, and query this geospatial data with incredibly low latency. Uber has famously developed and open-sourced **H3**, a hexagonal hierarchical geospatial indexing system, which is likely used as a location-index store library. H3 divides the world into hexagonal cells, making it efficient to find nearby entities.
* **Matching Algorithms:** When a rider requests a trip, the system doesn't just find *any* driver; it finds the *optimal* driver based on proximity, driver availability, traffic conditions, ride direction, and potentially other factors like driver rating or vehicle type. This involves complex algorithms running against real-time geospatial data.
* **Dispatch:** Once a match is made, the system dispatches the ride request to the selected driver and communicates acceptance back to the rider.

### 2. Dynamic Pricing (Surge Pricing)

Ever seen those surge multipliers during peak hours or bad weather? That's dynamic pricing in action.
* **Supply & Demand:** The system continuously monitors the ratio of available drivers to rider requests in specific geographic areas.
* **Real-time Calculation:** When demand outstrips supply, prices surge to incentivize more drivers to come online in that area and to temporarily reduce demand. This requires real-time aggregation and analysis of vast amounts of data.

### 3. Ride Lifecycle Management
A single ride goes through multiple states (e.g., requested, driver en-route, rider picked up, en-route to destination, ride completed, payment processed).
* **State Machines:** Each ride can be modeled as a state machine, with events triggering transitions between states.
* **Asynchronous Workflows:** Many parts of the ride lifecycle, like notifying drivers or processing payments post-ride, are handled asynchronously to ensure the system remains responsive. Uber utilizes tools like **Cadence** for async workflow orchestration.

### 4. Payments & Billing
Handling payments globally involves integrating with numerous payment gateways, managing different currencies, and ensuring security and compliance.
* **Payment Gateway Integration:** Securely processing credit/debit card transactions, digital wallet payments, etc.
* **Fraud Detection:** Sophisticated systems to identify and prevent fraudulent transactions.
* **Billing & Invoicing:** Accurately calculating fares (including surge, tolls, fees) and generating receipts for riders and earnings statements for drivers.

### 5. ETA Calculation & Real-Time Tracking ️

Knowing when your ride will arrive and being able to track it on a map is a core part of the Uber experience.
* **Predictive Algorithms:** ETAs are calculated using historical traffic data, real-time traffic conditions, driver speed, and route information.
* **Continuous Updates:** The map and ETA are constantly updated based on the driver's live location data.


## Peeking Under the Hood: Uber's Tech Stack & Architecture

Uber's engineering team has shared insights into their architecture and technology choices over the years. Here's a look at some key areas:

### Frontend & Mobile: The User Interface

* **Web Frontend:** For its web applications, Uber has developed and uses **Fusion.js**, a modern universal web framework built on React. They also utilize **visualization.js** for handling complex geospatial visualizations.
* **Mobile Apps (Rider & Driver):** Uber employs the **RIBs (Router, Interactor, Builder)** architecture for its mobile applications. RIBs is a cross-platform mobile architecture pattern focused on building highly modular, testable, and maintainable apps. It's similar in spirit to patterns like VIPER in its emphasis on separation of concerns. This architecture supports native development with **Swift for iOS** and **Java for Android**.

### Backend Services & Service Mesh: The Engine Room

Uber's backend is a quintessential example of a large-scale microservices architecture.
* **Primary Language:** **Java with the Spring Boot framework** is extensively used for developing backend microservices.
* **Service Mesh & Communication:**
    * **Uber Gateway:** Acts as the primary entry point for requests, built as a dynamic configuration layer on top of **NGINX**.
    * For efficient client-server and inter-service communication, Uber utilizes modern protocols like **gRPC** and **QUIC**.
    * **Apache Thrift** is often used for API definition between services.
* **Configuration Management:** A unified configuration store named **Flipr** (later evolved into UCDP - Unified Configuration Deployment Platform) allows for dynamic configuration changes for services without requiring redeployments.
* **Event-Driven Architecture:** Uber uses **uAct** for components of its event-driven architecture.
* **Workflow Orchestration:** **Cadence**, an open-source, scalable workflow orchestration service developed at Uber, is used for managing complex, long-running asynchronous workflows.

### Databases & Storage: Handling Diverse Data Needs

Uber deals with various types of data, each requiring suitable storage solutions:
* **OLTP Databases (Transactional Data):** For core transactional data, Uber primarily uses **DocStore**, their in-house strongly-consistent, distributed database system. DocStore cleverly employs underlying mature database technologies like **MySQL and PostgreSQL**, often enhanced with the **RocksDB** storage engine for performance.
* **Geospatial Indexing:** **H3**, Uber's hexagonal hierarchical geospatial indexing system, is a crucial open-source library. It divides the world into hexagonal cells at various resolutions, allowing for highly efficient indexing and querying of location data (e.g., finding nearby drivers/riders).
* **Big Data Storage & Processing:**
    * The **Hadoop ecosystem** is utilized for managing and processing massive datasets.
    * Efficient columnar file formats like **Apache Hudi** and **Apache Parquet** are used for their data lakes.
    * **Alluxio** often serves as a distributed caching layer for their big data stack, speeding up access to frequently used data.
    * Large-scale data processing is handled by **Apache Hive** and **Apache Spark**.
    * **Marmaray**, an open-source data ingestion and dispersal framework developed by Uber, facilitates moving data between different data stores and processing systems.
* **Time-Series Data:** For handling vast amounts of time-series metrics and operational data, Uber uses specialized databases like **Apache Pinot** (a real-time distributed OLAP datastore) and **AresDB** (a real-time analytics engine that leverages GPU acceleration).
* **Messaging & Streaming Middleware:**
    * **Apache Kafka:** Used extensively for high-throughput, real-time data pipelines, event streaming, and inter-service communication.
    * **Apache Flink:** Employed for sophisticated stream processing applications.

### DevOps, Build, and Deployment: Shipping Code Reliably

Operating at Uber's scale requires top-notch DevOps practices and tooling:
* **Repository Structure:** Uber utilizes a **Monorepo** strategy, housing a vast amount of its codebase in a single, large repository.
* **Build System:** **Bazel**, Google's open-source build system, is used for building code efficiently at scale within their monorepo. Services are packaged into containers using **uBuild**, which is powered by Buildkite.
* **Simplified Development Environment:** Uber developed `devpod` to provide engineers with a simplified and consistent development environment.
* **Code Quality Tools:** They've built custom tools to enhance code quality, such as **NullAway** (to tackle `NullPointerExceptions` in Java), **NEAL** (for linting code), and **Piranha** (for automatically cleaning up outdated feature flags).
* **Testing & Experimentation:**
    * **SLATE:** Used for managing short-lived, on-demand testing environments.
    * **Shadower:** A load testing tool that replays production traffic to staging or test environments to validate performance and stability.
    * **Ballast:** A tool focused on ensuring a smooth user experience during testing.
    * **Experiment Platform:** Uber has an experiment platform based on deep learning, with parts like **Pyro** (a universal probabilistic programming language) open-sourced.
* **Continuous Delivery:** **Netflix Spinnaker**, a powerful open-source multi-cloud continuous delivery platform, is leveraged for managing complex deployments, likely involving strategies like canary releases or blue-green deployments.

### Monitoring & Observability

Ensuring the health and performance of such a complex system requires robust monitoring:
* **uMetric:** Uber's internal platform for emitting metrics, built on top of **Apache Cassandra**.
* **uMonitor:** Handles alarms and alerting based on defined thresholds from uMetric.
* **M3:** A distributed time series database developed at Uber, used as a comprehensive observability database for storing and querying metrics at scale.
* **OnCall Dashboard:** Helps manage on-call duties and incident response.

### Specialized Operational Tooling

* **Peloton:** Uber's open-source unified resource scheduler, used for managing resources, capacity planning, and scheduling for stateless batch and stateful services.
* **Crane:** A tool developed by Uber for multi-cloud infrastructure cost optimization.

## Key Takeaways

* Uber's platform is a highly complex, real-time distributed system built on a microservices architecture, primarily leveraging Java/Spring Boot for backend services and RIBs for mobile development.
* They utilize a diverse array of specialized databases and storage solutions, including their in-house DocStore (MySQL/PostgreSQL + RocksDB), H3 for geospatial indexing, Cassandra, Kafka, Flink, Pinot, and AresDB, to handle different data types and workloads.
* A sophisticated DevOps culture is evident, with a monorepo managed by Bazel, custom build (uBuild) and testing tools (SLATE, Shadower), and advanced deployment orchestration via Spinnaker.
* Robust observability is achieved through custom monitoring platforms like uMetric, uMonitor, and the M3 time-series database.
* Uber has made significant contributions to the open-source community with projects like H3, Cadence, Marmaray, and Pyro.

The ability to match millions of riders with drivers in real-time, provide accurate ETAs, and process payments globally is a testament to Uber's sophisticated engineering and continuous innovation.
