---
title: Cloud Native - Containers, Orchestration, and Microservices
description: Cloud native is how an application is built and run, not where it is hosted. The path from virtualization to Kubernetes, the pillars of a cloud-native stack, and why microservices fit — along with the distributed-systems tax they charge.
date: 2024-10-10
draft: false
slug: /interview/cloud-native
tags:
  - System Design
  - Interview Prep
  - Cloud Native
---

Lifting a monolith onto a cloud provider's VMs does not make it cloud native. The term describes *how* an application is designed, packaged, and operated — to be scaled, replaced, and updated continuously by automation — not the address it runs at.

This traces the infrastructure evolution that made the approach possible, lays out the pillars of a cloud-native stack, and looks at microservices: why they pair naturally with cloud native, and the operational cost they add.

## How the ground shifted

Each step raised the level of abstraction and handed more operational work to a platform.

- **2001 — virtualization.** Hypervisors (VMware) let many OS instances share one physical server, ending the one-app-per-box era.
- **2006 — IaaS.** AWS EC2 and S3 made rented compute and storage available on demand, by the hour.
- **2009 — PaaS.** Heroku let developers push code and skip the OS and servers entirely.
- **2013 — containers.** Docker made a lightweight package of an app plus its dependencies the standard unit, virtualizing the OS instead of the hardware.
- **2015 — the CNCF.** The Cloud Native Computing Foundation formed around Kubernetes, which became the default way to schedule and manage containers at scale.

## The pillars of a cloud-native stack

- **Application design** — [twelve-factor](/citadel/interview/12-factor-app) practices: stateless processes, config in the environment, disposability, so instances are interchangeable.
- **Containers** — Docker images as the deploy artifact, identical from laptop to production.
- **Orchestration** — Kubernetes automating placement, scaling, restarts, and rollout of those containers.
- **Infrastructure as code** — Terraform or CloudFormation, so environments are provisioned reproducibly and under version control.
- **Observability** — metrics, structured logs, and distributed tracing, because a request now crosses many services.
- **Serverless (FaaS)** — an option for event-driven pieces that abstracts the runtime away entirely.

The anti-patterns are the inverse of the pillars: a monolith that can only scale as a whole, mutable servers patched in place, one shared database every service reaches into, bloated images that deploy slowly, and manual deploys with no [pipeline](/citadel/interview/ci-cd).

## Microservices, and why they fit

A **microservices architecture** structures an application as small, independently deployable services, each owning one business capability and one team. The fit with cloud native is mechanical: each service is a container, scaled on its own curve, scheduled by Kubernetes, shipped by its own pipeline.

What you gain:

- **Independent scaling** — scale the checkout service without scaling the catalogue.
- **Independent deploys** — small teams release on their own cadence.
- **Technology choice per service** — the right language and datastore for each job.
- **Fault isolation** — one service failing degrades a feature instead of the whole app.

What you pay:

- **Operational load** — dozens of deployables to monitor, log, and trace instead of one.
- **The network is now in your data model** — latency, partial failure, and no free distributed transactions; consistency becomes eventual and explicit.
- **Testing** — an end-to-end path spans services that deploy separately.

## Making microservices work

The practices that keep the cost bounded:

- **One service, one database.** Shared storage recreates the monolith's coupling with none of its simplicity. Each service owns its data and exposes it only through its API.
- **Design for failure.** Circuit breakers, retries with backoff, and bulkheads so a slow dependency does not cascade.
- **Keep services stateless** and put session state in a [shared store](/citadel/interview/session-cookies).
- **Lightweight communication** — [REST](/citadel/interview/rest-api) or [gRPC](/citadel/interview/grpc) for synchronous calls, a [message broker](/citadel/interview/message-queue) for asynchronous events.
- **Service discovery** — instances come and go, so services find each other through a registry (Consul, Eureka, or Kubernetes' built-in DNS) rather than hard-coded addresses.
- **Domain-driven boundaries** — draw service edges along business domains so they stay cohesive and loosely coupled.
- **Choose an interaction style** — central [orchestration](/citadel/interview/orchestration) versus event-driven choreography, deliberately, per workflow.

A production deployment then grows a supporting cast: an **API gateway** as the single entry point handling routing, auth, and rate limiting; a **service registry**; an **authorization server** (OAuth 2.0, Keycloak) covered under [identity management](/citadel/interview/identity-management); a database per service; a [distributed cache](/citadel/interview/caching); a [message broker](/citadel/interview/message-queue) for async flows; and metrics plus log aggregation for observability, with [CDNs](/citadel/interview/cdn) and [load balancers](/citadel/interview/load-balancing) out front.

## It is a threshold decision, not a default

Cloud native's payoff — independent scaling, frequent safe deploys, self-healing — is real, and so is its cost: you are now running a distributed system, with the debugging, consistency, and operational surface that implies. Below a certain team size and rate of change, a well-built monolith on containers captures most of the benefit for a fraction of the cost. The [architecture styles post](/citadel/interview/software-architecture) lays out that trade in more detail.
