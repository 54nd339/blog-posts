---
title: Elasticsearch - The Inverted Index as a Service
description: A distributed search engine built on Lucene. Documents are analyzed into terms and stored in an inverted index; queries are scored by BM25; shards and replicas spread it across a cluster. The gotcha is that it is near-real-time and has no cross-document transactions - it is a search index, not a database.
date: 2024-07-30
draft: false
slug: /tech/elasticsearch
tags:
  - Tools
  - Databases
---

`SELECT * FROM articles WHERE body LIKE '%distributed systems%'` is a full table scan, and it won't rank the results, handle typos, or know that "distributed" and "distributing" are related. Full-text search is a different problem from filtering rows, and it needs a different data structure: the **inverted index**.

**Elasticsearch** (and its fork **OpenSearch**) wraps Apache Lucene's inverted index in a distributed, JSON-over-HTTP service. It's what powers site search, log analytics (the ELK stack), and "search-as-you-type" for a huge slice of the web. This post is how the index works, how the cluster is laid out, and the ways it surprises people who treat it like a database.

## The inverted index

A normal (forward) index maps document → its contents. An **inverted index** maps term → the list of documents containing it (the **postings list**), plus positions and frequencies:

```
"distributed" → [doc 3 (pos 12, 47), doc 8 (pos 2), doc 19 (pos 88)]
"systems"     → [doc 3 (pos 13), doc 8 (pos 3), doc 41 (pos 5)]
```

A search for `"distributed systems"` intersects the two postings lists to find documents with both, and uses the positions to check they're adjacent (a phrase match). This is O(matching documents), not O(all documents) — the whole point.

## Analysis: text → terms

Before a field's text goes into the index, it's run through an **analyzer**:

1. **Character filters** — strip HTML, replace characters.
2. **Tokenizer** — split into tokens (usually on whitespace and punctuation).
3. **Token filters** — lowercase, remove stop words, apply **stemming** ("running" → "run"), add synonyms, generate n-grams for partial matching.

The *same* analyzer runs at index time and query time, so `"Distributed Systems!"` and `"distributing system"` both reduce to `["distribut", "system"]` and match. Choosing the analyzer per field is most of the tuning: an `edge_ngram` analyzer for autocomplete, a `keyword` (no analysis) for exact-match fields like status codes, language-specific analyzers for stemming.

## Mappings

The **mapping** is the schema: the type of each field and how it's analyzed. Two field types trip people up:

- **`text`** — analyzed, goes in the inverted index, used for full-text `match` queries. *Cannot* be sorted or aggregated efficiently.
- **`keyword`** — not analyzed, stored as-is, used for exact `term` queries, sorting, and aggregations.

A string field is often mapped as *both* (`title` for search, `title.keyword` for sorting). **Dynamic mapping** — Elasticsearch guessing the type from the first document it sees — is convenient and a footgun: one document with `"count": "12"` (a string) can lock the field to `text` and break aggregations.

## The cluster: shards and replicas

An index is split into **primary shards** (fixed at creation — you can't easily change the count later), each a self-contained Lucene index. Each primary has zero or more **replica shards** (changeable) on other nodes.

- **Writes** go to the primary, then replicate to the replicas.
- **Reads** (searches) are served by any copy — a search fans out to one copy of every shard, each returns its top results, and a **coordinating node** merges them.
- **Failover** — lose a node, and a replica of each lost primary is promoted.

Shard count is the key sizing decision. Too few and you can't spread load or grow; too many and each carries fixed overhead (file handles, memory, cluster-state entries) and merges thrash. Aim for shards in the tens-of-GB range, and for time-series data (logs) use **time-based indices** (`logs-2026.03.14`) with a rollover so old data can be dropped by deleting whole indices.

## Relevance scoring: BM25

When multiple documents match, they're ranked by a relevance score. Lucene's default is **BM25**, an evolution of TF-IDF:

- **Term frequency** — a document mentioning the query term more is more relevant, but with **saturation** (the 20th mention barely adds over the 10th).
- **Inverse document frequency** — a term that appears in few documents is more discriminating; matching "quokka" counts for more than matching "the".
- **Field-length normalization** — a match in a short title outweighs the same match buried in a long body.

Queries combine into a `bool` query (`must` / `should` / `must_not` / `filter`). The distinction that matters: **`filter` clauses don't score and are cached** (yes/no, "is this in the last 7 days?"), while **`must`/`should` clauses score** (contribute to relevance). Putting a date range in `filter` instead of `must` is a common, easy speedup.

## Aggregations

Beyond search, Elasticsearch does analytics: **bucket** aggregations (group by term, by date histogram, by range), **metric** aggregations (avg, percentiles, cardinality), and **pipeline** aggregations (derivatives, moving averages over other aggregations' output). This is what makes Kibana dashboards work — "error count per service per hour" is a nested date-histogram + terms aggregation. It runs over the same shards as search, so heavy aggregations and heavy search compete.

## The database-shaped surprises

- **Near-real-time, not real-time.** An indexed document isn't searchable until the next **refresh** (default: every 1 second). Index a doc and immediately search for it and you may not find it. The refresh interval is tunable (raise it for bulk loads).
- **No transactions.** Each document write is atomic on its own; there's no multi-document transaction, no rollback. Don't use it as your system of record — index *from* your database.
- **No joins.** You can nest documents or use a parent-child mapping, but there's nothing like a SQL join. Denormalize at index time.
- **Segments and merges.** Each Lucene shard is many immutable **segments**; new documents create new small segments, deletes are just tombstones, and background **merges** consolidate segments and purge tombstones. Merges are I/O-heavy; a delete-heavy or update-heavy workload keeps them busy.
- **Updates are delete + reindex.** There's no in-place update; a doc update writes a new version and tombstones the old.

## The stack

- **Beats** — lightweight shippers (Filebeat for logs, Metricbeat for metrics) on each host.
- **Logstash** / **ingest pipelines** — parse and enrich events before indexing.
- **Kibana** — visualization and cluster management.
- Alternatives: **OpenSearch** (the Apache-2.0 fork after Elastic's license change), **Vespa** (search + ML ranking at scale), and for pure log volume, systems like **Loki** or **ClickHouse** that trade full-text flexibility for cheaper storage.

## The one idea to keep

Elasticsearch is Lucene's inverted index made distributed: text is analyzed into normalized terms at both index and query time so "Running Systems" matches "run system", searches intersect postings lists and rank by BM25, and shards plus replicas spread it across a cluster with fan-out-and-merge reads. Treat it as a search and analytics index, not a database — it's near-real-time (1s refresh), has no transactions or joins, and updates by delete-and-reindex, so you feed it *from* your source of truth.
