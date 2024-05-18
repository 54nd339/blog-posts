---
title: The Internet of Things - The Four Layers and the Protocols Between Them
description: An IoT system is four layers - devices, a gateway, a cloud platform, applications - connected by protocols chosen for power and bandwidth constraints. What each layer does, why edge computing pushes work back down toward the devices, and where the security has to hold.
date: 2024-05-18
draft: false
slug: /tech/iot
tags:
  - Tools
  - IoT
  - Edge Computing
---

The **Internet of Things** is the idea of giving everyday objects a network connection and letting them exchange data without a person in the loop — a thermostat that reports temperature and accepts a new setpoint, a sensor that streams soil moisture, a machine that flags its own wear before it breaks.

Behind almost any of these is the same four-layer shape: the devices themselves, a gateway that aggregates them locally, a cloud platform that stores and analyses, and applications that people or automations use. The interesting engineering is in the constraints — battery life and flaky networks — which drive the protocol choices and push computation back toward the edge. This post walks the four layers and what connects them.

## Layer 1: things

The devices that touch the physical world. Two kinds of components:

- **Sensors** measure — temperature, humidity, light, pressure, motion, acceleration, orientation, GPS position, heart rate and SpO₂, plus cameras and microphones.
- **Actuators** act — switch a light, move a thermostat, unlock a door, drive a motor, dispense a product.

They communicate over low-power wireless, because many run on a battery that has to last months or years.

## Layer 2: the gateway

Most IoT devices don't talk to the cloud directly. A **gateway** is a local hub for a cluster of them, and it does four jobs:

- **Connectivity management** — hold connections to many local devices using whatever radio each one speaks.
- **Protocol translation** — devices use short-range protocols like Zigbee, Z-Wave, Bluetooth Low Energy, or LoRaWAN; the gateway converts these to IP-based protocols (MQTT, CoAP, HTTPS) for the trip to the cloud.
- **Edge preprocessing** — filter, aggregate, and run simple rules locally before sending anything up. This is *edge computing*, covered below.
- **Security** — authenticate devices, encrypt traffic, and keep the local devices off the public internet.

It often also handles device provisioning, configuration, and firmware updates.

## Layer 3: the cloud platform

Where storage, heavy processing, and fleet management happen:

- **Ingestion** — receive and route large streams of data, usually through a message broker like [Kafka](/citadel/tech/kafka) or a managed equivalent. See [message queues](/citadel/interview/message-queue).
- **Storage** — sensor readings are timestamped, so [time-series databases](/citadel/tech/timeseries-db) (InfluxDB, Amazon Timestream) fit them; [object storage](/citadel/interview/storage-systems) (S3, Azure Blob) holds raw dumps, logs, and firmware images; a SQL or NoSQL database holds device metadata, user accounts, and configuration.
- **Processing and analytics** — apply rules, run analytics, train models for predictive maintenance and anomaly detection.
- **Device management** — device identities, provisioning, over-the-air firmware updates, health monitoring.
- **APIs** — expose data and control securely to the application layer.

## Layer 4: applications

What consumes the data and closes the loop: web and mobile dashboards for monitoring, control apps for adjusting devices remotely, automated systems that act on readings without a human (irrigation that runs when soil moisture drops), BI tools for historical analysis, and ML services for prediction.

## The protocols

**Short-range, device to gateway:**

- **Bluetooth Low Energy** — short range, low power; wearables, smart-home gadgets.
- **Zigbee and Z-Wave** — low-power mesh networking; home automation.
- **LoRaWAN** — kilometres of range at very low power; smart cities, agriculture.

**Application layer, toward the cloud:**

- **MQTT** — a lightweight publish/subscribe protocol built for constrained devices on low-bandwidth, unreliable networks. The common default for IoT ingestion.
- **CoAP** — a compact web-transfer protocol for constrained nodes, usually over UDP.
- **HTTP/HTTPS** — fine for capable devices and gateway-to-cloud links; too heavy for simple battery-powered sensors.
- **gRPC** — for more capable devices and gateways talking to backends, where streaming or strong typing helps. See [API styles](/citadel/tech/apis).

## Edge computing

**Edge computing** moves computation and storage closer to where data is produced, rather than sending everything to a central cloud. The gateway is where it usually happens. The reasons:

- **Latency** — a local decision doesn't wait on a round trip to the cloud. Essential for autonomous vehicles, industrial robotics, traffic control.
- **Bandwidth** — filtering and aggregating locally cuts what you pay to transmit and store.
- **Privacy** — sensitive data can be processed on-site and never leave.
- **Autonomy** — the system keeps working when the cloud link drops.

## Security, at every layer

IoT widens the attack surface — many cheap devices, often physically accessible, often long-lived. The points that have to hold:

- **Device authentication and authorization** — only legitimate devices connect and send data.
- **Encryption in transit and at rest** — TLS or DTLS on the wire.
- **Signed firmware updates** — the OTA process is a prime tampering target.
- **Physical tamper resistance** — for devices in the field.
- **Network segmentation** — protect gateways and backends.
- **Data privacy** — govern the potentially sensitive data these devices collect.

## Where it's used

Smart homes (lighting, thermostats, security, voice assistants); wearables (fitness trackers, medical monitors); smart cities (traffic, street lighting, parking, environmental and waste monitoring); Industrial IoT (predictive maintenance, process optimisation, supply-chain tracking, precision agriculture); connected healthcare (remote monitoring, implants, asset tracking); automotive (telematics, fleet management, usage-based insurance); retail (smart shelves, inventory).

## The takeaway

An IoT system is four layers and the protocols between them, and both the protocol choices and the pull toward edge computing come from the same two constraints: devices have little power and networks are unreliable. Design from those constraints outward, and treat every layer as part of the attack surface, because a fleet of cheap physical devices is exactly that.
