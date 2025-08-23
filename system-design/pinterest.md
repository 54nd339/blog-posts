---
title: Inside Pinterest - The Pinboard Monorepo and Xenon Analytics
description: A 20 GB monorepo whose CI clone once took 40 minutes, the recommendation pipeline that fills the home feed, and Xenon — the Kafka, Flink, and Druid platform that cut analytics latency from days to seconds.
date: 2025-08-23
draft: false
slug: /system-design/pinterest
tags:
  - System Design
  - Case Study
  - Data Engineering
---

Pinterest is a visual discovery engine: billions of image "Pins", organised into Boards, surfaced through search and a heavily personalised home feed for hundreds of millions of users. Three pieces of its engineering are worth a close look — how it manages one enormous repository, how it builds the feed, and how it rebuilt analytics for real time.

## The Pinboard monorepo

Most of Pinterest's code lives in one repository, **Pinboard**, the way Google and Meta do it. It is large: around **20 GB**, over **350,000 commits**, roughly **100,000 modules**, about **60,000 git pulls a day**.

At that size, CI cloning became the bottleneck. Each build's clone took about **40 minutes** — because the pipeline fetched *all* refs, over 2,500 branches, even with shallow-clone options set. The fix was one line in the Jenkins `refspec`: fetch only the branch the build needs (`+refs/heads/master:refs/remotes/origin/master`). Clone time dropped from 40 minutes to about **30 seconds**, a 20x win, from telling Git not to pull 2,500 branches it would never use. Monorepos at scale need this kind of workflow tuning as a standing practice — see [the monorepo trade-off](/citadel/interview/repo).

## The recommendation system

The home feed shows Pins a user is likely to find inspiring, from a pipeline fed by three input sources: the **Pins** themselves (image, description, link), **user interactions** (saves, clicks, close-ups, comments), and the **user profile** (boards, saved Pins, stated interests, demographics).

1. **Candidate generation** — narrow billions of Pins to a relevant subset via collaborative filtering (users with similar taste), content-based matching, and traversal of Pinterest's interest graph.
2. **Ranking** — ML models score candidates for predicted engagement for that user.
3. **Filtering and diversification** — remove near-duplicates and low-quality content, and cap how many Pins come from one source, board, or micro-topic so the feed stays varied.
4. **Serving** — the ranked, filtered list goes to the home feed.

The supporting stack: **Spark** for batch feature engineering and model training, **Kafka** for ingesting interaction and Pin-creation events, **Storm** (later Flink) for stream processing, **S3** for training data and model artifacts, **Redshift** for the warehouse behind the models, and heavy **A/B testing** to compare algorithm versions.

## Xenon: real-time analytics

Pinterest needs timely numbers for ads reporting, experiment analysis, and creator analytics. The old platform, built on **Hive and Spark** batch jobs, was powerful but slow — insights took hours or days.

**Xenon** re-architected it for low latency:

- **Ingestion** — raw events (actions, ad impressions) arrive through **Kafka**.
- **Stream processing** — **Flink** aggregates over time windows, joins event streams with dimensional data (enriching events with user and Pin metadata), and filters and transforms.
- **Serving** — processed data is served from **Druid**, a real-time OLAP store built for fast ad-hoc aggregation on large datasets.
- **Data lake** — raw events also land in an **S3** data lake for history and batch reprocessing.

Result: data latency from hours or days down to seconds or minutes, plus lower processing cost.

Redis handles caching and MySQL holds structured transactional data like profiles and board metadata.

## The takeaway

Two of these three stories are the same move — a batch system replaced or bypassed because its latency stopped being acceptable: 40-minute clones became 30 seconds, day-old analytics became seconds-fresh. The recurring pattern here and at [Uber](/citadel/system-design/uber) is Kafka for ingestion, Flink for the stream, and a purpose-built OLAP store (Druid, Pinot) for the queries. See [data pipelines](/citadel/interview/data-pipelining) for that shape in general.
