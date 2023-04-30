---
title: Big Data Analytics - Questions, Models, Streams, and Scale
description: The four kinds of question analytics answers, how analytic scalability evolved from single machines to the cloud, text and social media analytics, the modelling techniques for regression and forecasting, and how to mine data that arrives as a continuous stream.
date: 2023-04-30
draft: false
slug: /big-data/data-analytics
tags:
  - Big Data
  - Data Engineering
  - Machine Learning
---

Collecting data is the easy part. The value comes from asking questions of it, and analytics is the discipline of answering those questions well — with the right method for the question, a team that can build and interpret the models, and infrastructure that keeps up as the data grows. This post covers the four kinds of question, how the tooling scaled to meet them, analytics on text and social media, the core modelling techniques, and the special case of data that never stops arriving.

It builds on the [big data overview](/citadel/big-data/big-data) and pairs with [storage and mining](/citadel/big-data/data-warehouse).

## Reporting versus analysis

The two words are often used loosely, but they mark different depths.

- **Reporting** answers *what happened*. It organises data into summaries — tables, charts, dashboards — to monitor operations and track KPIs. A monthly sales report by region is reporting.
- **Analysis** answers *why did it happen*, *what will happen next*, and *what should we do*. It is investigative and iterative: explore, spot patterns, test hypotheses, build models, draw conclusions. Working out why sales dropped in a region, forecasting next quarter, and recommending a campaign is analysis.

Reporting gives you information; analysis gives you understanding.

## Four kinds of analytics

| Type | Question | Typical techniques |
| --- | --- | --- |
| **Descriptive** | What happened? | Aggregation, dashboards, descriptive statistics |
| **Diagnostic** | Why did it happen? | Drill-down, data discovery, root-cause and correlation analysis |
| **Predictive** | What is likely to happen? | Regression, time-series forecasting, classification |
| **Prescriptive** | What should we do? | Optimisation, simulation, decision analysis, rule-based systems |

Each level does more work than the last, and prescriptive analytics generally needs the predictive layer beneath it.

## How analytic scale grew

Handling analysis at big data volume took a sequence of steps, each pushing the ceiling higher:

- **Single machines** — analysis limited by one processor's power.
- **Data warehousing and OLAP** — better for structured data, still bounded.
- **MPP (massively parallel processing) databases** — many processors, each with its own memory and disk, working on segments of data in parallel. Teradata, Greenplum.
- **Grid computing** — distributed heterogeneous resources for large-scale computation.
- **Hadoop and MapReduce** — batch processing on commodity clusters.
- **Cloud** — elastic, on-demand storage and compute, with managed Spark, Flink, and warehouse services.
- **In-memory computing** — engines like [Spark](/citadel/tech/apache) processing data in RAM, which sped up iterative analytics by orders of magnitude.

The [advanced DBMS](/citadel/dbms/advanced) post covers the MPP and parallel-database architectures in detail.

## Statistics underneath

Every technique here rests on statistical ideas: **descriptive statistics** (mean, median, variance, standard deviation) to summarise; **inferential statistics** (hypothesis testing, confidence intervals) to generalise from a sample to a population; probability distributions to reason about likelihood; **regression** to model relationships; and **sampling** to work with a representative subset when the full dataset is too large. One distinction does the most damage when ignored: **correlation is not causation** — a relationship between two variables does not mean one drives the other. See [statistics and probability](/citadel/maths/probablity-statistics).

## Running an analysis

Points that decide whether an analysis is worth anything:

- **Define the objective.** What business question, what decision?
- **Understand the data.** Sources, quality, biases, limits. Do exploratory data analysis first.
- **Preprocess.** Cleaning, transforming, and integrating data is usually the most time-consuming step and the one that most affects the result.
- **Choose the method.** Match it to the data type and the question.
- **Validate.** How does the model do on data it has not seen? Guard against overfitting.
- **Communicate.** An insight nobody understands or acts on is worthless; visualisation carries it.
- **Iterate.** Explore, model, refine, repeat.
- **Watch the ethics.** Privacy, fairness, and not baking in bias.

An analytics team spreads across roles: **data scientists** (statistics, ML, modelling), **data engineers** (pipelines, storage, infrastructure), **data analysts** (exploration, reports, visualisations), **domain experts** (context and problem definition), visualisation specialists, and project leads to keep it aligned with business goals.

## Text and social media

Most big data is unstructured text — posts, reviews, documents — and **text analytics** turns it into something analysable by transforming it into a structured form first. Social media is a rich source: user-generated content, profiles and demographics, the follower/friend network, and timestamps and locations.

The **text mining** pipeline:

1. **Collection** — gather text from the relevant sources.
2. **Preprocessing** — tokenisation (split into words or sentences), lowercasing, stop-word removal, stemming or lemmatisation (reduce to root form), part-of-speech tagging, noise removal (HTML tags, stray characters).
3. **Representation** — convert to numbers: bag-of-words, TF-IDF vectors, or word embeddings like Word2Vec and GloVe.
4. **Feature selection** — keep the most informative words or phrases.
5. **Pattern discovery** — apply classification, clustering, topic modelling, or sentiment analysis.
6. **Evaluation** — assess and interpret.

