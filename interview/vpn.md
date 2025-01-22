---
title: VPNs - How an Encrypted Tunnel Reroutes Your Traffic
description: A VPN wraps your device's traffic in an encrypted tunnel to a server that then talks to the internet on your behalf. The connection setup, what each party can and cannot see, and how it differs from a plain proxy.
date: 2025-01-22
draft: false
slug: /interview/vpn
tags:
  - System Design
  - Interview Prep
  - Network Security
---

A **VPN (virtual private network)** builds an encrypted tunnel between your device and a remote server, and routes your internet traffic through it. Two things change as a result: everything between you and the VPN server is unreadable to anyone on your local network, and the wider internet sees your traffic as coming from the VPN server's address, not yours.

This covers how the tunnel is set up, what each party can observe, the practical caveats, and the line between a VPN and a proxy.

## How it works

1. **Launch the VPN client** on your device.
2. **Establish the tunnel.** The client connects to a VPN server (run by the provider, in a location you often choose). It authenticates, then negotiates a **tunnelling protocol** (OpenVPN, IKEv2/IPsec, WireGuard) and encryption algorithms.
3. **Route traffic in.** All (or a configured subset of) your device's internet traffic goes into the tunnel to the VPN server.
4. **Server decrypts and forwards.** The VPN server decrypts your request and sends it to its actual destination on the public internet — from *its* IP address.
5. **Response comes back** to the VPN server, which encrypts it and returns it through the tunnel.
6. **Client decrypts** the response and hands it to your application.

## Who sees what

- **Someone on your local network** (a café Wi-Fi snooper) sees encrypted traffic to one server and nothing about its contents.
- **Your ISP** sees that you are connected to a VPN, but not which sites you visit or what you send.
- **The websites you visit** see the VPN server's IP and location, not yours — so IP-based tracking and geolocation point at the server.
- **The VPN provider** sees everything your ISP normally would. This is why the provider's trustworthiness is the whole game.

## What it is used for

- **Security on untrusted networks** — encryption stops eavesdropping on public Wi-Fi.
- **Privacy** — masks your real IP from sites and trackers; hides browsing content from your ISP.
- **Geo-unblocking and censorship bypass** — connect through a server in another country to reach region-locked content or sites blocked locally.
- **Corporate remote access** — the original use: employees tunnel into the company's private network to reach internal resources as if on-site.

## Caveats

- **Protocol choice** — OpenVPN, IKEv2/IPsec, WireGuard, L2TP/IPsec differ in speed, security, and device support; WireGuard is the modern, fast default.
- **Provider trust** — a strict **no-logs policy** matters most; also server spread, speed, a **kill switch** (cuts internet if the tunnel drops), and DNS-leak protection. Be wary of "free" VPNs — the business model is often selling what you were hiding.
- **Downsides** — encryption and the extra hop cost some speed; reputable services are paid; streaming platforms actively block known VPN IPs; legality varies by country (legal for privacy in most, restricted or banned in a few).

## VPN versus proxy

Both route through an intermediary, but a [proxy](/citadel/interview/proxies) typically handles one application's traffic (your browser), often without encryption, at the application level. A VPN encrypts **all** the device's traffic at the OS level. The proxy changes your apparent origin; the VPN changes your apparent origin *and* protects the data in transit.

## The trust just moves

A VPN does not make you anonymous — it moves the party who can see your traffic from your ISP and local network to the VPN provider, and asks you to trust that provider instead. That is a good trade on hostile Wi-Fi or against a censoring ISP, and a poor one if the provider logs and sells. It is a [network security](/citadel/interview/firewall) tool with a specific job — protect the link and relocate the exit point — not a blanket privacy guarantee.
