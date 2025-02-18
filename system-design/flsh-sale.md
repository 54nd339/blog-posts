---
title: Designing for a Flash Sale - Surviving Extreme Concurrency
description: "A limited stock, a huge discount, a short window: thousands of buyers hit the Buy button in the same second, all racing for the same rows. The principles — less is more, go async, isolate — and the architecture that keeps it stable and prevents overselling."
date: 2025-02-18
draft: false
slug: /system-design/flsh-sale
tags:
  - System Design
  - Interview Prep
  - Scalability
---

A flash sale sells a small, fixed stock at a steep discount for a short window. The demand curve is a spike: thousands or millions of users click Buy in the same instant, all contending for the same inventory rows. The failure modes are specific — database overload, payment surge, bots hoarding stock, and worst of all overselling: telling a customer they won, then cancelling.

## The design principles

**Less is more.** Strip the sale page to essentials — few elements to render, few database queries to build it, few HTTP requests, few downstream services in the purchase path.

**Shorten the critical path.** The sequence a user must complete to secure a deal should be minimal, even if that means merging or streamlining services just for the sale.

**Go asynchronous.** Put a [message queue](/citadel/interview/message-queue) (Kafka, RabbitMQ) on the Buy action. The click is acknowledged the moment it's queued; slower work — order creation, payment, notifications — happens behind it. This decouples the fast user interaction from the sustainable processing rate.

**Isolate resources.** Serve static content (images, CSS, JS) from a [CDN](/citadel/interview/cdn), separate from dynamic inventory logic. Where possible, isolate the flash-sale processes, servers, and databases entirely, so the spike can't degrade the rest of the site.

**Prevent overselling.** Decrement stock atomically and in real time. This is the hardest part, because it's the one point where all that concurrency converges on a single number.

**Keep the UX honest under load.** Communicate clearly — "you're in a queue", "checking inventory" — and for the most extreme demand, use a virtual waiting room or a lottery rather than letting everyone hammer the backend at once.

## The architecture

**Traffic management.** A WAF and anti-bot system filter malicious traffic that would otherwise consume inventory unfairly. An API gateway and load balancers enforce per-user and per-IP [rate limits](/citadel/interview/load-balancing) and spread traffic across backends.

**A lightweight "claim" phase.** The first Buy click doesn't create a full order. It's a fast, minimal request that registers intent or reserves a temporary token, and it goes straight onto a high-throughput queue. The queue absorbs the burst; workers drain it at a rate the backend can sustain.

**Asynchronous order processing.**
- **Inventory service** — workers consume from the queue and attempt an atomic, fast stock decrement. An in-memory store like Redis holds the live count for speed; distributed locks or lock-free algorithms guard it, though locks at this scale are genuinely hard (see [race conditions](/citadel/interview/race-condition)).
- **Order service** — creates the real order only after inventory is reserved.
- **Payment service** — runs after reservation, asynchronously, since external PSPs are slow.

**Database tier.** Redis for the live inventory count; the persistent database is the source of truth, pre-loaded into Redis before the sale and reconciled after. Product detail data is read-mostly — cache it hard and serve from [read replicas](/citadel/interview/read-replica-pattern).

**Virtual waiting room.** When interested users vastly outnumber stock or capacity, queue arrivals at the door and admit a controlled number into the purchase flow at a time, smoothing the spike before it reaches the matching-critical services.

**Pre-compute and cache everything** that can be: the sale page content, query results, static assets on the CDN.

## The takeaway

A flash sale is [scalability](/citadel/interview/scalability) under a stopwatch. The moves are the standard ones — cache, queue, isolate, scale out — but compressed into a few minutes and pointed at a single scarce resource. The design goal is to turn a synchronous stampede into an asynchronous queue that the backend drains at its own pace, while the one number that must stay correct — remaining stock — is decremented atomically in memory.
