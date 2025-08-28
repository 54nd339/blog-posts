---
title: Inside Reddit - Architecture and How the Home Feed Is Built
description: The stack behind posts, comments, and votes — Python servers, PostgreSQL, Redis, a search cluster, RabbitMQ workers — and the multi-stage pipeline that assembles a personalized home feed for tens of millions of daily users.
date: 2025-08-28
draft: false
slug: /system-design/reddit
tags:
  - System Design
  - Case Study
  - Recommendations
---

Reddit's interface is posts, nested comments, and up/down votes that decide ranking. The backend behind it is read-heavy, spiky, and has to turn a firehose of votes and comments into a fresh personalized feed for tens of millions of people a day. This covers the core components and then the feed pipeline in detail.

## The components

- **Web servers (Python)** — first contact for every request: routing, static assets, handing work to the app tier.
- **Application servers (Python)** — the business logic: posting, commenting, voting, subscribing; reading and writing the data stores; sessions and auth.
- **PostgreSQL** — the primary persistent store: accounts, subreddits, posts, comments, votes, subscriptions.
- **Cache (Redis, Memcached)** — essential given the read skew. Holds hot posts and subreddit listings, rendered page fragments for logged-out users, sessions, precomputed feeds, and **vote counts** (bumped in cache first, flushed to Postgres periodically). This is what keeps read load off the database.
- **Search cluster (Solr or Elasticsearch)** — posts, comments, and subreddits indexed from Postgres for full-text search.
- **Async workers (RabbitMQ)** — anything that needn't happen in the request: recomputing vote scores and rankings, updating karma, sending reply notifications, indexing new posts, NSFW and spam classification.
- **Zookeeper** — service discovery, shared configuration, leader election, distributed coordination.

## Building the home feed

For a logged-in user the feed mixes posts from joined communities, posts from recommended communities, and ads. Generating it per user, quickly, for ~50 million daily active users is the hard part.

1. **Candidate generation.** The Home Feed Service gathers raw material from two sources:
   - **Joined communities** — recent, hot, and top posts from the subreddits the user subscribes to, read from a Post DB (Cassandra, sized for the write and read throughput posts demand).
   - **Recommended communities** — a separate Recommendation Service pulls posts from communities the user *doesn't* follow but might like, based on their history, similar users, and trends.

   The two lists are merged.
2. **Filtering.** Drop posts the user has already seen, posts from users or communities they've blocked, and anything failing their content filters (NSFW) or platform policy.
3. **Ranking.** An ML model scores each remaining post for *this* user's likely engagement, using post signals (upvotes, comments, awards), recency, the user's past interactions, community popularity, and inferred interests.
4. **Mixing.** The Ads Service inserts ads into the ranked list; modules like "Who to Follow" and trending subreddits may join too.
5. **Serving.** The assembled feed goes to the client. Redis caches sit after the Recommendation Service, the Joined Communities Service, and the Filtering-and-Ranking stage, so repeat loads skip the expensive work.

This is the same candidate → filter → rank → mix shape as [Twitter's For You timeline](/citadel/system-design/twitter).

## The recurring challenges

- **Hot content.** A viral post or a breaking-news thread dumps a sudden read spike on specific rows. The defence is aggressive caching plus read replicas for hot data — see [cache stampede and hot keys](/citadel/interview/cache-pitfalls).
- **Voting.** Updating counts atomically at scale without races, then reflecting them in rankings fast, is done with in-memory counters flushed to Postgres and asynchronous ranking recomputes.
- **Real-time comments.** Deep comment trees updating live for thousands of concurrent viewers, over [WebSockets](/citadel/interview/socket-program) with tree-shaped data structures.
- **Consistency by criticality.** Strong consistency where it matters (a vote counted once, a post durably saved), [eventual consistency](/citadel/interview/latency-consistency) where it doesn't (global karma, search freshness).

## The takeaway

Reddit's scale story is mostly caching and asynchrony: keep Postgres as the source of truth, serve reads from Redis, and push everything that can wait onto [RabbitMQ workers](/citadel/interview/message-queue). The feed pipeline is where the ML lives, but the thing that keeps the site up under a front-page spike is how little of each request actually reaches the database.
