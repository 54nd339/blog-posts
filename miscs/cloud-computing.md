---
title: Cloud Computing - Roots, Service Models, and Trade-offs
description: "Cloud computing is on-demand access to pooled, metered computing resources over a network. Where it came from - distributed systems, clusters, grids - the five NIST characteristics, the deployment models, the IaaS/PaaS/SaaS stack and what each layer hands you, and the real costs: lock-in, noisy neighbours, and compliance."
date: 2023-07-10
draft: false
slug: /miscs/cloud-computing
tags:
  - Cloud Computing
  - Distributed Systems
---

Stream a film, check email on a phone, edit a shared document — all of it runs on infrastructure you never see and pay for by the minute. That is **cloud computing**: on-demand access to a shared pool of computing resources — servers, storage, applications — delivered over a network and billed by use. This post covers where the idea came from, the standard models for describing it, and the trade-offs that come with not owning the hardware.

## Where the cloud came from

The cloud is an evolved [distributed system](/citadel/miscs/distributed-algorithms) — independent computers cooperating so they look like one. The ideas it inherits:

- **Distributed systems** — resource sharing, concurrency, scalability, fault tolerance, and transparency (hiding the complexity of many machines behind one interface).
- **Cluster computing** — many machines on a fast local network acting as one, built for high availability, load balancing, and parallel processing. Cloud providers run huge clusters as their base layer.
- **Grid computing** — pooling geographically dispersed, heterogeneous resources into a virtual supercomputer for large computational jobs. The cloud's on-demand pooling has roots here.
- **[Mobile computing](/citadel/miscs/mobile-computing)** — resource-constrained devices that needed powerful backends for storage and processing, which drove demand for exactly what the cloud provides.

## The NIST definition: five characteristics

1. **On-demand self-service** — provision resources automatically, without contacting the provider.
2. **Broad network access** — reachable over the network from diverse devices.
3. **Resource pooling** — the provider's resources serve many customers (multi-tenancy), assigned and reassigned by demand; you generally do not know the physical location.
4. **Rapid elasticity** — scale up and down quickly; it feels unlimited.
5. **Measured service** — usage is monitored and reported, which is what makes pay-as-you-go possible.

**Why organisations move to it:** capital expense (buying hardware) becomes operational expense (renting it); elastic scaling for peak loads; provider-run redundancy and [disaster recovery](/citadel/interview/cloud-disaster-recovery); access from anywhere; faster provisioning, so faster time to market.

## Deployment models

- **Private cloud** — infrastructure for one organisation, on- or off-premise, self- or third-party managed. Trades cost for control, security, and customisation.
- **Public cloud** — owned and run by a provider (AWS, Azure, GCP), available to anyone. Cost-effective, scalable, no upfront hardware.
- **Hybrid cloud** — private and public bound together so data and applications can move between them (for example, "cloud bursting" to the public cloud under load).
- **Community cloud** — shared by organisations with common requirements (a regulatory regime, a mission).

## The service stack

Each layer hands you more and asks you to manage less.

- **IaaS (Infrastructure as a Service)** — virtual servers, storage, networking. You manage the OS, applications, and data. *Examples:* Amazon EC2, Azure Virtual Machines, Google Compute Engine.
- **PaaS (Platform as a Service)** — a platform (OS, runtimes, build tools, databases) for building and running applications without touching the infrastructure. *Examples:* Google App Engine, Azure App Service, Heroku.
- **SaaS (Software as a Service)** — finished applications over the network, usually by subscription; the provider runs everything. *Examples:* Gmail, Salesforce, Microsoft 365, Dropbox.

A pizza makes the layers concrete: with IaaS you get an oven and ingredients and make the pizza; with PaaS you get a base and toppings in a stocked kitchen; with SaaS the pizza is delivered cooked. (An even rawer tier, sometimes called RaaS, is leasing bare-metal servers or data-centre space.)

### Inside IaaS

- **Virtualisation** — a **hypervisor** (VMware ESXi, Hyper-V, KVM) runs multiple virtual machines on one physical server, each with its own OS. See [the big data stack](/citadel/big-data/tech-foundation) for virtualisation versus containers.
- **VM provisioning and migration** — creating and configuring VMs with set CPU, RAM, storage, and network; moving them between hosts, sometimes while running (**live migration**), for load balancing, maintenance, or recovery.
- **Placement scheduling** — algorithms choose which physical host a new VM lands on, weighing utilisation, energy, QoS for existing VMs, and reservations for critical workloads.

### Inside PaaS

- **Integrated lifecycle platforms** cover the whole application lifecycle — code, build, deploy, test, host, update — for example Google App Engine (Python, Java, Go, with autoscaling) and Azure App Service.
- **Anchored lifecycle platforms** are built around a core application or data model that developers extend — for example the Salesforce Platform, where custom apps integrate deeply with Salesforce's CRM data and security model.

### Inside SaaS

- **Multi-tenant architecture** — one software instance serves many customers, with data isolated per tenant.
- **Web access**, **centralised updates** (the provider patches everything), **subscription billing**.

## What the cloud runs

Web hosting and [CDNs](/citadel/interview/cdn), big-data analytics, mobile backends, [IoT](/citadel/tech/iot) platforms, backup and disaster recovery, dev/test environments, game infrastructure, and AI/ML platforms. Modern building blocks like [containers and orchestration](/citadel/tech/k8s), [serverless databases](/citadel/interview/serverless-db), and [12-factor](/citadel/interview/12-factor-app) [cloud-native](/citadel/interview/cloud-native) design assume a cloud underneath.

## The trade-offs

- **Vendor lock-in** — deep use of a provider's proprietary services makes leaving expensive.
- **Network dependency** — "the cloud is someone else's computer", and you need a good link to reach it.
- **Performance variability** — in multi-tenant environments a "noisy neighbour" heavy on shared hardware can degrade your performance.
- **Compliance and legal** — data sovereignty (where data physically lives) and regulations like HIPAA and GDPR need deliberate handling.
- **Management complexity** — controlling cost, security, and resources across many services is its own discipline.

### Security concerns

Data breaches, account hijacking (weak credentials, phishing), insecure management APIs, cross-tenant exploits (rare with reputable providers), insider threats, and data-privacy handling. These are consistently the top hesitation for organisations moving workloads.

## Mobile and sensor integration

- **Mobile backend as a Service (MBaaS)** — ready-made authentication, storage, push notifications, and APIs, so mobile apps skip building a backend.
- **Offloading** — pushing heavy work (image processing, ML inference) from the device to the cloud to save battery.
- **Wireless sensor networks** generate large data volumes from many small sensors; the cloud supplies the storage, processing, and scale — behind smart cities, environmental monitoring, remote healthcare, and precision agriculture.

## Key takeaways

- Cloud computing is **on-demand, pooled, elastic, metered** computing over a network — a distributed system descended from clusters and grids.
- **Deployment models** (public, private, hybrid, community) say who owns and shares it; **service models** (IaaS, PaaS, SaaS) say how much you manage.
- The costs are real: lock-in, dependence on connectivity, noisy-neighbour variance, compliance obligations, and the complexity of managing it all.
