---
title: HTTP - The Request/Response Model and Its Evolution
description: The stateless client-server protocol under the web, its methods and headers, and the version history from connection-per-request to HTTP/2 multiplexing to HTTP/3 over QUIC, each release chasing one source of latency.
date: 2024-07-23
draft: false
slug: /interview/http
tags:
  - System Design
  - Interview Prep
  - HTTP
---

Every link you click is an **HTTP** exchange: your browser sends a request for a resource, a server sends back a response with that resource or an error. The model is simple and has barely changed. What has changed, across four versions, is the transport underneath — and each change targeted one specific way the old version wasted time.

This covers the request and response structure, then walks the version history and the latency problem each release solved.

## Stateless, plus cookies

HTTP is **stateless**: the server remembers nothing between requests, each one stands alone. State — a login, a cart — is bolted on with **cookies**: the server sends a small value with `Set-Cookie`, the browser stores it and returns it on every later request to that origin, letting the server correlate them into a session. The mechanics are in [sessions and cookies](/citadel/interview/session-cookies).

## The request

- **URL** — `https://example.com/product/phone?color=black`: scheme, domain, path, and optional **query parameters** after `?`, joined by `&`, used to filter or modify the request.
- **Method** — the action on the resource:

| Method | Request body | Response body | Safe | Idempotent | Cacheable |
| --- | --- | --- | --- | --- | --- |
| GET | optional | yes | yes | yes | yes |
| HEAD | optional | no | yes | yes | yes |
| POST | yes | yes | no | no | yes |
| PUT | yes | yes | no | yes | no |
| DELETE | optional | yes | no | yes | no |
| PATCH | yes | yes | no | no | no |
| OPTIONS | optional | yes | yes | yes | no |
| TRACE | no | yes | yes | yes | no |
| CONNECT | optional | yes | no | no | no |

**Safe** means it does not modify the resource. **Idempotent** means N identical requests have the same effect as one — which is why a client can safely retry a failed `PUT` or `DELETE` but not a `POST`. **Cacheable** means the response may be stored and reused.

- **Headers** — metadata: `Accept` and `Accept-Encoding` (what formats and compressions the client will take), `Content-Type` and `Content-Length` (what the body is), `Cookie`, `Cache-Control`, `Authorization`, `User-Agent`, `Referer`.
- **Body** — present for `POST`, `PUT`, `PATCH`: form data, a JSON payload, an upload.

## The response

- **Status line** — version, code, reason: `HTTP/1.1 200 OK`.
- **Status code** — three digits in five classes (1xx informational, 2xx success, 3xx redirect, 4xx client error, 5xx server error), detailed in [HTTP status codes](/citadel/interview/http-codes).
- **Headers** — `Content-Type`, `Content-Length`, `Location` (for redirects), `Set-Cookie`, `Cache-Control`, `Access-Control-Allow-Origin` (CORS), `Alt-Svc` (advertises HTTP/3).
- **Body** — the resource, or an error.

## The evolution, by the latency it removed

![HTTP versions on a timeline. HTTP/1.0 opens a fresh TCP connection per request (SYN, SYN-ACK, ACK, then one request and response). HTTP/1.1 keeps one connection open and reuses it for many requests. HTTP/2 interleaves numbered stream frames over a single TCP connection. HTTP/3 runs independent streams over a QUIC-on-UDP connection so a lost packet stalls only its own stream.](../images/http-evolution.png "Each version attacks a different delay: connection setup (1.0 to 1.1), application-layer head-of-line blocking (1.1 to 2), and transport-layer head-of-line blocking (2 to 3).")

**HTTP/1.0 (1996).** A separate TCP connection per request. Loading one page's HTML, CSS, and images meant repeated connection setups — pure overhead.

**HTTP/1.1 (1997).** **Keep-alive**: one TCP connection stays open and is reused for many requests. Pipelining allowed sending several requests without waiting, but responses still had to come back in order, and browsers capped parallel connections per domain at 6–8. A slow response still blocked everything queued behind it on its connection — **application-layer head-of-line blocking**.

**HTTP/2 (2015).** A **binary framing layer** splits messages into frames, and **multiplexing** interleaves many independent **streams** over a *single* TCP connection, in any order — killing application-layer head-of-line blocking. Plus **stream prioritisation**, **server push**, and **HPACK** header compression. One problem remained: it still runs on TCP, so a single lost packet stalls *every* stream on that connection until it is retransmitted — **transport-layer head-of-line blocking**.

**HTTP/3 (QUIC, ~2020).** Replaces TCP with **QUIC**, built on UDP, where streams are first-class at the transport layer and delivered independently — a lost packet stalls only its own stream. QUIC also folds the connection and TLS 1.3 handshakes together, cutting round trips to establish a secure connection, and supports **connection migration** so a session survives a phone switching from Wi-Fi to cellular.

## The model is stable; the plumbing keeps moving

Thirty years on, the request-method-headers-body shape is the same one from 1996 — which is why it is worth knowing cold. Every version bump has been underneath it, chasing the next dominant source of delay: connection setup, then blocking at the HTTP layer, then blocking at the transport layer. HTTP/3's connection migration hints at where the pressure is now — a web assumed to be mostly mobile and frequently changing networks.
