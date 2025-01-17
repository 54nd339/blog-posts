---
title: Cloud Disaster Recovery - Trading Cost for RTO and RPO
description: When a whole region goes down, how fast you recover and how much data you lose are set by choices made beforehand. RTO and RPO as the two dials, and the four standard strategies from cold backup to active-active.
date: 2025-01-17
draft: false
slug: /interview/cloud-disaster-recovery
tags:
  - System Design
  - Interview Prep
  - Reliability
---

A region fails — a data-centre outage, a bad deploy that corrupts the primary database, a fibre cut. The two numbers that matter next were decided months ago: how long until service is back, and how much recent data is gone for good. Disaster recovery is the practice of setting those two numbers deliberately instead of discovering them during the outage.

This covers how disaster recovery differs from high availability, the two metrics that drive every decision, and the four standard strategies — each a different point on the line between "cheap but slow" and "instant but expensive".

## Not the same as high availability

**High availability** prevents downtime *within* a location: redundant servers, load balancers, and power, with automatic failover between them, transparent to users. **Disaster recovery** is what you fall back on when the failure takes out the whole location — a region, a data centre — and the local redundancy goes with it. HA handles a dead server; DR handles a dead region.

## The two dials: RTO and RPO

- **Recovery Time Objective (RTO)** — the maximum acceptable time the service is down before the business is hurt. An RTO of 4 hours means the plan must restore service within 4 hours.
- **Recovery Point Objective (RPO)** — the maximum acceptable amount of data loss, measured as a span of time. An RPO of 15 minutes means recovery must lose no more than the last 15 minutes of writes, which sets how often data must be replicated or backed up.

Tighter targets on either dial cost more and add complexity. The strategy you pick is essentially a budget for these two numbers.

## Four strategies, cold to hot

| Strategy | Running in the DR region | Typical RTO | Typical RPO | Ongoing cost |
| --- | --- | --- | --- | --- |
| **Backup and restore** | nothing — just backups in another region | hours to days | hours to a day (backup interval) | lowest (storage only) |
| **Pilot light** | core only — replicated database, minimal compute, switched off | tens of minutes to hours | seconds to minutes (async replication) | low |
| **Warm standby** | a scaled-down but running full stack, taking replicated data | minutes to tens of minutes | seconds to minutes | medium |
| **Active-active** | full-scale stack serving live traffic in every region | near zero | near zero (sync or near-sync replication) | highest (multiple full stacks) |

**Backup and restore.** Back up data and machine images to another region. On disaster, provision fresh infrastructure there, restore from the latest backup, and repoint DNS. Cheapest to hold; slowest to recover, and you lose everything since the last backup.

**Pilot light.** Keep the irreducible core alive in the DR region — the database, continuously replicated — with the compute tier defined but not running. On disaster, scale the compute up to production size and cut traffic over. Faster than a cold restore because the data is already there; the delay is the scale-up.

**Warm standby.** Run a smaller but complete copy of the stack in the DR region all the time, taking near-real-time replication. On disaster, redirect traffic and scale the standby up to full load. Recovery is quick because something is already serving; the cost is running that second stack around the clock.

**Active-active.** Every region runs full capacity and serves live traffic, distributed by latency- or geo-based DNS or global load balancing. A region failing just means its traffic reroutes to regions already handling load, so failover is near-instant. The price is N full production environments plus the hard part: keeping data consistent and resolving write conflicts across active sites.

## What sits around the strategy

- **Replication choice.** Synchronous replication gives near-zero RPO but adds latency to every primary write; asynchronous keeps the primary fast but leaves an RPO window. This is the same [latency-versus-consistency](/citadel/interview/latency-consistency) trade the CAP theorem describes.
- **Infrastructure as code.** The DR environment has to be provisioned fast and identically — Terraform or CloudFormation templates, not a runbook of console clicks.
- **Failback, not just failover.** Returning to the primary region once it recovers is its own procedure, and the one teams most often skip rehearsing.
- **DNS.** Traffic redirection rides on low-TTL records and a routing policy — failover or weighted — that is configured before the incident, covered under [internet routing](/citadel/interview/internet-routing).
- **Drills.** An untested DR plan is a hypothesis. Scheduled failover exercises are how you find the gap before it finds you.

## The plan is a business decision priced in advance

Picking a strategy is stating, in dollars, how much an hour of downtime and a minute of lost data are worth to this system — and then paying that continuously so the bill does not arrive all at once during an outage. Most organisations run a mix: active-active for the revenue path, pilot light or backup-and-restore for everything whose RTO can be measured in hours. The [read-replica pattern](/citadel/interview/read-replica-pattern) and [database scaling](/citadel/interview/data-sharding) choices feed directly into which RPO is even reachable.
