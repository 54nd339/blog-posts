---
title: Pinning Down Performance - A Look Inside Pinterest's System Architecture
description: How does Pinterest serve a personalized visual feast to millions and provide real-time analytics? We explore its monorepo strategy (Pinboard), recommendation engine, and the Xenon real-time analytics platform powered by Kafka, Flink, and Druid.
date: 2024-08-05
draft: true
slug: /pensieve/pinterest
tags:
  - system-design
  - SDE
---

Hey everyone, and welcome back to the blog! Pinterest. For many of us, it's that delightful "visual discovery engine" where we find inspiration for everything from new recipes and home decor ideas to travel destinations and style trends. Users "pin" images and links to their "boards," creating a personalized world of ideas. Here in Bengaluru, a city teeming with creativity and a love for visual aesthetics, Pinterest is a go-to platform for countless individuals.

But behind that seemingly simple, endlessly scrolling canvas of beautiful images lies a sophisticated and massively scalable backend system. How does Pinterest manage its vast codebase? How does it deliver a highly personalized home feed to millions of users in real-time? And how does it crunch the numbers for analytics at scale? Let's take a peek into some of the key aspects of Pinterest's system design, drawing from insights shared by their engineering teams.

## What is Pinterest? More Than Just Pinboards

At its core, Pinterest is a visual discovery engine. Its mission is to help people discover and do what they love. This involves:
* **Visual Content:** Handling billions of images (Pins).
* **Organization:** Users organize Pins into Boards.
* **Discovery:** Helping users find new ideas through search, recommendations, and Browse.
* **Personalization:** Tailoring the experience (especially the home feed) to each user's tastes and interests. This is critical for engagement.
* **Scale:** Serving hundreds of millions of users globally.

## Managing a Massive Codebase: The Pinterest Monorepo ("Pinboard")

One of the interesting aspects of Pinterest's engineering is its use of a **monorepo** – a single, unified version control repository for a large portion of its codebase. This approach is also famously used by companies like Google and Meta.

