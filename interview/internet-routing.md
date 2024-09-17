---
title: DNS Routing Policies - Steering Users to the Right Server
description: When a service runs in many places, the DNS answer to a name lookup is a decision. The six standard policies — simple, failover, geolocation, latency, multivalue, and weighted — and what each one optimizes for.
date: 2024-09-17
draft: false
slug: /interview/internet-routing
tags:
  - System Design
  - Interview Prep
  - Networking
---

A global service has servers in many regions, so "what is the IP for `example.com`?" has more than one right answer. **DNS routing policies** are the rules that pick which IP (or IPs) the resolver returns for a given query — a decision made before any packet reaches your infrastructure, aimed at some mix of performance, availability, localisation, and load.

Here are the six policies cloud DNS services offer, and what each is for.

![Six panels. Simple: DNS sends 100% of users to one app. Failover: DNS sends users to the primary app, switching to a failover app on health-check failure. Geolocation: users are routed by location — Virginia users to a default region, Mumbai users to Mumbai. Latency: users are served from the least-latency region. Multivalue: DNS returns several healthy IPs and the client picks one. Weighted: DNS splits traffic 60/40 across two apps.](../images/internet-trp.png "The six DNS routing policies. Geolocation routes by where the user is; latency routes by which region is fastest for them; weighted and multivalue split traffic without either.")

## Simple

One record, one endpoint. Every query gets the same IP, with no health check or logic. Trivial to set up; no failover, no latency optimisation, no geo targeting. Right for a single-server app.

## Failover

A primary endpoint with a **health check**, and a standby. While the primary's checks pass, its IP is returned; when they fail for long enough, the DNS service starts returning the secondary's IP. Built for high availability and [disaster recovery](/citadel/interview/cloud-disaster-recovery). The standby often sits idle and still costs money, and the switch is only as fast as DNS caching allows — low TTLs help.

## Geolocation

Map geographic regions to endpoints: India → Mumbai, Europe → Frankfurt, everywhere else → a default. Location is inferred from the resolver's IP. Use it for localised content (language, currency), for data-residency compliance (keep EU users' data in the EU), and incidentally for latency. The catch: geolocation databases are approximate, and a user on a VPN or a corporate resolver in another country gets routed by where their resolver appears to be.

## Latency

Route to the endpoint with the lowest *measured* network latency to the user, not the one that is geographically nearest — the two often differ. The DNS provider maintains measurement points across the internet and continuously samples latency between regions, then answers each query with the region that currently tests fastest for the user's area. Good for latency-critical apps (gaming, real-time). Its quality is entirely the quality of the provider's latency data.

## Multivalue answer

Return several healthy IPs at once (up to eight), each optionally health-checked so unhealthy ones are dropped from the answer. The client picks one, usually at random. This is crude client-side load spreading and a resilience boost — **not** a substitute for a real [load balancer](/citadel/interview/load-balancing): no distribution algorithm, no session affinity, and evenness depends entirely on client behaviour.

## Weighted

Split traffic by assigned proportions — 80% to version A, 20% to version B — with the DNS service returning IPs in that ratio over time. This is the DNS-level mechanism behind [canary and blue-green rollouts](/citadel/interview/deployments) and A/B tests, and it also lets you send more traffic to a more powerful server. It needs the weights and each endpoint's health watched as you shift them.

## They compose

Real global architectures layer these. A common stack: **geolocation** to send a user to their regional cluster, then **latency** or **weighted** routing plus an actual load balancer to distribute within it. DNS routing gets the request to the right region; a load balancer takes it from there.

## DNS decides the region, not the request

Every policy here works by controlling a name-to-IP answer, which means it is coarse and slow to change — bounded by TTLs and resolver caching, and blind to anything finer than "which endpoint". It is the right tool for regional steering and failover; it is the wrong tool for per-request balancing, health-aware retries, or session stickiness, which all belong to the [load balancer](/citadel/interview/load-balancing) the DNS answer points at.
