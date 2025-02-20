---
title: Inside Netflix - Transcoding, Open Connect, and a Microservices Backend
description: How Netflix turns a studio master into thousands of streamable versions, pushes them onto caching boxes inside ISP networks, and runs the surrounding logic as hundreds of AWS microservices with the fault-tolerance tools it open-sourced.
date: 2025-02-20
draft: false
slug: /system-design/netflix
tags:
  - System Design
  - Case Study
  - Streaming
---

Netflix has to solve two very different problems well. One is a batch problem: take a high-quality studio master and turn it into every format, resolution, and bitrate a phone or a 4K TV might ask for. The other is a real-time problem: serve those bytes to millions of concurrent viewers on every kind of connection, and keep the sign-in, browse, and recommendation logic up while doing it. The batch half runs on a purpose-built CDN; the real-time half is one of the earliest large microservices deployments.

## The video pipeline

1. **Ingest and store.** Studio master copies land in **Amazon S3**, which also holds the intermediate assets, chosen for durability at scale.
2. **Transcode.** A master isn't streamable as-is — devices differ in codec support, and bandwidth varies second to second. Netflix's in-house transcoder produces many versions of each title: resolutions from SD through HD, 4K, and HDR; multiple bitrates per resolution for **adaptive bitrate streaming** (the player steps quality up or down as the network changes); and several codecs for device compatibility. The result is hundreds to thousands of files per title, produced in parallel.
3. **Distribute via Open Connect.** Netflix runs its own CDN, **Open Connect**. It ships **Open Connect Appliances** — caching servers preloaded with popular titles — to ISPs, who host them inside their own networks. Content then sits one short hop from subscribers, which cuts latency and startup time and cuts the ISP's transit costs. Commercial CDNs (Akamai, Level3, Limelight) supplement capacity and cover thinner regions. Each edge caches what its region actually watches.

## The backend

The logic behind sign-in, browse, recommendations, billing, and playback authorization runs as hundreds of fine-grained microservices on **AWS EC2**, with **Elastic Load Balancing** spreading both user requests and service-to-service calls across instances.

Netflix open-sourced much of the machinery that makes that many services manageable:

- **Zuul** — the API gateway and dynamic front door: routing, traffic monitoring, auth offload, request shaping, load shedding.
- **Hystrix** — fault tolerance via the **circuit breaker** pattern: isolate a call to a failing dependency and stop retrying it, so one slow service doesn't cascade into a full outage.
- **Eureka** — service registration and discovery, so instances that scale up and down can find each other.

### Data and caching

- **MySQL** for transactional data that needs strong consistency — accounts, billing.
- **Cassandra** for high write throughput where eventual consistency is fine — the prime example is viewing history, which is write-heavy and enormous.
- **EVCache**, a distributed cache built on Memcached, used four ways: as a lookaside cache in front of Cassandra, a transient store for sessions and intermediate results, a *primary* store for precomputed data like a user's ready-assembled homepage, and a fast home for high-read config and A/B assignments.

The recommendation engine sits on top of all this, turning viewing history, ratings, browse behaviour, time of day, and device into the personalised rows on the homepage.

## Delivery and operations culture

Deployment and reliability get the same investment:

- **Spinnaker** — Netflix's open-source multi-cloud delivery platform, driving canary rollouts and immutable infrastructure (Java services, built with Gradle, baked into AMIs).
- **Chaos engineering** — Chaos Monkey and its successors inject failures into production on purpose, so weaknesses surface on a Tuesday afternoon rather than during a launch.
- **Atlas** for near-real-time time-series telemetry, **Kayenta** for automated canary analysis against a baseline, **PagerDuty** for incident alerting.

## The trade being made

Netflix bet heavily on microservices, and the whole toolchain above — Zuul, Hystrix, Eureka, Spinnaker, chaos testing — is the cost of that bet: you need all of it to operate hundreds of services safely. It's not a universal answer. Amazon's own Prime Video monitoring team publicly moved a component *back* from serverless microservices to a monolith to cut cost. The architecture follows the workload, not the trend. See [CDNs](/citadel/interview/cdn) for the Open Connect side, [microservices](/citadel/interview/cloud-native) for the backend style, and [deployment strategies](/citadel/interview/deployments) for canaries and immutable infra.
