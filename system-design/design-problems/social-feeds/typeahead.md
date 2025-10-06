---
title: Designing Autocomplete - Trie, Top-K, and Freshness
description: Search-as-you-type has to return suggestions in the time between two keystrokes. The structure is a trie with the best completions precomputed at every node, sharded by prefix, rebuilt from query logs, and kept warm entirely in memory.
date: 2025-10-06
draft: false
slug: /system-design/typeahead
tags:
  - System Design
  - Design Problems
---

You type `har` into a search box and, before you reach for the `d`, a dropdown offers `harry potter`, `harvard`, `hard drive`. The whole round trip — keystroke to rendered suggestions — has a budget of maybe 100 ms, and a chunk of that is network. The server has single-digit milliseconds to turn a prefix into the five best completions out of hundreds of millions of candidate phrases.

That constraint rules out anything that touches disk or scans. The answer is a **trie** with the top completions cached at every node, held in RAM, sharded by prefix, and rebuilt offline from what people actually searched for.

## What it has to do

- Given a prefix, return the top `k` (5–10) completions, ranked by popularity and recency.
- p99 well under 100 ms end to end.
- Reflect trending queries within hours, not weeks.
- Handle typos, and filter junk / unsafe suggestions.

## The trie, with top-k baked in

A **trie** (prefix tree) has one node per prefix; the path from the root spells the prefix, and children extend it by one character. To find completions of `har`, walk to the `har` node and collect the phrases in its subtree.

Collecting the subtree at query time is too slow for a popular prefix with millions of descendants. So **precompute**: at every node, store the top `k` phrases in its subtree, already ranked.

```
root
 └── h ── a ── r        top-k: [harry potter, harvard, hard drive, harley, ...]
              ├── d      top-k: [hard drive, hardware, hard disk, ...]
              ├── r      top-k: [harry potter, harry styles, ...]
              └── v      top-k: [harvard, harvest, ...]
```

A lookup is now: walk `len(prefix)` nodes, return the stored list. O(length of prefix), independent of how many completions exist. The trie is compressed (**radix tree** — chains of single-child nodes collapse into one edge labelled with a substring) to cut memory and pointer chasing. Facebook and others use a **finite-state transducer** for the same job with even less memory.

## Building the index

The ranking signal is the **query log** — what users searched, how often, how recently. An offline pipeline:

1. Aggregate raw query logs into `(phrase, count)` over a trailing window, with a **time decay** so last week's spike fades (`score = Σ e^(-λ·age)` per occurrence, or a simpler weighted sum of daily counts).
2. Filter: drop rare phrases (below a count threshold), profanity, PII-shaped strings, anything on a blocklist.
3. Build the trie bottom-up, propagating each node's top-`k` from its children's top-`k` (a `k`-way merge).
4. Serialize and ship the immutable trie to the serving fleet.

A full rebuild every few hours keeps suggestions current for most queries. It's an immutable artifact, so deploys are atomic — build the new trie, load it, flip a pointer.

## Freshness for trending terms

A few-hour rebuild is too slow for a breaking-news term that goes from zero to millions of searches in minutes. Run a **streaming counter** alongside: query events flow through Kafka into a windowed aggregator (Flink / Spark Streaming) that maintains counts over the last few minutes. A lightweight process periodically merges the hottest streaming terms into the live trie's affected nodes' top-`k` lists — a small in-place patch, not a rebuild.

## Serving at scale

- **Everything in RAM.** The trie for a large search engine is tens of GB — it fits, and it must, because a disk seek blows the latency budget.
- **Shard by prefix.** Route by the first 1–3 characters: node group A owns prefixes `aa`–`az`, group B owns `ba`–`bz`, and so on. Each request goes to exactly one shard. Replicate each shard 3× for throughput and failover.
- **Client-side help.** The client debounces (waits ~150 ms after the last keystroke before sending), caches suggestions for prefixes already fetched, and can answer a longer prefix locally by filtering a shorter prefix's cached results — cutting request volume by more than half.

## Typos and fuzzy matching

`gogle` should still suggest `google`. Options, cheapest first:

- **Deletion neighbourhoods** — at index time, also index each phrase under prefixes with one character deleted, mapping back to the real phrase. Cheap lookup, catches single-edit typos.
- **Edit-distance search** over the trie — a bounded BFS that allows a few insert/delete/substitute steps. More thorough, more expensive.
- **Phonetic** (Soundex/Metaphone) keys for "sounds like" matches.

Blend the fuzzy results below the exact-prefix results in the ranking.

## The one idea to keep

Autocomplete has no time to search, so it doesn't: a trie stores the top-`k` completions at every prefix node, precomputed offline from query logs with a recency decay, held entirely in RAM and sharded by the first few characters. A lookup is a short walk down the tree to a ready-made list. Trending terms that a slow rebuild would miss get patched in from a streaming counter, and the client debounces and caches so most keystrokes never reach the server at all.
