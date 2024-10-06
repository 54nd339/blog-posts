---
title: Forward and Reverse Proxies - Two Kinds of Middleman
description: Both sit between client and server relaying traffic; the difference is which side they represent. A forward proxy acts for the client, a reverse proxy acts for the servers, and that determines what each is used for.
date: 2024-10-06
draft: false
slug: /interview/proxies
tags:
  - System Design
  - Interview Prep
  - Networking
---

A **proxy server** is an intermediary: instead of a client talking straight to a server, it talks to the proxy, which relays the request. Both kinds of proxy do that same relaying. What separates them is *whose side they are on* — and that single fact decides what each is good for.

## Forward proxy — acts for the client

Sits between clients on an internal network and the public internet, making requests **on the client's behalf**. The destination server sees the proxy, not the original client. Like asking an office librarian to fetch a book from the public library — the library only meets the librarian.

Uses, all client-side:

- **Anonymity** — hides the client's real IP from the destination.
- **Bypassing restrictions** — a proxy located elsewhere reaches content blocked by geography or by network policy.
- **Content filtering** — a school or company blocks categories of sites for everyone behind the proxy.
- **Shared caching** — popular resources are cached once and served to many internal clients, cutting bandwidth.

## Reverse proxy — acts for the servers

Sits in front of one or more backend servers and takes requests **from internet clients**, forwarding each to a backend and returning the response as if it had served it itself. Like a single reception desk for a large company — visitors only ever deal with the desk, and the internal departments stay hidden.

Uses, all server-side:

- **Shielding backends** — the servers' IPs and characteristics are not exposed to the internet, so direct attacks land on the proxy.
- **[Load balancing](/citadel/interview/load-balancing)** — distribute requests across backend instances. The most common reason to run one.
- **Caching** — serve backend responses (static, sometimes dynamic) directly on repeat requests.
- **[TLS termination](/citadel/interview/https)** — handle HTTPS decryption at the proxy, forward plain HTTP internally, moving the crypto cost off the app servers.
- **Compression, centralised logging, path-based routing** — gzip responses, log all ingress in one place, route `/api/*` and `/static/*` to different backends.

Nginx and HAProxy are the common software; Envoy is the service-mesh variant.

## The difference in one table

| | Forward proxy | Reverse proxy |
| --- | --- | --- |
| Acts for | the client | the server(s) |
| Protects | client identity, internal network | backend identity, infrastructure |
| Client awareness | configured to use it, knows it is there | transparent — client thinks it is the origin |
| Controls | outbound access *to* the internet | inbound access *from* the internet |
| Typical uses | filtering, anonymity, bypass | load balancing, TLS termination, caching |

## In a modern stack

Forward proxies are still standard in enterprises for security and policy enforcement. Reverse proxies are foundational to web and [microservice](/citadel/interview/cloud-native) architecture — usually the first thing an internet request meets, doing traffic management and defence before anything reaches an application server. In a service mesh, a per-service Envoy proxy handles inter-service load balancing, circuit breaking, and observability at every hop. The line between a reverse proxy, a [load balancer](/citadel/interview/load-balancing), and an API gateway is mostly one of emphasis — often they are the same component configured for different jobs.
