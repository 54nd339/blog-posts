---
title: How a Search Engine Works - Crawler, Inverted Index, and Ranking
description: "Three stages turn the web into a search box: a crawler that discovers and downloads pages, an indexer that builds an inverted index from their text, and a query engine that matches your words against it and ranks what comes back."
date: 2025-08-07
draft: false
slug: /system-design/search-engine
tags:
  - System Design
  - Interview Prep
  - Search
---

A search engine has to discover billions of constantly changing documents, understand their content, and return a ranked list in a fraction of a second. That splits into three stages — crawling, indexing, querying — each a large distributed system.

![The search engine pipeline: a crawler feeds downloaded pages to an indexer that builds an inverted index and a document store, which a query engine reads to answer and rank user queries](../images/search-engine.png "Crawler, indexer with inverted index, and query engine.")

## The crawler

The crawler (spider, bot) walks the web downloading pages:

1. Start from **seed URLs** — known reputable sites.
2. Fetch each page.
3. Extract its hyperlinks.
4. Add new URLs to the **crawl frontier**, a prioritised queue of what to visit next.
5. Obey **politeness policies**: respect `robots.txt`, and space out requests to the same server with crawl delays.
6. **Deduplicate URLs.** The same content appears under many URLs, so canonicalize each URL to a standard form and check it against a **Bloom filter** — "definitely new" gets added to the frontier, "probably seen" is skipped or checked more carefully. See [deduplicating URLs at web scale](/citadel/system-design/dedupe-urls).

Output: a stream of downloaded pages to the indexer.

## The indexer

The indexer turns page text into a structure built for keyword lookup:

1. **Parse** the document — pull out text, title, headings, meta tags, links, structure.
2. **Tokenize** the text into words.
3. **Remove stop words** — "the", "is", "of" carry little signal and bloat the index.
4. **Stem or lemmatize** — reduce "running", "runs", "ran" to "run" so a query matches other forms.
5. **Build the inverted index** — the core structure: a map from each term to a **postings list** of the documents containing it, along with per-document term frequency and positions (for phrase search). Apache Lucene, which powers Elasticsearch and Solr, is the reference implementation.

Storage is two parts: a **document repository** holding page content for snippet generation, and the **index** itself (the inverted index plus supporting structures like a forward index and the link graph).

## The query engine

1. **Parse the query** — identify keywords, correct spelling, handle operators (`AND`, `OR`, `NOT`, quoted phrases), expand with synonyms.
2. **Look up the inverted index** — retrieve the postings lists for the query terms and intersect or union them to get candidate documents.
3. **Rank** — the hard part. Signals include:
   - **Term frequency (TF)** — how often the terms appear in the document.
   - **Inverse document frequency (IDF)** — how rare the terms are across the whole collection. TF-IDF combines the two.
   - **Link analysis (PageRank)** — pages linked to by many authoritative pages rank higher.
   - **Content quality and freshness**, **user context** (location, history, language, device), **page speed and mobile-friendliness**, and hundreds of other signals, increasingly weighted by machine-learned models.
4. **Build the results page** — take the top documents, extract a snippet around the query terms for each, and format the SERP.

## Around the core

- **Scale and fault tolerance** — every component runs distributed across thousands of servers in multiple data centres.
- **Freshness** — mechanisms to re-crawl and re-index as pages change.
- **Anti-spam** — detecting link farms, cloaking, and other manipulation.
- **Personalization** and **specialized indexes** for images, video, news, scholarly papers, and products.
- **A/B testing** to refine ranking against real user behaviour.

## The takeaway

The inverted index is the whole trick: precompute term → documents once, at index time, so a query is a lookup and a merge rather than a scan of the web. Everything hard then moves to ranking — deciding, among the millions of pages that contain your words, which forty to show. [Twitter's Earlybird and Blender](/citadel/system-design/twitter) are the same crawler-free pattern applied to tweets.
