---
title: Storing and Mining Big Data - NoSQL Models, Warehouses, and Data Mining
description: How relational databases, NoSQL stores, data warehouses, and data lakes divide the storage problem in the big data era, and the data mining techniques - association rules, classification, clustering, and their specialised variants - that pull patterns out of what you have stored.
date: 2023-04-29
draft: false
slug: /big-data/data-warehouse
tags:
  - Big Data
  - Databases
  - Data Engineering
---

Storing big data and analysing it are two problems, and this post takes both. The first half is about where the data goes: why relational databases hit a wall at scale, what NoSQL trades away to get past it, and how warehouses and lakes split the job. The second half is about getting knowledge back out — **data mining**, the search for patterns, correlations, and anomalies in data you already hold.

It follows on from the [big data overview](/citadel/big-data/big-data) and pairs with the [analytics post](/citadel/big-data/data-analytics), which covers the analytics process and streaming.

## Where relational databases stop

**Relational database management systems** have run business data for decades: structured tables, a fixed schema, SQL, and **ACID** guarantees ([atomicity, consistency, isolation, durability](/citadel/interview/db-isolation)). They are mature and give strong consistency, and for transactional systems they remain the right choice.

Big data strains them in three places:

- **Scaling.** Relational databases scale vertically — a bigger machine — which is expensive and has a ceiling. Scaling horizontally across many machines is possible but hard.
- **Flexibility.** A rigid schema does not fit the variety of big data — unstructured text, evolving semi-structured records.
- **Velocity.** Very high, rapidly changing ingestion rates are difficult to keep up with.

## NoSQL: what it trades

**NoSQL** ("not only SQL") databases relax the relational model to get scale and flexibility. They typically offer flexible or absent schemas, horizontal scalability across commodity servers, a distributed design from the start, and often eventual rather than strong consistency. Four models cover most of them:

| Model | Shape | Good for | Examples |
| --- | --- | --- | --- |
| **Key-value** | Opaque value under a key | Fast simple lookups, caches, sessions | [Redis](/citadel/tech/redis), DynamoDB, Memcached |
| **Document** | Self-describing documents (JSON/BSON) | Semi-structured data, evolving schemas | [MongoDB](/citadel/tech/mongo), Couchbase |
| **Column-family** | Rows with dynamic columns in families | Wide sparse tables, column-oriented reads | Cassandra, HBase |
| **Graph** | Nodes, edges, properties | Connected data — social graphs, recommendations, fraud rings | Neo4j, Amazon Neptune |

Two supporting ideas:

- **Schema-on-read.** Relational databases enforce schema-on-write: data must fit the schema when inserted. Many NoSQL stores are schema-on-read: data goes in as-is, and the application imposes structure when it reads. This suits diverse and changing data.
- **Materialised views.** A regular view recomputes its query every time; a **materialised view** stores the pre-computed result physically and refreshes it periodically. It trades storage and staleness for fast reads on expensive joins and aggregations — common in warehousing and increasingly offered by NoSQL systems for pre-aggregated reads.

**Distribution.** NoSQL stores spread data two ways, usually together: **replication** keeps copies on multiple servers for availability and read throughput (master-slave or peer-to-peer — see [read replicas](/citadel/interview/read-replica-pattern)), and **partitioning** (sharding) splits the dataset into chunks, each server owning a subset, which is what enables horizontal scale. **Sharding** routes each record to a server by a **shard key**; a poorly chosen key creates hot spots where some shards are overloaded, and operating a sharded cluster adds complexity. The [data sharding](/citadel/interview/data-sharding) post covers key selection in depth.

**Polyglot persistence** is the acknowledgement that no single database fits every need: one application might use a relational database for orders, a document store for the product catalogue, a key-value store for sessions, and a graph database for recommendations. See [choosing a database](/citadel/interview/db-models) and [data systems](/citadel/interview/data-systems).

## Warehouses and lakes

A **data warehouse** is a central repository of integrated, historical, structured data for reporting and analysis. The classic characterisation: **subject-oriented** (organised around customers, products, sales), **integrated** (reconciled from many sources), **time-variant** (historical, not just current), and **non-volatile** (loaded periodically, not updated by users).

- **OLTP** (online transaction processing) systems handle day-to-day operations — many small, fast reads and writes — and use normalised schemas.
- **OLAP** (online analytical processing) systems handle complex queries over large historical datasets and use denormalised schemas — a **star schema** (central fact table, surrounding dimension tables) or a **snowflake schema** (dimensions further normalised).

A **data lake** stores raw data in native format — structured or not — on HDFS or object storage, for exploratory analysis and machine learning. The two are complementary: the lake is for raw storage and data science, the warehouse for established business reporting. Integration strategies include extending ETL to pull summaries from big data systems into the warehouse, **data federation** (querying across both in place), and the **lakehouse**, which adds warehouse features to a lake. The [advanced DBMS](/citadel/dbms/advanced) post covers query optimisation and OLAP mechanics in detail.

Deployment has shifted to the **cloud** — scalable storage and compute, managed services, pay-as-you-go — with hybrid setups keeping sensitive or legacy data on-premise, and serverless architectures for bursty workloads.

