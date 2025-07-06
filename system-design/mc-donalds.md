---
title: Inside McDonald's - An Event-Driven Architecture on Managed Kafka
description: McDonald's processes the events from a global restaurant operation through a standardised event pipeline — an event registry, custom SDKs, an auth gateway — running on regional AWS MSK clusters that shard by business domain and auto-scale on broker load.
date: 2025-07-06
draft: false
slug: /system-design/mc-donalds
tags:
  - System Design
  - Case Study
  - Event-Driven
---

McDonald's runs a genuinely large event system: millions of orders, point-of-sale transactions, and supply-chain updates a day, from restaurants on every continent. The design problem isn't any single event — it's making tens of thousands of producers and consumers agree on what an event looks like, and keeping the pipeline up as volume swings through the day. Their answer is a standardised, **event-driven architecture** on managed Kafka.

## Standardising the events

The pipeline leans on a few shared components so every team isn't inventing its own event format:

- **Event registry** — the single place that defines the schema for each event type. Every producer and consumer works against the registered shape, so a change is visible and versioned rather than a surprise.
- **Custom SDKs** — libraries teams use to produce and consume the standardised events, with error handling for failed processing built in.
- **Event gateway** — the entry point for incoming events, doing identity authentication and authorization before anything downstream sees the event.
- **Operational tooling** — utilities to repair bad events, check cluster health, and run administrative tasks.

## Scaling it regionally

The processing tier is deployed per region on AWS, for availability and to keep events near where they're produced.

- **Domain sharding.** Within a region, producers shard events by business domain — orders handled separately from inventory, and so on — so one domain's spike doesn't starve another.
- **MSK per domain.** Each domain's events run through an AWS **MSK** (Managed Streaming for Kafka) cluster — Kafka without the broker operations, suited to high-throughput streaming.
- **Auto-scaling.** The MSK clusters scale on metrics like broker CPU. The scaling workflow itself — adding capacity and running partition re-assignment so load actually spreads to the new brokers — is orchestrated with AWS Step Functions.

## Why event-driven fits

Beyond the specifics, the style suits an operation this size: producers and consumers are **decoupled** and evolve independently; the pipeline **scales** with fluctuating volume; a failed consumer just means events wait in the log and get processed on recovery, which is **resilience**; and systems from the till to the supply chain can **react in near real time**. The reusable idea here is the investment in standardisation — a registry, shared SDKs, one gateway — which is what lets an event architecture grow to thousands of participants without fragmenting. See [event sourcing](/citadel/interview/event-sourcing) for the persistence side of this, and [message queues](/citadel/interview/message-queue) for the Kafka-style log itself.
