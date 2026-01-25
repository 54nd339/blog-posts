---
title: Powering Professionals - A Look Inside LinkedIn's GraphQL Adoption
description: Discover how LinkedIn, the world's largest professional network, leverages GraphQL to manage complex data interactions, improve developer workflows, and scale its API architecture.
date: 2024-07-11
draft: false
slug: /pensieve/linkedin
tags:
  - system-design
  - SDE
---

Hey everyone! LinkedIn, the sprawling digital space for professionals, connects hundreds of millions of members with opportunities, insights, and each other. Imagine the sheer complexity of data involved: user profiles, company pages, job postings, articles, messages, connections – all interconnected. Serving this data efficiently and enabling rapid product development is a massive engineering challenge.

One of the key technologies LinkedIn adopted to tackle this complexity at the API layer is **GraphQL**. Today, we'll explore how LinkedIn integrated this powerful query language, changing how their developers work and how data is served to their numerous applications.

## The Challenge: Data for a Dynamic Network

Traditional REST APIs, while powerful, can sometimes lead to inefficiencies when dealing with highly interconnected data and diverse client needs:

* **Over-fetching:** Clients receive more data than they actually need for a particular view.
* **Under-fetching:** Clients need to make multiple API calls to different endpoints to gather all the necessary information, leading to increased latency and complexity.

For a platform like LinkedIn, where a single page might display information aggregated from various sources (your profile, your connections' activities, relevant job postings, company updates), these challenges can become significant bottlenecks.

## LinkedIn's Solution: Embracing GraphQL

LinkedIn recognized the potential of GraphQL to address these issues. As noted, "Moving to GraphQL was a huge initiative that changed the development workflow for thousands of engineers...". Their approach to adopting GraphQL involved a structured workflow designed for scalability and developer efficiency.

### The LinkedIn GraphQL Workflow

The overall workflow for developing and deploying GraphQL queries at LinkedIn is a multi-stage process:

**Part 1 - Edit and Test a Query:**
* Client-side developers begin by developing a GraphQL query tailored to the specific data needs of their feature or application.
* They then test this query against backend services to ensure it fetches the correct data and performs efficiently.

**Part 2 - Register a Query:**
* Once satisfied, the client-side developer commits the query.
* This query is then published to a central **query registry**.  This registry acts as a catalog of approved and versioned queries.

**Part 3 - Use in Production:**
* The registered query is released along with the client-side application code that will use it.
* Crucially, **routing metadata** is included with each registered query.
* This metadata is utilized by LinkedIn's traffic routing tier to direct incoming requests to the appropriate backend service cluster responsible for resolving that particular part of the GraphQL schema.
* To optimize performance, these registered queries are cached at the service runtime.
* An example of how a query might flow: a request could first hit an identity service to retrieve member information and then proceed to an organization service to fetch company details, all orchestrated via the GraphQL layer.

### Architectural Choice: No Central GraphQL Gateway

Interestingly, LinkedIn made a deliberate decision **not** to deploy a single, monolithic GraphQL gateway that would sit in front of all their services.  They cited two main reasons for this choice:

1.  **Preventing an additional network hop:** A central gateway would introduce another layer that every GraphQL request would have to pass through, potentially adding latency.
2.  **Avoiding a single point of failure:** A centralized gateway could become a critical failure point for their entire API infrastructure.

Instead, their federated approach, where queries are routed to appropriate service clusters, aligns with a more distributed and resilient architecture.

## Beyond GraphQL: LinkedIn's Open Source Impact

LinkedIn's engineering prowess isn't just limited to its internal architecture. The company has also made significant contributions to the open-source world, developing and sharing technologies that are now industry standards. Some notable examples include:

* **Apache Kafka:** A distributed event streaming platform that is fundamental to many real-time data pipelines worldwide.
* **Apache Samza:** A distributed stream processing framework.
* **Apache Pinot:** A real-time distributed OLAP datastore, designed to deliver ultra-low latency analytics, even at massive scale.

These contributions highlight a culture of tackling hard engineering problems and sharing the solutions with the broader community.

## Why This Matters: Lessons from LinkedIn's Approach

LinkedIn's adoption of GraphQL and their overall engineering philosophy offer valuable lessons:

* **Addressing API Complexity:** For applications with rich, interconnected data models and diverse client needs, GraphQL can provide a more efficient data fetching mechanism than traditional REST APIs.
* **Iterative Adoption and Workflow Evolution:** Introducing a new technology like GraphQL often requires changes to developer workflows, including how queries are created, tested, and managed.
* **Strategic Architectural Decisions:** The choice to avoid a central GraphQL gateway in favor of a more distributed routing mechanism reflects a careful consideration of performance and resilience trade-offs.
* **Focus on Developer Experience:** A query registry and tools for testing help streamline the development process for thousands of engineers.

## Key Takeaways

* LinkedIn uses GraphQL to manage the complex data requirements of its professional networking platform, enabling efficient data fetching for various client applications.
* Their GraphQL workflow involves query development, registration in a central registry, and intelligent routing to backend service clusters.
* LinkedIn opted against a single GraphQL gateway to avoid additional latency and a single point of failure.
* The company is also a major contributor to open-source projects, including Kafka, Samza, and Pinot.

LinkedIn's engineering blog is a fantastic resource for anyone interested in learning more about how they tackle challenges at scale.
