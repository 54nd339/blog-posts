---
title: Load Balancing - Distributing Traffic Across Servers
description: A load balancer spreads requests over a pool of servers, drops the unhealthy ones, and lets you add capacity by adding machines. The types by layer, the static and dynamic distribution algorithms, and how it differs from a reverse proxy and an API gateway.
date: 2024-09-30
draft: false
slug: /interview/load-balancing
tags:
  - System Design
  - Interview Prep
  - Networking
---

One server has a request-per-second ceiling and a single point of failure. A **load balancer** removes both: it sits in front of a pool of backend servers, spreads incoming requests across them, stops routing to any that fail a health check, and lets you grow capacity by adding machines to the pool. It is the first thing reached for when an application outgrows one box.

This covers what a load balancer does beyond distribution, the types by where they operate, the algorithms they distribute with, and where they sit relative to a reverse proxy and an API gateway.

## What it does

- **Distribute traffic** — spread requests so no server is the bottleneck.
- **High availability** — continuous **health checks** on each backend; a failing server is pulled from the pool and its traffic rerouted to healthy ones, with no user-visible outage.
- **Horizontal scaling** — add servers to the pool and the balancer starts using them automatically.
- **SSL termination** — decrypt HTTPS at the balancer and forward plain HTTP over the trusted internal network, moving the crypto cost off the application servers.
- **Session stickiness** — for stateful apps, pin a user's requests to the server holding their session, via source-IP hash or a cookie.

## Types

By deployment:

- **Hardware** — dedicated appliances, high throughput, expensive, common in on-prem data centres.
- **Software** — Nginx, HAProxy on ordinary machines; flexible and cheap.
- **Cloud** — managed services (AWS ELB, Google Cloud Load Balancing, Azure Load Balancer); scalable, integrated, low operational overhead.

By layer:

- **Layer 4 (transport)** — routes on IP addresses and TCP/UDP ports without looking at the payload. Very fast.
- **Layer 7 (application)** — inspects HTTP headers, cookies, and URL paths, so it can route `/images/*` to one pool and `/api/*` to another. Slower, smarter.
- **Global server load balancing (GSLB)** — distributes across geographically separate data centres, for disaster recovery, proximity routing, and data-locality compliance. Overlaps with [DNS routing policies](/citadel/interview/internet-routing).

## Algorithms

**Static** — ignore current server state:

- **Round robin** — requests cycle A, B, C, A, … Assumes equal servers and stateless requests.
- **Sticky round robin** — round robin, but once a user lands on a server their session stays there.
- **Weighted round robin** — bigger servers get a higher weight and a proportionally larger share.
- **IP / URL hash** — hash the client IP (or URL) to a server, giving natural stickiness as long as the IP and the pool are stable.

**Dynamic** — react to load:

- **Least connections** — send the next request to the server with the fewest active connections. Handles widely varying request durations well.
- **Least response time** — send to the server currently responding fastest.

## Load balancer, reverse proxy, API gateway

They overlap and are often the same box, but the emphasis differs:

- A **reverse proxy** is a single front door for one or a few servers, focused on TLS, caching, and shielding the origin.
- A **load balancer** is focused on spreading load across *many instances of one service* and health-checking them.
- An **API gateway** routes to *different* services and adds auth, rate limiting, and request transformation.

Large systems stack them: an external load balancer in front of API gateway instances, each using an internal load balancer to reach instances of a specific microservice.

## It is the enabler for everything stateless

A load balancer only works well if any server can handle any request — which is why [stateless processes](/citadel/interview/12-factor-app) and externalised sessions matter so much. Get that right and scaling is "add machines, the balancer finds them"; get it wrong and you are forced into stickiness, which re-creates the single-server failure mode one user at a time. The [scalability](/citadel/interview/scalability) post covers where the load balancer sits in the wider growth path.
