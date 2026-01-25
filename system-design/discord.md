---
title: Discord's Data Deep Dive - How They Store Trillions of Messages
description: An inside look at how Discord evolved its message storage architecture from MongoDB to Cassandra, and finally to ScyllaDB, to handle trillions of messages and achieve high scalability and performance.
date: 2024-07-06
draft: false
slug: /pensieve/discord
tags:
  - system-design
  - SDE
---

Hey everyone! Discord has exploded in popularity, connecting millions of users through text, voice, and video. But have you ever wondered about the sheer scale of data they handle? We're talking *trillions* of messages! Supporting this kind of volume presents some fascinating engineering challenges, especially when it comes to data storage.

Today, we're going to dive into how Discord has tackled the monumental task of storing and serving these messages, looking at the evolution of their database architecture. It's a great case study in scaling, performance tuning, and making tough engineering choices.

## The Core Challenge: Trillions and Growing

The primary challenge for a platform like Discord is the immense volume of messages generated and accessed constantly. Storing this data efficiently, ensuring fast retrieval, and maintaining high availability as the user base grows requires a robust and scalable backend.

## Discord's Database Evolution for Message Storage

Discord's journey to its current message storage solution is a tale of outgrowing initial choices and adapting to new demands. Their engineering team has shared insights into this evolution, which can be broadly categorized into three phases.

### Phase 1: The Early Days with MongoDB

In its initial stages back in 2015, Discord started with a single MongoDB replica for its message storage. This was a common choice for startups looking for flexibility and ease of use.

* **Initial Scale:** By November 2015, MongoDB was already storing around 100 million messages.
* **Growing Pains:** As the platform grew, MongoDB started showing signs of strain. The primary issue was that the RAM could no longer hold all the data and necessary indexes, leading to unpredictable latency. It became clear that a new solution was needed.

### Phase 2: Scaling with Cassandra

To address the limitations of MongoDB, Discord decided to migrate its message storage to Cassandra. Cassandra, a distributed NoSQL database known for its scalability and fault tolerance, seemed like a good fit.

* **Scaling Up:** By 2017, Discord was running 12 Cassandra nodes and storing billions of messages. This footprint continued to expand dramatically. By early 2022, they had an impressive 177 Cassandra nodes managing trillions of messages.
* **Challenges at Trillion-Message Scale:** Even with Cassandra, new challenges emerged at this massive scale:
    * **Unpredictable Latency:** Similar to the later stages of MongoDB, latency became a significant issue.
    * **Expensive Maintenance:** Operations crucial for Cassandra's health, like compacting SSTables (Sorted String Tables), became very expensive to run and negatively impacted performance.
    * **Read Performance:** Cassandra utilizes an LSM (Log-Structured Merge-Tree) tree structure for its storage engine. In this structure, reads can be more expensive than writes. With hundreds of users potentially in a single server generating many concurrent reads, this led to "hotspots".
    * **Garbage Collection Pauses:** Java-based Cassandra experienced significant latency spikes due to garbage collection pauses.

### Phase 3: Embracing ScyllaDB for Enhanced Performance

Facing these new bottlenecks, the Discord team looked for an alternative that could offer better performance and operational efficiency at their scale. They landed on ScyllaDB.

* **What is ScyllaDB?** ScyllaDB is a NoSQL database designed to be highly performant and Cassandra-compatible (it uses the same API), but it's written in C++. This C++ implementation helps avoid issues like Java's garbage collection pauses. It's also known for its "shards per core" architecture.
* **Architectural Redesign:** The move to ScyllaDB was part of a broader architectural shift that included a monolithic API and a data service written in Rust, with ScyllaDB as the storage backend.
* **Significant Performance Gains:** The results were remarkable:
    * The p99 read latency with ScyllaDB dropped to 15ms, compared to 40-125ms with Cassandra.
    * The p99 write latency improved to 5ms, down from 5-70ms with Cassandra.

This evolution showcases a common pattern in fast-growing systems: solutions that work well at one scale can become bottlenecks at another, requiring continuous re-evaluation and re-architecture.

## A Note on Unique Identifiers

Beyond message storage, another interesting aspect of Discord's infrastructure is how they generate unique IDs at scale. For this, they employ a solution similar to Twitter's Snowflake ID generator. These IDs are designed to be globally unique and roughly sortable by time, which is crucial for distributed systems.

## Learning from Discord's Engineering

Discord's engineering team often shares valuable insights on their technical blog. Their journey through different database technologies for a core feature like message storage provides a wealth of learning for anyone building or scaling large-scale applications.

## Key Takeaways

* **Scalability is a Journey:** Database choices must evolve with the scale and demands of the application.
* **Understand Your Workloads:** Discord's read patterns and the impact of LSM-tree characteristics on Cassandra at scale were key factors in their decision to switch.
* **Technology Choices Have Trade-offs:** MongoDB offered early flexibility, Cassandra offered distributed scale, and ScyllaDB provided enhanced performance and operational efficiency by addressing Cassandra's limitations at extreme scale.
* **Performance Metrics Matter:** Tracking metrics like p99 latency is crucial for identifying bottlenecks and validating improvements.

Discord's story is a testament to the engineering effort required to keep a massive platform running smoothly. It highlights the importance of being willing to revisit architectural decisions and adopt new technologies when necessary.