## Data mining: the tasks

**Data mining** is the discovery of interesting patterns, correlations, anomalies, and knowledge from large datasets. It sits at the intersection of statistics, machine learning, database systems, and AI. The core tasks:

| Task | Question |
| --- | --- |
| Classification | Which predefined category does this belong to? |
| Clustering | Which items are similar, given no predefined groups? |
| Association rule mining | Which items co-occur? |
| Regression | What continuous value does this predict? |
| Anomaly detection | Which points are unlike the rest? |
| Summarisation | What is a compact description of this dataset? |
| Sequence analysis | What patterns appear in sequences of events? |

Recurring difficulties: data quality (noise, missing values, inconsistency), scaling to large datasets, interpreting results, privacy and ethics, and handling complex data types. Models are evaluated with task-appropriate metrics — accuracy, precision, recall, and F1 for classification; silhouette and Davies-Bouldin scores for clustering; support and confidence for association rules.

## Preprocessing

Quality data comes first — "garbage in, garbage out." Four stages:

1. **Cleaning** — fill or drop missing values, smooth noise, remove outliers, resolve inconsistencies.
2. **Integration** — combine data from heterogeneous sources into one coherent store.
3. **Transformation** — normalisation (scale to a range), aggregation, generalisation, constructing new attributes.
4. **Reduction** — shrink the data while keeping the information: dimensionality reduction ([PCA](/citadel/artificial-intelligence/dimesionality-reduction)), numerosity reduction (sampling, clustering), compression.

**Similarity measures** underpin clustering, k-NN classification, and anomaly detection: Euclidean, Manhattan, and Minkowski distance for numeric attributes; simple matching coefficient and Jaccard index for categorical; cosine similarity for documents.

## Association rules

Association rule mining finds relationships among items in transactional data — "customers who buy X also buy Y."

- **Apriori** is the classic algorithm. It works bottom-up: find frequent single items, extend them to larger itemsets, and prune any candidate whose subset is infrequent. This pruning rests on the **Apriori principle** — every subset of a frequent itemset is itself frequent — so an itemset with an infrequent subset cannot be frequent and need not be counted.
- **Incremental** variants update the rules as new transactions arrive without rescanning the whole database.
- Rule quality: **support** (fraction of transactions containing the itemset), **confidence** (conditional probability that a transaction with X also has Y), and **lift** (how much more likely Y is given X versus Y on its own — lift greater than 1 means positive correlation).

## Classification

Building a model that predicts a categorical label from labelled training data. The algorithm families:

- **Statistical** — Bayesian classifiers such as **Naive Bayes**, which applies Bayes' theorem with an independence assumption between features; simple and effective for text.
- **Distance-based** — **k-nearest-neighbours**, which labels a new point by majority vote of its k closest neighbours; a lazy learner with no training phase.
- **Decision trees** — ID3, C4.5, CART. Each internal node tests an attribute, each leaf is a class label. Easy to read.
- **Advanced** — genetic algorithms for feature selection, rough set theory for minimal feature sets under uncertainty, fuzzy set theory for graded membership, and **neural networks**, with deep networks now state of the art on image and text classification.

The [machine learning](/citadel/artificial-intelligence/ml) post derives these in depth.

## Clustering

Grouping objects so that members of a cluster are more similar to each other than to members of other clusters — an unsupervised task. Two main families:

- **Hierarchical** — build a tree of clusters (a dendrogram). **Agglomerative** starts with each object as its own cluster and merges the closest pairs; **divisive** starts with one cluster and splits recursively.
- **Partitional** — divide the data into k non-overlapping clusters. **k-means** alternates assigning points to the nearest centroid and recomputing centroids; it is sensitive to the initial centroids. **k-medoids** (PAM) uses actual data points as centres, which is more robust to outliers.

For large databases, scalability drives the choice: sampling, grid-based methods (STING), and density-based methods (DBSCAN, OPTICS) that find arbitrarily shaped clusters and handle noise. For categorical data, **k-modes** replaces means with modes. See [unsupervised learning](/citadel/artificial-intelligence/unspervised-learning) for the k-means and PCA derivations.

## Specialised mining

Data mining extends to particular data types:

- **Web mining** — *content* mining (information from page content), *structure* mining (link analysis, PageRank to find authoritative pages), *usage* mining (access patterns from server logs and clickstreams).
- **Spatial mining** — patterns in geographic or astronomical data: spatial clusters, co-location patterns, spatial outliers.
- **Temporal mining** — trends, seasonality, and sequential patterns in time-series data.
- **Text mining** — sentiment analysis, topic modelling, summarisation, named entity recognition. See [NLP](/citadel/artificial-intelligence/nlp).
- **Multimedia mining** — feature extraction, similarity search, and content-based retrieval over images, audio, and video.

## The one idea to keep

Storage and mining are the two ends of the same pipeline. The storage choice is a series of trade-offs — consistency for scale, schema rigidity for flexibility, freshness for query speed — and there is no single winner, which is why real systems use several stores at once. Mining then works on whatever you kept: association rules for co-occurrence, classification for known categories, clustering for unknown groups, and specialised variants for text, space, and time.