* **The Scale of "Pinboard" (Pinterest's Monorepo):**
    Pinboard is enormous:
    * Around **20GB** in size.
    * Containing over **350,000 commits**.
    * Comprising roughly **100,000 modules**.
    * Seeing about **60,000 Git pulls per day** by developers.
* **The Challenge with Scale: CI/CD Bottlenecks:**
    With such a large monorepo, Continuous Integration (CI) build times became a significant challenge. Initially, cloning the repository for each CI build could take around **40 minutes**, primarily because the build pipeline was fetching all refs (over 2,500 branches) even when using shallow clone options.
* **The Solution: Optimizing Git Clones:**
    Pinterest's engineers found a clever solution: a one-line change in their Jenkins build pipeline's Git `refspec` option. By explicitly telling Git to fetch only the `master` branch (or the specific branch relevant to the build) for shallow clones (e.g., `+refs/heads/master:refs/remotes/origin/master`), they dramatically reduced the amount of data fetched. This simple but impactful optimization slashed the clone times from 40 minutes down to about **30 seconds** – a 20x improvement!
    This highlights a key aspect of working with monorepos at scale: the need for specialized tooling and continuous optimization of development workflows.

## Powering Discovery: The Personalized Recommendation System

A core part of the Pinterest experience is the home feed, which is highly personalized to show users Pins they are likely to find inspiring. This is driven by a complex recommendation system.

* **Goal:** To improve user experience by surfacing the most relevant and engaging Pins for each individual user.
* **Key Components & Workflow:**
    1.  **Input Sources:** The system gathers data from various sources:
        * **Pins:** The content and metadata of the Pins themselves (images, descriptions, links).
        * **User Interactions:** How users engage with Pins (likes, comments, saves/repins, clicks, close-ups).
        * **User Profile:** Information about the user, including their boards, saved Pins, explicit interests, and demographics.
    2.  **Candidate Generation:** From the billions of available Pins, this stage identifies a smaller, relevant subset of candidate Pins for a specific user. Techniques might include:
        * Collaborative filtering (finding users with similar tastes).
        * Content-based filtering (matching Pin content to user interests).
        * Graph traversal on Pinterest's "interest graph."
    3.  **Ranking:** The candidate Pins are then scored and ranked based on their predicted relevance and likelihood of engagement for that particular user. This stage heavily relies on **Machine Learning models**.
    4.  **Filtering & Diversification:** After ranking, further filtering might be applied to:
        * Remove duplicate or near-duplicate Pins.
        * Filter out low-quality or inappropriate content.
        * Ensure diversity in the feed (e.g., not showing too many Pins from the same source, board, or on the exact same micro-topic).
    5.  **Serving:** The final, ranked, and filtered list of recommended Pins is then presented to the user, typically on their home feed.
* **Technologies Used (mentioned for their recommendation systems):**
    * **Apache Spark:** For large-scale batch processing, feature engineering, and training machine learning models.
    * **Apache Kafka:** For ingesting real-time user interaction data and Pin creation events.
    * **Apache Storm (or similar stream processors like Flink later on):** For real-time processing of event streams.
    * **Amazon S3:** For storing training data, model artifacts, and large datasets.
    * **Amazon Redshift:** For data warehousing and analytics that feed into the recommendation models.
    * **A/B Testing:** Extensively used to evaluate the performance of different recommendation algorithms and model versions.

## Real-Time Insights: Pinterest's Analytics Platform (Xenon)

Understanding user behavior, ad performance, and creator engagement in real-time is crucial for a platform like Pinterest. Initially, their analytics relied heavily on batch processing, which introduced significant latency.

* **The Need for Speed:** Pinterest needed real-time analytics for ads reporting (to show advertisers how their campaigns are performing), experimentation analysis (to quickly evaluate A/B tests), creator analytics (to give Pin creators timely feedback), and general user engagement insights.
* **Previous System Limitations:** Batch processing systems built on **Apache Hive and Apache Spark** were powerful but had high latency, with insights often taking hours or even days to become available.
* **The Solution - Xenon: A Real-Time Analytics Platform:**
    Pinterest re-architected its analytics platform, creating Xenon, to provide low-latency insights.
    * **Ingestion:** Raw events (user actions, ad impressions, etc.) are ingested via **Apache Kafka**.
    * **Stream Processing:** **Apache Flink** is used for real-time data processing. Flink performs tasks like:
        * Aggregating data over time windows.
        * Joining event streams with dimensional data (e.g., enriching events with user or Pin metadata).
        * Filtering and transforming data.
    * **Serving Layer:** The processed, aggregated, and near real-time data is then served from **Apache Druid**. Druid is a real-time OLAP (Online Analytical Processing) data store designed for fast ad-hoc queries and aggregations on large datasets.
    * **Data Lake Storage:** Raw events are also persisted in an **Amazon S3 data lake** for long-term storage, historical analysis, and potential reprocessing by batch systems.
* **Benefits Achieved with Xenon:**
    * **Reduced Data Latency:** From hours/days down to seconds or minutes.
    * **Cost Savings:** By optimizing the processing pipeline.
    * **More Timely Decision-Making:** Enabled by faster access to critical analytics.

## Other Key Technologies Glimpsed

While the above systems are major highlights, other technologies play important roles:
* **Caching:** Pinterest uses **Redis** as a distributed caching layer to speed up access to frequently used data.
* **Databases:** **MySQL** is mentioned as one of the databases they use, likely for structured, transactional data like user profiles or board metadata.

## Key Takeaways

* Pinterest, the visual discovery engine, is powered by sophisticated backend systems designed for massive scale, deep personalization, and increasingly, real-time analytics.
* They manage their extensive codebase using a **monorepo ("Pinboard")** and have invested in optimizing their build systems (like Git clone times for CI) to maintain developer productivity.
* Their **recommendation system** is a complex, data-driven engine using ML, Spark, Kafka, and other big data technologies to deliver a personalized home feed.
* For analytics, Pinterest evolved from batch processing to a **real-time platform (Xenon)** leveraging Kafka for ingestion, Flink for stream processing, and Druid for fast OLAP queries, all while using S3 as a data lake.
* They utilize a mix of technologies including Redis for caching and MySQL for persistent storage, common in many large-scale web architectures.

The beautiful and seemingly effortless experience of discovering ideas on Pinterest is the result of deep engineering work across distributed systems, big data processing, machine learning, and real-time analytics.
