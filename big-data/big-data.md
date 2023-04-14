---
title: Big Data - What Makes a Dataset Big and What You Do With It
description: The working definition of big data, the five Vs that describe it, how structured, semi-structured, and unstructured data differ, the path from punched cards to Hadoop and the cloud, and where the analytics pays off in social media, fraud detection, and retail.
date: 2023-04-14
draft: false
slug: /big-data/big-data
tags:
  - Big Data
  - Distributed Systems
  - Data Engineering
---

"Big data" is a bad name for a real problem. The problem is not that a dataset is large in some absolute sense — it is that a dataset outgrows the tools you would normally reach for. When a spreadsheet crashes opening the file, or a single-server database takes a day to answer one query, the ordinary approach has run out, and you need a different one: split the data across many machines and split the computation to match.

This post is the map of that territory. It covers the working definition, the five properties people use to describe such datasets (the "Vs"), the kinds of structure data comes in, the forty-year path that led here, and the places the effort actually earns its keep. The mechanics — the [technology stack](/citadel/big-data/tech-foundation), the [Hadoop ecosystem](/citadel/big-data/hadoop), [storage and mining](/citadel/big-data/data-warehouse), and [analytics](/citadel/big-data/data-analytics) — each get their own post.

## A working definition

**Big data** refers to datasets large and complex enough that traditional software, hardware, and analytical techniques cannot capture, store, process, and analyse them within a tolerable amount of time. The threshold is relative: it moves as ordinary hardware gets faster and as tools improve. What stays constant is the shape of the response — horizontal scaling (add commodity machines) instead of vertical scaling (buy a bigger machine), and computation that moves to the data rather than pulling the data to the computation.

## The five Vs

The defining characteristics are conventionally summarised as words starting with V. The original three were volume, velocity, and variety; veracity and value were added to capture what the first three miss.

| V | What it measures | Why it is hard |
| --- | --- | --- |
| **Volume** | Sheer quantity — terabytes, petabytes, exabytes | Storing and scanning it without the cost or time blowing up |
| **Velocity** | Speed of arrival — batch to near-real-time streams | Processing fast enough that the insight is still useful (fraud, ad bidding) |
| **Variety** | Forms and sources — structured, unstructured, semi-structured | Integrating and querying across incompatible shapes |
| **Veracity** | Quality and trustworthiness | Noise, gaps, bias, and inconsistency turn into bad decisions downstream |
| **Value** | Usefulness of what you can extract | Volume alone is a cost; only extracted insight is a benefit |

Two more sometimes appear: **variability** (the flow rate and the format both change over time) and **visualisation** (presenting the results so a human can act on them). Value is the one that matters most — the rest are the cost side of the ledger, and value is the only entry on the benefit side.

## Three kinds of structure

How data is organised — or not — decides which tools can touch it.

- **Structured data** fits a predefined schema: rows and columns in a relational database. Customer records, sales transactions, financial ledgers. Easy to store and query with SQL; this is the well-understood case.
- **Unstructured data** has no predefined format: text documents, emails, social posts, images, audio, video, raw sensor logs. An estimated 80–90% of data generated today is unstructured. Extracting anything from it needs specialised techniques — natural language processing for text, computer vision for images.
- **Semi-structured data** carries tags or markers that impose hierarchy without a rigid schema: JSON, XML, web server logs, many NoSQL records. More flexible than a relational table, more tractable than free text.

## How we got here

The idea of analysing large datasets is not new; the scale and the economics are.

- **1970s–1980s.** Relational databases and SQL become the standard for structured data. **Data warehousing** — collecting data from several source systems into one place for reporting — takes shape.
- **1990s.** The web arrives and data generation jumps. **Data mining** techniques spread as a way to find patterns in the growing pile.
- **Early 2000s.** Google, Yahoo, and Amazon hit data volumes that off-the-shelf software cannot handle. Google publishes papers on **MapReduce** (a programming model for splitting a computation across a cluster) and the **Google File System** (a way to store huge files across commodity disks).
- **Mid-2000s.** Open-source projects reimplement those ideas: **Apache Hadoop** (MapReduce plus a distributed file system) and a wave of **NoSQL** databases built for scale and flexible schemas rather than strict consistency.
- **2010s onward.** Social media, smartphones, the **Internet of Things**, and cloud computing all push volume up and, at the same time, provide affordable infrastructure to store and process it. "Big data" becomes an ordinary term.

## Where the payoff is

The techniques show up across industries. A few concrete patterns:

- **Social networks.** Ranking a content feed and suggesting connections from behaviour and the social graph; targeting ads from demographics and activity; **sentiment analysis** of posts and comments to track brand perception; detecting fake accounts and spam.
- **Fraud detection (general).** **Anomaly detection** flags transactions that deviate from a learned pattern of normal behaviour; **network analysis** maps relationships between accounts and devices to surface fraud rings; **predictive scoring** assigns a risk score to each transaction so high-risk cases get reviewed first.
- **Insurance fraud.** Claims analysis cross-references a claim against history and external sources (public records, social media) for red flags such as staged accidents; underwriting models score the fraud risk of a new application; link analysis connects seemingly unrelated claims to expose organised schemes.
- **Retail.** Customer segmentation and personalised recommendations from purchase history; **market-basket analysis** finds products bought together to inform layout and promotions; demand forecasting and dynamic pricing from sales data, competitor prices, and seasonality; supply-chain optimisation to cut both stockouts and overstock.

The analysis itself splits into four levels of question — *what happened*, *why*, *what will happen*, *what to do* — covered in the [analytics post](/citadel/big-data/data-analytics).

## Where it is heading

- **More data.** Volume keeps growing with IoT, faster mobile networks, and AI applications that both consume and generate data.
- **Tighter AI integration.** Machine learning becomes the default way to find patterns and generate insights rather than a separate step.
- **Real-time by default.** Demand shifts from overnight batch reports toward stream processing that reacts within seconds.
- **Privacy as a constraint.** Regulation (GDPR and its successors) and public expectation push privacy-preserving techniques — **federated learning** (train a model without centralising the data), **differential privacy** (add calibrated noise so individuals cannot be re-identified) — into the standard toolkit.
- **Edge computing.** More processing happens close to where data is generated — on the device or a local server — to cut latency and bandwidth and keep raw data local.
- **Data fabric and data mesh.** Architectural approaches that make distributed data discoverable and governable without forcing it all into one system.

## The one idea to keep

Big data is not a size; it is the point where the ordinary tool breaks and you switch strategy — from one machine to many, and from moving data to code to moving code to data. The five Vs name the pressures that force the switch, but only one of them, value, is a reason to bother: the rest are what it costs you.
