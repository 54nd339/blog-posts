---
title: Software Architecture Styles - A Catalogue
description: A named architectural style constrains design decisions to produce specific qualities. A tour of the common ones grouped by theme — layered, component-based, service-oriented, distributed, domain-driven, event-driven, and data-centric — with what each optimizes for and what it costs.
date: 2024-08-16
draft: false
slug: /interview/software-architecture
tags:
  - System Design
  - Interview Prep
  - Architecture
---

An **architectural style** is a named set of design decisions that constrains how a system is structured, in exchange for specific qualities — scalability, testability, evolvability, auditability. Picking one is a trade: you accept the constraint ("dependencies only point inward", "services own their data") because the constraint is what buys the quality. A style also gives a team a shared vocabulary and a proven answer to a recurring shape of problem, so nobody re-derives the layered web app from first principles.

This is a catalogue of the common styles, grouped by theme, with the trade each one makes and where it shows up in practice. Most real systems combine several.

![A wheel diagram of software architecture styles radiating from a centre, grouped into Layered (Client/Onion, n-tier), Component-Based (Object-Oriented, Microkernel, Plug-in), Service-Oriented (SOA, Broker, Microservices, Serverless), Distributed System (Space-Based, Peer-to-Peer), Domain-Driven (Hexagonal/Ports and Adapters, DDD), Event-Driven (Publish-Subscribe), Separation of Concerns (MVP, MVC), Interpreter, Concurrency (Orchestration, Choreography, Primary-Secondary, Pipeline/Pipe-Filter), and Data-Centric (CQRS, Event Sourcing, Lambda, Kappa).](../images/software-architectures.png "The common architectural styles, grouped by theme. Most real systems combine several. Source: ByteByteGo.")

## Layered

- **N-tier** — horizontal layers with fixed responsibilities: presentation, application/business logic, data access. Requests flow top to bottom, and a layer only ever calls the one below it. This is the default shape of a server-rendered web app or a classic enterprise system. You get a clean separation of concerns and per-layer testing; you pay pass-through overhead (a request touches every layer even when it needs nothing from most of them) and you risk **leaky abstractions** — a data-access detail like an ORM's lazy-loading behaviour bleeding up into business logic because the boundary wasn't held.
- **Clean / Onion / Hexagonal** — concentric layers where dependencies point *inward* only: entities at the core, then use cases, then interface adapters, then frameworks and drivers. The single rule — outer knows inner, never the reverse — is the whole style. The business logic has no import of the web framework or the database driver, so you can swap Postgres for DynamoDB, or REST for a CLI, by rewriting only the outermost ring. The core is trivially unit-testable because it has no I/O to mock. The cost is indirection and boilerplate: interfaces and adapter classes that a three-endpoint CRUD app doesn't need.

## Component-based

Decompose the design into independent, interchangeable, reusable units.

- **Object-oriented design** is a paradigm rather than a top-level style, but its principles — **encapsulation** (data bundled with the methods that touch it), **inheritance**, **polymorphism** — are the micro-level design language for how a single component is built inside almost any of the styles below. See [OOP](/citadel/interview/oop).
- **Microkernel (plug-in)** — a minimal core provides base services and a set of extension points; every feature beyond the minimum is a plug-in module. VS Code, Eclipse, and browsers with extensions are built this way, as are many IDEs and heavily-customised enterprise products. A faulty plug-in can be isolated so it doesn't crash the core, and third parties can extend the product without a fork. The hard part is designing a plug-in API stable enough that a five-year-old extension still loads — every core change is now a public contract change.

## Service-oriented

Structure the application as distinct services that each provide a business capability and talk over a network.

- **SOA** — coarse-grained business services communicating through an **enterprise service bus**, middleware that routes messages, transforms formats, and mediates protocols. Reusable and interoperable across heterogeneous systems; the ESB concentrates logic ("smart pipes") and becomes a bottleneck, a single point of failure, and a governance chokepoint.
- **Broker** — a central middleware component coordinates otherwise-decoupled producers and consumers, handling routing and protocol mediation. Loose coupling and independent evolution; the broker must itself be highly available and scalable.
- **[Microservices](/citadel/interview/cloud-native)** — many small, independently deployable services, each owning one capability *and its own database* (the database-per-service rule that prevents back-door coupling through a shared schema), talking over REST/gRPC or asynchronous messaging, fronted by an API gateway with service discovery. The reaction to SOA's smart pipes: "dumb pipes, smart endpoints." You get independent scaling, per-service technology choices, and fault isolation; you take on the full operational and distributed-systems cost — hundreds of deployables, no distributed transactions, eventual consistency, and end-to-end tracing as a requirement rather than a nicety. Netflix and Amazon run hundreds to thousands of them.
- **Serverless (FaaS)** — small event-triggered functions run in provider-managed stateless containers (AWS Lambda, Cloud Functions). Auto-scales from zero, billed per invocation, no servers to patch; you pay in **cold starts** (tens to hundreds of milliseconds to spin a fresh container for the first request after idle), a hard execution-time cap (15 minutes on Lambda), and forced statelessness — any state lives in an external store. See [serverless databases](/citadel/interview/serverless-db) for the data-tier version.

## Distributed

