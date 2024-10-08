---
title: Firewalls - From Packet Filters to Next-Gen
description: A firewall decides which traffic crosses a network boundary, and the depth of that decision has grown from packet headers to full application awareness. The five firewall types by OSI layer, how rules are written, and the defences that sit alongside them.
date: 2024-10-08
draft: false
slug: /interview/firewall
tags:
  - System Design
  - Interview Prep
  - Network Security
---

A **firewall** sits on the boundary between a trusted network and an untrusted one — usually the internet — and decides, per a configured ruleset, which traffic may cross. It can be software on one host or a hardware appliance guarding a whole network. What has changed over decades is not the job but the *depth* of the inspection: from reading packet headers to understanding the application inside the packet.

This walks the firewall types by how deep they look, how rules are expressed, and the other network defences a firewall is usually deployed with.

## The types, by inspection depth

Each maps roughly to a layer of the OSI model.

- **Packet-filtering** (L3–L4) — accept or drop each packet on its headers alone: source and destination IP, source and destination port, protocol. Fast and simple; blind to content and to whether a packet belongs to a real conversation.
- **Circuit-level gateway** (L5) — watches the TCP handshake to decide a session is legitimate, then lets its packets flow without inspecting them. Good at hiding internal structure; no content filtering, so a hostile packet on an established connection passes.
- **Application-level gateway / proxy firewall** (L7) — terminates the connection and acts as a proxy for a specific protocol (HTTP, FTP, SMTP), so it can read the content and enforce protocol-specific rules. Strong protection; adds latency, and often needs a separate proxy per protocol.
- **Stateful inspection** — tracks the state of every active connection (setup, data, teardown) and admits a packet only if it fits an existing, legitimate session. More secure than stateless filtering, more resource-hungry.
- **Next-generation firewall (NGFW)** — stateful inspection plus deep packet inspection of the payload, an integrated intrusion prevention system, application identification independent of port, and threat-intelligence feeds. The most capable, the most complex and costly to run.

## Writing rules

A firewall is only as good as its ruleset. Rules are typically keyed on:

- **Port** — allow 80 and 443 to a web server, deny the rest.
- **IP address** — allow-list trusted networks, block-list known-bad addresses or ranges.
- **Protocol** — permit TCP on 22 for SSH, deny UDP on it.
- **Time** — different rules by schedule, e.g. internal resources reachable only in business hours.
- **Application** — in NGFWs and WAFs, allow Zoom and Teams while blocking BitTorrent, regardless of port.

The safe default is deny-all, then allow only what is needed.

## Beyond the firewall

A real network security posture is layered:

- **Web application firewall (WAF)** — an L7 filter specialised for HTTP(S), built to stop the [OWASP Top 10](/citadel/interview/secure-web-api) — SQL injection, XSS, file inclusion. Deployed in front of web servers or as a cloud service.
- **IDS / IPS** — an intrusion *detection* system monitors for malicious activity and alerts; an intrusion *prevention* system also acts on it, blocking the source or killing the session. NGFWs usually bundle an IPS.
- **DDoS protection** — services that absorb volumetric floods from botnets using traffic scrubbing, rate limiting, and large global capacity.
- **[VPN](/citadel/interview/vpn)** — an encrypted tunnel over the public internet for remote access or site-to-site links, protecting data in transit and masking the origin IP.
- **Network segmentation** — divide the network into isolated zones with their own policies, so a breach in one is contained instead of spreading.
- **Zero trust network access (ZTNA)** — "never trust, always verify": no user or device is trusted by network location. Every request is checked against identity, device posture, and context, and granted narrowly for one resource for a limited time.

## The perimeter is necessary and not sufficient

A firewall enforces a boundary, and boundaries still matter — but the zero-trust shift exists because the boundary leaks: remote workers, cloud services, and lateral movement after a single compromised host all route around it. The modern answer is a firewall at the edge *and* segmentation inside *and* per-request verification, so that getting past one control does not grant the run of the network. The [secure web API](/citadel/interview/secure-web-api) post covers the application-layer half of this.
