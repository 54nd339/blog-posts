---
title: Bare Metal, VMs, and Containers - The Hosting Spectrum
description: Four ways to put an application on a server, each adding an abstraction layer that trades performance and isolation for flexibility and density. What each layer virtualizes, and why cloud Kubernetes runs containers inside VMs.
date: 2024-07-18
draft: false
slug: /interview/hosting
tags:
  - System Design
  - Interview Prep
  - Containers
---

Every way of running an application on a server is a choice about how many abstraction layers sit between your code and the hardware. Add a layer and you gain flexibility, density, and portability; you pay in performance and resource overhead. Four points on that spectrum cover almost everything in use.

![Four layered stacks over Physical Server and Networking. Bare metal: OS then apps directly. Virtualized: hypervisor, then VMs each with a guest OS and apps. Containerized: a container engine on the host OS, then containers sharing that kernel. Containerized on virtualized: hypervisor, VMs with guest OS, a container engine inside each VM, then containers.](../images/hosting.png "Each column adds an abstraction: a hypervisor virtualizes hardware, a container engine virtualizes the OS, and the hybrid stacks both.")

## Bare metal

The OS and applications run directly on the physical hardware — no virtualization layer. Stack: hardware, OS, apps.

Direct hardware access gives the **highest performance and lowest overhead**, and full control of machine and OS. The costs are flexibility and utilization: running several unrelated applications or different OS environments on one box is awkward, an app that does not use the whole machine wastes the rest, and provisioning is manual.

## Virtualization (VMs)

A **hypervisor** (VMware ESXi, KVM, Hyper-V) sits on the hardware — or on a host OS, for a Type 2 hypervisor — and presents virtual hardware to multiple **virtual machines**. Each VM runs its own full **guest OS**, virtual CPU, memory, disk, and network, then its libraries and app.

- **Strong isolation** — every VM has its own kernel; a compromise or crash stays inside it.
- **OS flexibility** — Linux and Windows VMs on the same host.
- **Heavy** — a full OS per VM means gigabytes of memory and disk and a full boot on start.

This was the first generation of cloud computing.

## Containers

A **container engine** (Docker) runs on the host OS. The app and its libraries are packaged into a **container**, and many containers on a host **share that host's kernel**, isolated by kernel features — Linux **namespaces** for what a process can see, **cgroups** for how much it can use.

- **Lightweight and fast** — no guest OS, so startup is a process launch, not a boot, and provisioning is near-instant.
- **Dense** — many more containers than VMs fit on a host.
- **Portable** — the image carries the dependencies, so it runs the same everywhere.
- **Weaker isolation** — a shared kernel is a shared attack surface; a kernel exploit crosses containers in a way it cannot cross VMs.

## Containers on VMs

Stack both: hardware, hypervisor, VM with a guest OS, a container engine *inside* the VM, then containers.

You get the VM's hardware-level isolation between tenants or environments and the container's agility within each VM. VMs carve out dedicated resource pools for groups of containers, and the hypervisor lets you run, say, a Linux VM on a Windows host and Linux containers inside it — which is exactly how Docker Desktop works on Windows and macOS. Managed Kubernetes in the cloud is this model: the worker nodes running your containers are themselves VMs.

## Comparison

| | Bare metal | VMs | Containers | Containers on VMs |
| --- | --- | --- | --- | --- |
| Virtualizes | nothing | hardware | the OS | hardware + OS |
| OS | one, shared | guest OS per VM | share host kernel | share guest kernel per VM |
| Overhead | lowest | high (full OS each) | low (shared kernel) | medium |
| Startup | OS boot | OS boot | process start | VM boot + process start |
| Isolation | none between apps | strong (kernel) | process-level | strong + process-level |
| Density | n/a | low | high | medium |

## The default is containers, on VMs you did not choose

For most applications the answer is containers — the portability and startup speed pay for themselves — and in the cloud they are already running inside VMs whether you think about it or not, because that is how a provider gives you isolation from other tenants on shared hardware. Bare metal is for the cases where the virtualization tax is unacceptable: latency-floor trading systems, GPU training clusters, databases tuned to specific hardware. The [cloud native](/citadel/interview/cloud-native) and [twelve-factor](/citadel/interview/12-factor-app) posts pick up where the packaging choice ends.
