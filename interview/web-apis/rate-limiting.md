---
title: Rate Limiting Algorithms - Token Bucket to Sliding Window
description: A rate limiter decides whether to admit a request based on how many arrived recently. The five standard algorithms differ in how they define "recently" - a hard window, a rolling log, or a leaky bucket - and each trades accuracy against memory and against burst tolerance.
date: 2025-07-08
draft: false
slug: /interview/rate-limiting
tags:
  - System Design
  - Interview Prep
  - Networking
---

An API endpoint can serve 1,000 requests per second before it falls over. One misbehaving client, or one aggressive scraper, can send that many alone. A rate limiter is the component that says "you've had enough, come back later" — returning `429 Too Many Requests` instead of letting the backend drown.

The job sounds trivial: count requests, reject past a threshold. The interesting part is *what you count over*. A fixed one-second window, a rolling window, a bucket that drains at a steady rate — each gives a different answer at the boundaries, and each costs a different amount of memory per client. This post is the five standard algorithms and when to pick which.

## Fixed window counter

Divide time into fixed buckets — say, one per minute. Keep a counter per client per bucket. Increment on each request; reject when the counter exceeds the limit; the counter resets when the bucket rolls over.

```python
def allow(client_id, limit=100, window=60):
    bucket = int(time.time()) // window
    key = f"{client_id}:{bucket}"
    count = redis.incr(key)
    if count == 1:
        redis.expire(key, window)
    return count <= limit
```

One counter per client, trivial to implement, `INCR` is atomic. The flaw is the **boundary burst**: a client can send `limit` requests in the last second of one window and `limit` more in the first second of the next — `2·limit` requests in a two-second span, twice the rate you meant to allow. If bursts on the minute boundary matter, this isn't tight enough.

## Sliding window log

Store a timestamp for every request in a sorted set per client. On a new request, drop all timestamps older than `now - window`, then count what's left. Admit if the count is under the limit.

```python
def allow(client_id, limit=100, window=60):
    now = time.time()
    key = f"log:{client_id}"
    redis.zremrangebyscore(key, 0, now - window)   # evict old
    count = redis.zcard(key)
    if count < limit:
        redis.zadd(key, {str(uuid4()): now})
        redis.expire(key, window)
        return True
    return False
```

This is **exact** — the rate over any trailing `window` is never exceeded, no boundary artefact. The cost is memory: one entry per request in the window, so a client at 10,000 req/min holds 10,000 timestamps. For high limits that's expensive, and the eviction scan adds work per request.

## Sliding window counter

The practical compromise. Keep a counter per fixed window (like the first algorithm) but *estimate* the rolling count by weighting the previous window by how much of it still overlaps:

$$\text{estimate} = C_{\text{current}} + C_{\text{previous}} \cdot \frac{\text{window} - \text{elapsed in current}}{\text{window}}$$

If you're 25% into the current minute, count all of this minute's requests plus 75% of last minute's. Admit if the estimate is under the limit.

Two counters per client instead of thousands of timestamps, and it smooths the boundary burst down to a small over- or under-count (it assumes the previous window's requests were uniformly spread, which they weren't). Cloudflare's edge rate limiter uses this shape. It's the default choice when the log is too heavy and the fixed window is too loose.

## Token bucket

Model the client as a bucket that holds up to `B` tokens and refills at `R` tokens per second. Each request removes one token; if the bucket is empty, reject. You don't run a timer — compute the refill lazily from the elapsed time since the last request.

```python
def allow(client_id, rate=10, burst=20):
    key = f"tb:{client_id}"
    now = time.time()
    tokens, last = redis.hmget(key, "tokens", "last")
    tokens = float(tokens) if tokens else burst
    last = float(last) if last else now
    tokens = min(burst, tokens + (now - last) * rate)   # lazy refill
    if tokens >= 1:
        redis.hset(key, mapping={"tokens": tokens - 1, "last": now})
        return True
    redis.hset(key, mapping={"tokens": tokens, "last": now})
    return False
```

Token bucket allows **controlled bursts**: a client that's been quiet accumulates up to `B` tokens and can spend them all at once, then is throttled to the steady `R`. That matches how real clients behave — idle, then a flurry — which is why it's the most common choice for public APIs (Stripe, AWS, GitHub all use a variant). Two numbers of state per client. The knobs are `R` (sustained rate) and `B` (how big a burst you tolerate).

## Leaky bucket

The same bucket, but requests *queue* in it and drain at a fixed rate `R`; the bucket has a maximum depth. Overflow is rejected. Where token bucket lets bursts through and refills the allowance, leaky bucket **smooths output** — downstream sees a perfectly even stream regardless of how bursty the input was.

Use it when the thing you're protecting cannot absorb bursts at all (a legacy system, a third-party API with its own hard limit, a physical device). The cost is added latency — requests wait in the queue — and the queue itself is state. Most web APIs prefer token bucket precisely because they *want* to serve a burst immediately if capacity exists.

## Choosing, and where it runs

| Algorithm | State per client | Boundary burst | Allows bursts | Note |
|---|---|---|---|---|
| Fixed window | 1 counter | Yes (2×) | — | Simplest |
| Sliding log | N timestamps | No (exact) | No | Accurate, memory-heavy |
| Sliding counter | 2 counters | Minor | No | Good default |
| Token bucket | 2 values | No | Yes, up to `B` | API default |
| Leaky bucket | queue | No | No (smooths) | Protects fragile downstream |

Two more decisions independent of the algorithm:

- **Where it lives** — client SDK (polite, easily bypassed), API gateway or reverse proxy (the usual spot), a sidecar, or inside the service. Earlier is cheaper; later sees more context (per-user quotas, per-endpoint cost).
- **What key you limit on** — IP, API key, user ID, or a tuple. IP alone punishes users behind a shared NAT and is trivial to rotate; API key is the honest unit for a paid API.

Making any of these correct *across a fleet* — many gateway nodes sharing one client's counter — is its own problem, covered in [designing a rate limiter](/citadel/system-design/rate-limiter). The short version: a central store (Redis with a Lua script for atomicity) is simplest; per-node local limits with periodic sync trade accuracy for latency.

Whatever you choose, tell the client: return `429`, a `Retry-After` header, and `X-RateLimit-Remaining` so a well-behaved client can back off instead of hammering.

## The one idea to keep

Every rate limiter counts requests over "recently"; the algorithms are just different definitions of recently. Fixed window is one counter and a boundary bug. Sliding log is exact and expensive. Sliding counter is the cheap approximation of the log. Token bucket — two numbers, lazy refill, burst up to the bucket size — is the default for public APIs because it matches how clients actually send traffic. Leaky bucket is for when the thing downstream genuinely cannot take a burst.
