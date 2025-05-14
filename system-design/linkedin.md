---
title: How LinkedIn Adopted GraphQL - A Query Registry Instead of a Gateway
description: LinkedIn moved its API layer to GraphQL for thousands of engineers without deploying a central GraphQL gateway. Instead, queries are registered in a catalogue and carry routing metadata that a traffic tier uses to reach the right backend cluster.
date: 2025-05-14
draft: false
slug: /system-design/linkedin
tags:
  - System Design
  - Case Study
  - APIs
---

A LinkedIn page pulls from many sources at once — your profile, your connections' activity, relevant jobs, company updates. Served over REST, that means either **over-fetching** (an endpoint returns more than the view needs) or **under-fetching** (the client makes several calls to assemble one screen). GraphQL fixes both by letting the client ask for exactly the fields it wants in one request. The interesting part of LinkedIn's adoption is what they *didn't* build: a central GraphQL gateway.

## The workflow

Moving to GraphQL, in LinkedIn's words, "changed the development workflow for thousands of engineers." The workflow has three stages:

1. **Edit and test.** A client developer writes a query for their feature's exact data needs and tests it against the backend services until it fetches the right data efficiently.
2. **Register.** The developer commits the query, which is published to a central **query registry** — a versioned catalogue of approved queries. Production clients run registered queries only, not arbitrary ad-hoc ones.
3. **Use in production.** The registered query ships with the client code. It carries **routing metadata** that LinkedIn's traffic-routing tier uses to send each part of the query to the backend cluster that resolves that slice of the schema. Registered queries are cached at service runtime. A request might hit an identity service for member data, then an organization service for company details, stitched together through the GraphQL layer.

## No central gateway

Most GraphQL deployments put one gateway in front of every service. LinkedIn deliberately did not, for two reasons:

1. **An extra network hop.** Every request would pass through the gateway, adding latency.
2. **A single point of failure.** One gateway down would take the whole API layer with it.

The registry-plus-routing-metadata approach is federated: queries are routed straight to the service clusters that can answer them, with no shared chokepoint. The cost is that routing has to be resolved from metadata rather than discovered dynamically at a gateway — which is exactly why the registration step exists.

## Open-source track record

LinkedIn has a habit of building infrastructure for its own scale and releasing it: **Apache Kafka**, the event-streaming log now under half the world's data pipelines; **Apache Samza**, a stream-processing framework; **Apache Pinot**, a real-time distributed OLAP store for low-latency analytics.

## The takeaway

The reusable decision here is refusing the obvious central component. A GraphQL gateway is the default pattern, and for many teams it's right; at LinkedIn's scale the latency and availability cost outweighed the convenience, so they pushed the routing knowledge into registered queries instead. See [GraphQL](/citadel/interview/graphql) for the query language itself and [REST APIs](/citadel/interview/rest-api) for the over/under-fetching problem it addresses.
