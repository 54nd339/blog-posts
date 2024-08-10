---
title: Vector Databases and Approximate Nearest-Neighbour Search
description: Embeddings turn text, images, and users into points in a few hundred dimensions where "similar" means "close". Finding the nearest points exactly does not scale, so vector search uses approximate indexes - graph-based HNSW or partition-based IVF - that trade a few percent of recall for a thousandfold speedup.
date: 2024-08-10
draft: false
slug: /tech/vector-search
tags:
  - Tools
  - Databases
---

An **embedding** model maps a piece of content — a sentence, an image, a product, a user's history — to a vector of a few hundred to a few thousand floating-point numbers, arranged so that semantically similar things land near each other. "How do I reset my password" and "I forgot my login" end up close; a photo of a golden retriever ends up near other dogs.

Once your data is vectors, a lot of problems become "find the nearest vectors to this query vector": semantic search, retrieval for [LLM prompts](/citadel/system-design/ai-llms) (RAG), recommendations, deduplication, anomaly detection. The catch is that finding the *exact* nearest neighbours in high dimensions is slow, so vector search is built on **approximate** methods.

## Why exact nearest-neighbour doesn't scale

To find the `k` closest of `N` vectors to a query, you compute `N` distances and keep the smallest `k` — O(N·d) per query. For 100M vectors of dimension 768 that's ~77 billion multiply-adds per query. Worse, the usual tree structures (kd-trees, ball trees) that give O(log N) in 2D or 3D **degrade to linear scan** in high dimensions — the "curse of dimensionality," where distances between points become nearly uniform and pruning stops working.

So you give up exactness. **Approximate nearest-neighbour** (ANN) indexes return *almost* the true nearest neighbours — measured as **recall@k** (what fraction of the true top-`k` you retrieved) — in a millisecond instead of a second. Recall of 0.95–0.99 at a 100–1000× speedup is the typical operating point, and for search and recommendations, missing one of the true top-10 occasionally doesn't matter.

## Graph indexes: HNSW

**Hierarchical Navigable Small World** is the most widely used ANN structure. Build a graph where each vector is a node connected to its approximate nearest neighbours, in layers:

- The bottom layer connects every node to its nearby neighbours.
- Each higher layer is a sparse sample of the one below, with longer-range links — like express lanes.

A search starts at an entry point in the top layer, greedily walks toward the query (move to whichever neighbour is closer), drops to the next layer down when it can't improve, and repeats, doing a best-first search in the bottom layer to collect candidates.

Parameters:

- **M** — links per node. More links = better recall, more memory.
- **ef_construction** — how hard the build searches for good neighbours. Higher = better graph, slower build.
- **ef_search** — how many candidates the query keeps in flight. The runtime recall/latency dial — turn it up for more recall, down for speed.

HNSW gives excellent recall at low latency, supports incremental inserts, but is **memory-hungry** (the whole graph plus vectors in RAM) and deletes are awkward (usually a tombstone + periodic rebuild).

## Partition indexes: IVF and quantization

**Inverted File** (IVF): run k-means to cluster the vectors into `nlist` cells, each with a **centroid**. To index a vector, assign it to its nearest centroid's cell. To search, find the `nprobe` centroids nearest the query and scan only those cells.

- `nlist` large + `nprobe` small = fast, lower recall (you might miss cells near a boundary).
- `nprobe` is the recall/speed dial.

IVF alone still stores full vectors. **Product quantization** (PQ) compresses them: split each vector into `m` sub-vectors, run k-means on each sub-space (256 centroids → 1 byte), and store each vector as `m` bytes of centroid IDs. A 768-dim float32 vector (3 KB) becomes ~96 bytes — a 30× reduction — and distances are computed approximately from precomputed centroid-distance tables. **IVF-PQ** combines both: partition to narrow the search, quantize to shrink memory. FAISS's workhorse for billion-scale on modest RAM.

**ScaNN** (Google) and **DiskANN** (Microsoft, keeps most of the index on SSD) are further refinements for scale.

## Distance metrics

- **Cosine similarity** — angle between vectors, ignores magnitude. Standard for text embeddings.
- **Dot product (inner product)** — cosine if vectors are normalized; used when magnitude carries meaning (some recommendation models).
- **Euclidean (L2)** — straight-line distance.

Normalize your vectors and cosine and dot product coincide, which simplifies the index. The metric must match what the embedding model was trained with.

## Hybrid queries

Pure vector search often isn't enough:

- **Metadata filtering** — "nearest documents *from this user, in English, created this year*." Pre-filtering (restrict the candidate set first) is exact but can leave too few candidates for the ANN graph to navigate; post-filtering (retrieve more, then filter) can drop below `k`. Good vector DBs do filtered search *within* the index traversal.
- **Sparse + dense** — combine a keyword score (BM25, exact term matches) with the vector score, because embeddings are weak at exact matches (product codes, names, rare jargon). Reciprocal rank fusion or a weighted sum.
- **Reranking** — retrieve ~100 candidates cheaply with ANN, then rerank the top few with a slower, more accurate cross-encoder model.

## Operational concerns

- **Build cost** — indexing millions of vectors takes minutes to hours; HNSW builds are especially heavy.
- **Updates and deletes** — inserts are usually fine; deletes tombstone and need periodic compaction/rebuild.
- **Memory sizing** — for in-memory indexes (HNSW), budget `N × (d × 4 bytes + M × 8 bytes)` plus overhead; quantization is how you fit more.
- **Sharding** — partition vectors across nodes, query all shards, merge — same fan-out as any distributed index.

## The landscape

- **Libraries** — FAISS (Meta; the reference, embed it in your app), Annoy, hnswlib, ScaNN. No server, no persistence layer — you manage it.
- **Databases with vector support** — `pgvector` (Postgres extension; great when your data is already in Postgres and volume is modest), Elasticsearch/OpenSearch kNN, MongoDB Atlas, Redis.
- **Dedicated vector databases** — Milvus, Qdrant, Weaviate, Pinecone (managed). Built around ANN indexes with filtering, sharding, replication, and hybrid search as first-class features.

Rule of thumb: a library or `pgvector` for up to a few million vectors alongside existing data; a dedicated vector DB when you're at tens of millions or more, need heavy filtered/hybrid queries, or want it managed.

## The one idea to keep

Embeddings make "similar" mean "close in vector space," turning search and recommendation into nearest-neighbour queries — but exact nearest-neighbour degrades to a linear scan in high dimensions, so vector search uses approximate indexes. HNSW builds a navigable multi-layer graph and walks it greedily toward the query (tune `ef_search` for recall vs latency); IVF-PQ partitions into cells and compresses vectors to bytes for billion-scale on limited RAM. Real queries also filter by metadata and blend in keyword scores, because embeddings alone miss exact matches.
