---
title: Serverless Databases - Scale to Zero, Pay Per Use
description: A database where you never pick an instance size — it scales capacity automatically with load, down to nothing when idle, and bills by consumption. How it differs from a provisioned cloud database, and when the model fits.
date: 2024-12-05
draft: false
slug: /interview/serverless-db
tags:
  - System Design
  - Interview Prep
  - Databases
---

"Serverless" does not mean no servers — it means the servers are the provider's problem. For a database, that translates to: you never provision an instance size, capacity tracks load automatically, and you are billed for what you actually use rather than for capacity sitting idle.

This covers how that differs from a provisioned cloud database, Aurora Serverless as the reference example, and where the model does and does not fit.

## The three characteristics

- **On-demand** — capacity is allocated when requests arrive.
- **Automatic scaling** — up and down with load, sometimes to zero when idle.
- **Consumption pricing** — billed on actual usage (compute, storage, I/O), not on a provisioned tier.

## Versus a provisioned cloud database

A traditional managed database (RDS, Azure SQL in provisioned mode) already beats on-prem, but you still choose an instance (CPU, RAM, storage) and manage scaling.

| | Provisioned | Serverless |
| --- | --- | --- |
| Provisioning | pick and manage instance sizes | starts on request, can scale to zero |
| Scaling | change instance size or add replicas, often with planning or downtime | automatic, up and down, seamlessly |
| Pricing | pay for provisioned capacity, used or not | pay for consumption; compute and storage billed separately |

The e-commerce example: a serverless database can scale across instances within milliseconds to absorb a promotion's traffic, then scale back down after.

## Aurora Serverless

Amazon's example — an auto-scaling configuration of Aurora (its cloud-native MySQL/PostgreSQL-compatible engine):

- **Auto-scaling** of database capacity to match demand.
- **Decoupled compute and storage**, so each scales independently and billing is more granular. The architecture puts a proxy fleet in front for connection management, a multi-AZ cluster for the engine, and a shared storage volume underneath.
- **Mixed mode** — provisioned and serverless instances in one cluster, so an existing provisioned database can join a serverless pool.

## When it fits

Benefits: less operations work (no capacity planning, provisioning, or scaling to run), real cost savings for **intermittent, spiky, or idle-heavy** workloads, elastic scaling for spikes, and developer time redirected from DBA tasks to application work.

Trade-offs:

- **Workload shape** — it shines for unpredictable or spiky traffic, new apps with unknown load, and dev/test. A steady, high, predictable load can be cheaper on a right-sized provisioned instance.
- **Cold starts** — after scaling to zero, the first request waits for resources to spin back up. Implementations keep shrinking this, but it is real.
- **Vendor lock-in** — serverless database offerings are provider-specific.
- **Feature parity** — some advanced options from the provisioned version may not be there yet.

## It moves the elasticity question to the database

The value proposition is the same one [serverless compute](/citadel/interview/cloud-native) made: stop paying for idle, stop capacity-planning, let load drive allocation. For a database that is harder — state has to persist and stay consistent through scaling events — which is why the compute/storage split matters and why cold starts exist at all. It is the right default for workloads whose load you genuinely cannot predict; for the ones you can, [provisioned scaling](/citadel/interview/scalability) with [read replicas](/citadel/interview/read-replica-pattern) is still often the cheaper answer.
