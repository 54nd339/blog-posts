---
title: Content Delivery Networks - Edge Caching and DNS Routing
description: A CDN puts copies of your content in hundreds of cities and uses DNS to send each user to the nearest one. How the request is steered, the cache hit and miss paths, push versus pull, and what a CDN does and doesn't help with.
date: 2024-12-07
draft: false
slug: /interview/cdn
tags:
  - System Design
  - Interview Prep
  - Networking
---

A user in Bengaluru requests a logo from a server in New York. The bytes are small; the problem is the round trip — thousands of kilometres and a dozen network hops, paid on every request, by every user, against one origin server.

A **content delivery network** fixes this by keeping cached copies of your content on **edge servers** in many locations, and using DNS to route each user to a nearby one. This covers how that routing decision is made, what happens on a cache hit versus a miss, the two models for getting content onto the edge, and the kinds of content it works for.

## Steering the request to an edge

The routing happens during DNS resolution, before any content is fetched.

A site typically points its asset hostname — say `images.example.com` — at the CDN with a `CNAME` record aliasing it to a CDN-owned domain. When a resolver looks up the asset hostname, it is redirected to the CDN's own authoritative DNS servers, and those do the real work: instead of one fixed IP, they return the address of an edge server chosen for this user, by

- **location** — inferred from the resolver's IP, or handled structurally with **anycast**, where many edge servers announce the same IP and the network delivers the packet to the topologically nearest one;
- **load and health** — current traffic and monitoring status of candidate edges;
- **network latency** — which edge currently has the best path to the user's region.

These records carry a **low TTL**, so clients re-resolve often and the CDN can move a user to a different edge as conditions change.

## Hit and miss

The browser then makes its HTTPS request straight to the chosen edge server.

- **Cache hit** — the edge already has the object (someone nearby requested it recently, or it was pre-loaded). It serves it directly. This is the fast path: a nearby server returning bytes it already holds.
- **Cache miss** — the edge fetches from the **origin**, optionally through a regional "shield" cache that concentrates origin traffic and protects it. The edge stores the response according to its `Cache-Control` / TTL rules, then serves it. The next request for that object in the region is a hit.

## Push versus pull

Two models for populating the edge:

- **Pull** (the common default) — the edge fetches an object from the origin the first time it is requested in that region. Simple to run, and only requested content uses edge storage. Cost: the first user per region per object pays the full origin round trip — a per-object cold start.
- **Push** — the origin uploads objects to the edges ahead of demand, used for big launches or known-critical assets. The first user gets a hit, but you now manage what to push, where, and when, and you pay to store content that may never be requested.

## What a CDN is for

CDNs are built for **static assets** that are identical across users: images, video, CSS, JavaScript, fonts, downloads. Personalised or fast-changing responses are harder, and modern CDNs address them differently — running logic at the edge (Cloudflare Workers, Lambda@Edge), caching shared fragments of otherwise-dynamic pages, or **dynamic site acceleration**: optimising the edge-to-origin connection rather than caching the response.

The side benefits matter too. Offloading origin traffic cuts bandwidth cost and absorbs spikes; distribution across many edges adds redundancy; and because all traffic already passes through the edge, that is a natural place for DDoS filtering and a web application firewall.

## The origin still owns correctness

A CDN is a read-through [cache](/citadel/interview/caching) for your content with a global [routing](/citadel/interview/internet-routing) layer in front of it, and it inherits the usual cache question: how do stale objects get replaced? That is what cache-control headers, versioned asset URLs, and purge APIs are for. The edge makes a correct origin fast and global; it cannot make an incorrect origin right. For the routing tier in front of the edge, see [load balancing](/citadel/interview/load-balancing) and [proxies](/citadel/interview/proxies).
