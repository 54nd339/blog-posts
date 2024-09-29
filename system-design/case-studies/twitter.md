---
title: Inside Twitter - The Life of a Tweet and the For You Timeline
description: How a tweet gets from Post to a million follower timelines, how search indexes it, and the multi-stage machine-learning pipeline that assembles the algorithmic For You feed in about 1.5 seconds.
date: 2024-09-29
draft: false
slug: /system-design/twitter
tags:
  - System Design
  - Case Study
  - Recommendations
---

Twitter (now X) has two hard problems hiding behind a text box. The first is fan-out: when someone with millions of followers posts, that tweet has to reach millions of timelines fast. The second is ranking: the "For You" feed is not chronological — it's the output of a machine-learning pipeline that sifts hundreds of millions of candidate tweets down to the few dozen you see.

The core write and read paths here come from a 2013 Twitter tech talk; the ranking pipeline comes from the recommendation-algorithm code Twitter open-sourced in 2023. Both have evolved, but the shape holds.

![The write path: a tweet enters the Write API, is annotated by the Ingester, and the Fanout service writes it into followers' Redis-cached home timelines while also triggering the asynchronous push-notification path](../images/twitter.png "The write path of a tweet: Write API, Ingester, Fanout to cached timelines.")

## Posting a tweet: the write path

1. **Write API** takes the tweet in.
2. **Ingester** annotates and tokenizes it so it can be indexed for search.
3. **Fanout** does the distribution work. Its main job is writing the tweet into the **home timeline** of every follower — timelines held in an aggressive Redis cache (the "timeline cache") so a timeline load is a cache read, not a query across everyone you follow. Fanout also kicks off an asynchronous path for mobile push and HTTP pushes to other services.

Precomputing timelines on write is the trade: posting costs more work, but the far more frequent timeline *read* becomes cheap.

This breaks for accounts with tens of millions of followers — writing one tweet into 100 million cached timelines is too much work to do synchronously, and most of those timelines won't be read before the next tweet lands. So Twitter runs a **hybrid**: fan-out-on-write for ordinary accounts, and for the handful of very-high-follower accounts, the tweet is *not* pushed. Instead it's pulled in and merged at read time when a follower loads their timeline. Your displayed timeline is the precomputed cache plus a live splice of whatever the celebrities you follow just posted.

## Reading your timeline

The **Timeline Service** handles a home-timeline pull. It locates and returns your precomputed timeline from the Redis caching layer. Most reads never touch durable storage.

## Search and discovery

- The **Ingester** tokenizes every tweet on the way in, feeding the index.
- **Earlybird** stores Twitter's search index, tuned for fast retrieval against a query.
- **Blender** builds the search results and discovery timelines — Trends and other suggested content.

## The For You timeline

The algorithmic feed runs in roughly 1.5 seconds and has five stages:

1. **Candidate sourcing.** Gather a large pool — around 500 million tweets — from people you follow, topics you engage with, and popular content beyond your graph.
2. **Global filtering.** Cut that pool to about 1,500 candidates using heuristics and safety checks.
3. **Scoring and ranking.** A neural network with roughly 48 million parameters scores each candidate for predicted relevance, drawing on your past interactions. Author signals — including whether the author has a premium subscription — feed the score.
4. **Post-ranking filtering.** Trim for diversity of author and content type so the feed isn't one person or one format repeated.
5. **Mixing.** Interleave the ranked tweets with ads and "Who to Follow" suggestions, then assemble the timeline.

This is a large step up in cost and complexity from a chronological feed, spent entirely on personalisation.

## The historical tech stack

Twitter repeatedly hit the ceiling of existing tools and built its own. A snapshot of "Twitter 1.0":

- **Clients** — Swift (iOS), Kotlin (Android), a Progressive Web App; JavaScript, React, Redux on web.
- **Services** — Finagle, Twitter's async RPC framework, on Mesos for cluster management.
- **Caching** — Pelikan (Twitter's in-memory cache) and Redis.
- **Databases** — Manhattan (Twitter's distributed key-value store), MySQL and PostgreSQL for structured data, FlockDB for the social graph, MetricsDB for time-series metrics.
- **Messaging** — Apache Kafka and Kestrel.
- **Data processing** — Apache Heron (stream processing, successor to Storm), Flume (log aggregation), Scalding (Scala over MapReduce), Tableau.
- **Storage and infra** — Hadoop and blob stores; Twitter data centres alongside AWS and Google Cloud; Puppet for config; Audubon (service registry) and Wilson (RPC tracing).

## The through-line

Both hard problems are solved by moving work off the read path. Fan-out precomputes timelines so a read is a cache hit; the ranking pipeline does its half-second of machine learning and caches the result. Every tweet also carries a roughly time-sortable [Snowflake ID](/citadel/system-design/unique-ids), which is what lets timelines and search merge results from many shards in order.
