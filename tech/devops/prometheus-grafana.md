---
title: Prometheus and Grafana - Metrics and Dashboards
description: Prometheus scrapes numeric metrics from your services on a timer, stores them as time series labelled by dimension, and lets you query and alert on them with PromQL. Grafana draws the graphs. The recurring gotcha is cardinality - one unbounded label can multiply your series count into the millions.
date: 2024-10-20
draft: false
slug: /tech/prometheus-grafana
tags:
  - Tools
  - DevOps
---

You want to know your service's request rate, error rate, and p99 latency, per endpoint, over time — and get paged when any of them goes wrong. **Prometheus** is the de-facto open-source answer: your service exposes a text page of current metric values, Prometheus fetches ("scrapes") it every 15 seconds, stores the numbers as time series, and lets you query and alert on them. **Grafana** turns the queries into dashboards.

The model is deliberately simple, and most of the operational pain comes from one place — **cardinality**, the number of distinct time series — so that's where this post spends its time.

## The data model

A **time series** is uniquely identified by a metric name plus a set of key-value **labels**:

```
http_requests_total{method="POST", handler="/checkout", status="500"}  →  [(t1, 4021), (t2, 4023), ...]
```

Every unique combination of label values is a *separate* time series with its own stored history. `http_requests_total` with 5 methods × 20 handlers × 8 status codes = 800 series. That multiplication is the thing to watch.

Four metric **types** (a convention, not enforced by storage):

- **Counter** — monotonically increasing (total requests, total errors). You never read its raw value; you read its *rate*.
- **Gauge** — goes up and down (memory bytes, queue depth, temperature).
- **Histogram** — pre-defined buckets counting observations `≤` each boundary (`request_duration_seconds_bucket{le="0.1"}`, `le="0.5"`, …), plus a `_count` and `_sum`. Lets you compute quantiles *after the fact* and aggregate them across instances.
- **Summary** — client-computed quantiles. Cheaper to query but you **cannot** aggregate them (you can't average two p99s), so histograms are usually preferred.

## The pull model

Prometheus **scrapes** targets — it makes an HTTP GET to `/metrics` on each one and parses the exposition format. This is the opposite of most monitoring systems, which have agents push. Pull has advantages: Prometheus controls the rate, a target being down is itself a signal (`up == 0`), and you can point a browser at `/metrics` to debug. Targets are found via **service discovery** (Kubernetes, Consul, EC2, file) so a new pod is scraped automatically.

The exception is short-lived batch jobs that finish before a scrape — they push to a **Pushgateway** that Prometheus then scrapes. Use it sparingly; it's a common source of stale metrics.

Instrumentation is via client libraries (Go, Java, Python, …) that maintain the counters in your process, or **exporters** — sidecar processes that translate something else's metrics into Prometheus format (`node_exporter` for host metrics, `blackbox_exporter` for probing endpoints, per-database exporters).

## Storage

Prometheus stores data in a local **TSDB**: samples are written to a write-ahead log and compacted into immutable 2-hour **blocks** on disk, each with its own index. Retention is time-based (default 15 days). It's designed to run on one node with local SSD; there's no clustering in core Prometheus.

For long retention, high availability, or a global view across many Prometheus servers, you add **Thanos**, **Cortex**, or **Grafana Mimir** — they take Prometheus's data (via remote write, or by reading its blocks), store it in object storage, deduplicate across HA pairs, and serve unified queries. This is how you keep a year of metrics without a giant local disk.

## PromQL

The query language operates on two things:

- **Instant vector** — one sample per series at a single instant: `http_requests_total`.
- **Range vector** — a window of samples per series: `http_requests_total[5m]`.

The most-used functions:

- **`rate(counter[5m])`** — per-second average rate of increase over the window, correctly handling counter resets (restarts). This is how you turn a monotonic counter into "requests per second."
- **`increase(counter[1h])`** — total increase over the window.
- **Aggregation** — `sum(rate(http_requests_total[5m])) by (handler)` — rate per series, then summed per handler.
- **`histogram_quantile(0.99, sum(rate(request_duration_seconds_bucket[5m])) by (le))`** — p99 latency from a histogram, aggregated across all instances. Note you aggregate the *buckets* first, then compute the quantile — you cannot average per-instance p99s.

Two framings for *what* to graph:

- **RED** for request-driven services — **R**ate, **E**rrors, **D**uration, per endpoint.
- **USE** for resources — **U**tilization, **S**aturation, **E**rrors, per resource (CPU, disk, connection pool).

## Rules

- **Recording rules** — precompute an expensive query on a schedule and store the result as a new series, so dashboards and alerts read the cheap precomputed version.
- **Alerting rules** — an expression that, when it returns results for a duration (`for: 5m`, to avoid flapping), fires an alert.

Alerts go to **Alertmanager**, a separate component that:

- **Groups** related alerts into one notification (all pods of a service down → one page, not 40).
- **Inhibits** — suppress alerts that are downstream of a firing higher-level alert (don't page for "high latency" when "service down" is already firing).
- **Silences** — mute alerts during known maintenance.
- **Routes** to receivers (PagerDuty, Slack, email) by label.

Alert on **symptoms users feel** (error rate, latency), not causes (CPU high — irrelevant if users are fine), and prefer **burn-rate** alerts tied to an [SLO](/citadel/interview/observability) — page when the error budget is being consumed fast enough to run out.

## Grafana

Grafana is the visualization layer — not tied to Prometheus, it queries many data sources (Prometheus, Loki for logs, Tempo for traces, SQL databases, cloud monitoring). You build **dashboards** of panels, each a query rendered as a graph, stat, gauge, heatmap, or table. **Template variables** (`$service`, `$env`) turn one dashboard into many via dropdowns. Dashboards are JSON, so they can live in git and deploy with the service.

## Cardinality — the recurring failure

Each time series costs memory (Prometheus keeps recent series' metadata and chunks in RAM) and disk. **Series count is the product of label-value counts.** Put a bounded label (endpoint, status class, region) and it's fine. Put an **unbounded** one — a user ID, a request ID, an email, a full URL with query params, a raw error message — and one metric explodes into millions of series, Prometheus's memory blows up, and it OOMs.

Rules: labels must be low-cardinality and finite; never put identifiers or free text in labels; put high-cardinality context in [logs and traces](/citadel/interview/observability) instead; watch `prometheus_tsdb_head_series` and `scrape_samples_scraped`; and use `metric_relabel_configs` to drop noisy labels or series at scrape time.

## The one idea to keep

Prometheus scrapes numeric metrics from your services on a timer and stores each unique metric-plus-label combination as its own time series; PromQL turns counters into rates (`rate(x[5m])`) and histograms into aggregatable quantiles (`histogram_quantile`). Grafana draws it, Alertmanager groups and routes the pages. The one thing that reliably breaks a Prometheus is cardinality — a single unbounded label (user ID, request ID, raw URL) multiplies your series into the millions — so keep labels finite and push the high-cardinality detail into logs and traces.