See [NLP](/citadel/artificial-intelligence/nlp) for the language-modelling side.

**Sentiment analysis** (opinion mining) determines whether a piece of text is positive, negative, or neutral. Two approaches: **lexicon-based**, using a dictionary of words pre-scored for sentiment, and **machine-learning-based**, training a classifier (Naive Bayes, SVM, a deep model) on labelled text. It drives brand monitoring, product-review analysis, and market research.

A social-media sentiment pipeline on tweets: collect via the API by keyword or hashtag; preprocess (strip URLs, mentions, special characters; expand slang); classify each tweet; aggregate by time, location, or topic and visualise the trend; read off the insight about public opinion on a brand, product, or event.

## Modelling techniques

Beyond the classification and clustering families covered in the [mining post](/citadel/big-data/data-warehouse), these are the workhorses of predictive analytics:

- **Regression** models a numeric outcome from predictors. **Linear** ($y = mx + c$), **multiple linear** ($y = b_0 + b_1 x_1 + \cdots + b_n x_n$), **non-linear** (polynomial, exponential), and **logistic** regression, which predicts a binary outcome by modelling its probability with a sigmoid. Derived in [machine learning](/citadel/artificial-intelligence/ml).
- **Bayesian modelling** updates the probability of a hypothesis as evidence arrives. **Bayesian networks** are directed acyclic graphs whose nodes are variables and whose edges encode conditional dependencies, used for reasoning under uncertainty, diagnosis, and prediction.
- **Support vector machines** find the hyperplane that separates two classes with the widest margin; the **kernel trick** maps inputs into a higher-dimensional space so a linear boundary there corresponds to a non-linear one in the original space.
- **Time-series analysis** works on data indexed by time — stock prices, sensor readings. The goal is to identify trend, seasonality, and cycles, forecast, and flag anomalies. Techniques: moving averages, ARIMA (autoregressive integrated moving average), exponential smoothing, spectral analysis, and recurrent networks like [LSTMs](/citadel/artificial-intelligence/dl).
- **Rule induction** extracts IF-THEN rules from data. The **sequential covering** algorithm learns one rule, removes the data it covers, and repeats on the rest until no significant rule remains. Decision trees perform rule induction implicitly.

## Mining data streams

Much of today's data arrives as a continuous high-velocity stream. Two properties make this a distinct problem:

- The data is **transient** — it cannot all be stored, so algorithms often get a **single pass**.
- Resources per element are bounded, and the underlying pattern can shift over time — **concept drift**.

A **data stream management system** runs **continuous queries** — queries that run indefinitely over the incoming data — the streaming analogue of a DBMS over static data. Stream mining shows up in network monitoring and intrusion detection, real-time fraud detection, IoT sensor analysis, clickstream analysis, and algorithmic trading.

The techniques are built around doing more with less state:

- **Sampling** — keep a representative subset. **Reservoir sampling** maintains a uniform random sample of fixed size from a stream of unknown length.
- **Filtering** — pass only elements matching a criterion, using a [Bloom filter](/citadel/data-structures/bloom-filters) for probabilistic set membership in tiny space.
- **Counting distinct elements** — estimate the number of unique items with the Flajolet-Martin algorithm or **HyperLogLog**, covered with **Count-Min Sketch** for frequency estimation in the [advanced structures](/citadel/algorithms/advanced) roundup.
- **Windows** — process a slice of the stream rather than all of it. **Sliding windows** hold the last N elements or last T time units; **tumbling windows** are non-overlapping consecutive blocks. The **DGIM algorithm** estimates the number of 1s in a binary stream over a sliding window in logarithmic space.
- **Decaying windows** — weight recent data more heavily than old, reflecting that recent information is usually more relevant.

A **real-time analytics platform** stitches these together — a stream processor, a messaging system, and a fast data store — to ingest, process, and act on streaming data within seconds.

## Frameworks and their limits

Hadoop MapReduce is foundational but has high latency and is awkward for iterative algorithms, which is why [Spark](/citadel/tech/apache) rose to replace it for interactive and machine-learning workloads. NoSQL adoption is driven by the need for scalability, flexible data models, high availability, and lower cost on commodity hardware, with social platforms, e-commerce, and IoT as the canonical case studies. And whatever the pipeline produces, visualisation — Tableau, Power BI, D3.js — is what turns a result into a decision.

## The one idea to keep

Analytics is the discipline of matching a method to a question: descriptive and diagnostic to explain the past, predictive and prescriptive to shape the future. The infrastructure exists to keep those methods feasible as data grows, and streaming forces a rethink — one pass, bounded memory, drifting patterns — that produces its own toolbox of approximate algorithms.