- **Space-based** — an in-memory data grid (GigaSpaces, Hazelcast, Coherence) with processing units that each hold application logic *and* co-located cached data; units are added and removed dynamically while data is replicated and partitioned across them. There is no central database in the request path, so the database stops being the scaling ceiling. It suits spiky, high-volume load — flash sales, online betting, trading — and it is genuinely hard to get distributed state, replication, and consistency right.
- **Peer-to-peer** — decentralised, all nodes equal, communicating directly with no central server; peers find each other and resources through a distributed hash table (Kademlia in BitTorrent's DHT). Resilient and censorship-resistant with no single point of failure; discovery, security against malicious peers, and consistency are all harder than the centralised version. BitTorrent, blockchains, some distributed databases.

## Domain-driven

Align the software with the business domain it serves.

- **DDD** — build a shared **ubiquitous language** with domain experts and use it everywhere, in conversation and in code. Partition the domain into **bounded contexts** — regions where a term has one precise meaning ("account" in billing is not "account" in auth) — and those boundaries become natural service boundaries. Inside a context, the tactical patterns: **entities** (identity that persists through change), **value objects** (defined only by their attributes, immutable), **aggregates** (a cluster of objects with one root that guards the cluster's invariants and is the only thing you load or save as a unit), **repositories** (the collection-like interface for retrieving aggregates), and **domain events** (something happened that other parts of the domain care about).
- **Hexagonal (ports and adapters)** — isolate the core logic behind **ports** (interfaces the core defines for how it wants to interact) and bridge to specific technologies with **adapters** (a Postgres adapter, a Kafka adapter, an HTTP adapter). External tools swap by writing a new adapter; the core never changes and tests against in-memory adapters.

## Event-driven

Flow is determined by **events** — records that something happened. An event **producer** publishes to an event **channel** (Kafka, RabbitMQ) that routes to **consumers** who subscribed to that topic. The **publish-subscribe** pattern is the core: publishers and subscribers know only the topic name, never each other. You get extreme decoupling — add a consumer without touching any producer — and independent scaling of each consumer. You pay in traceability: the end-to-end flow of a business operation is now spread across handlers with no call stack to read, and message ordering and exactly-once processing are non-trivial and broker-dependent. See [orchestration versus choreography](/citadel/interview/orchestration) for the two ways to compose event-driven workflows.

## UI separation of concerns

**MVC**, **MVP** (a passive view delegates every input to a presenter), and **MVVM** (a view-model exposes model data for the view to bind against) all pull UI-independent logic out of the view so it can be unit-tested without a running UI. The cost is indirection and binding boilerplate on simple screens. Detailed in [UI patterns](/citadel/interview/ui-patterns).

## Interpreter

Define a grammar and an interpreter that executes statements written in it: a context holding global state, plus terminal and non-terminal expression types that each know how to evaluate themselves. SQL engines, regular-expression matchers, and business-rule engines are built this way — anywhere the system's real job is to run a little language.

## Concurrency and data-flow patterns

Less about top-level structure, more about flow within or between components:

- **Orchestration vs choreography** — a central coordinator that holds the workflow logic and directs each service, versus services that react to each other's events with no coordinator. Central control is easier to see and to add compensation logic to; it is also a bottleneck and a coupling point. [Their own post](/citadel/interview/orchestration).
- **Primary-secondary (leader/follower)** — one node takes writes and coordinates; followers replicate its log and stand ready to be promoted if it fails. Underpins [read replicas](/citadel/interview/read-replica-pattern) and consensus protocols.
- **Pipe-filter** — data flows through independent **filters** connected by **pipes**, each filter's output the next one's input. Filters are reusable and the stages parallelise; it is poor for interactive systems and for flows that need to double back. ETL jobs, and a compiler's lex → parse → typecheck → codegen pipeline.

## Data-centric

- **Lambda** — a batch layer recomputing views over all history, a speed layer covering the recent window the batch layer hasn't caught up to, and a serving layer merging the two at query time. Robust and handles both historical and real-time questions; the weak point is maintaining the same logic in two codebases (batch and streaming).
- **Kappa** — one stream engine for everything; reprocessing history means replaying the log through the same stream job. Simpler, one codebase; pushing a large historical backfill through a stream processor is the strain point.
- **[Event sourcing](/citadel/interview/event-sourcing)** — store every state change as an immutable event and rebuild current state by replaying them; the event log is the source of truth. Full audit trail and "what was the state on this date" for free; replay cost (mitigated with periodic snapshots) and current-state queries that are indirect compared to reading a row.
- **CQRS** — separate the write model (commands that change state) from the read model (queries), often in different stores each tuned for its job — normalised Postgres for writes, denormalised Elasticsearch for reads. Reads and writes scale independently; you add the complexity of two models and eventual consistency between them.

Lambda and Kappa are covered further in [data pipelines](/citadel/interview/data-pipelining).

## No style is a default

The catalogue is not a menu to pick one item from — real systems layer several: a microservices system where each service is internally hexagonal, the services communicate event-driven, and the two or three services with heavy read/write asymmetry use CQRS. The skill is reading which qualities a system actually needs — is the pain testability, raw scale, team autonomy, auditability? — and composing the styles that deliver those while paying only for the complexity they cost. Every style on this list is somebody's over-engineering and somebody else's essential structure.
