---
title: How Email Works - SMTP, Queues, and the Path From Send to Inbox
description: Hitting Send drops a message into a queue, where SMTP workers pick it up, find the recipient's server by DNS, and hand it over. The receiving side queues it again for spam and virus checks before storage. A walk through both halves.
date: 2025-01-02
draft: false
slug: /system-design/emails
tags:
  - System Design
  - Interview Prep
  - Networking
---

Email is old and still a resilient distributed system: several protocols, fleets of specialized servers, queues at every stage, and a lot of filtering. Clicking Send doesn't transmit anything to the recipient — it enqueues work. Here's the path, both directions.

## The protocols

- **SMTP** — sends mail: client to server, then server to server toward the recipient's mail server.
- **POP3** — a client retrieval protocol; downloads messages to the device and often deletes them from the server.
- **IMAP** — a client retrieval protocol; keeps messages on the server and syncs state, so every device sees the same mailbox.

## Sending

![The send path: client to load balancer to web server, an intra-domain shortcut to internal storage, otherwise into an outgoing queue that SMTP workers drain, doing an MX lookup and transmitting over SMTP](../images/email-send.png "Send path: web server, outgoing queue, SMTP workers, MX lookup, transmit.")

1. **Client to web server.** You compose and hit Send. For webmail, the request goes through a load balancer (which rate-limits) to one of the sender's web servers.
2. **Basic validation.** The web server checks simple things like size limits.
3. **Intra-domain shortcut.** If sender and recipient share a domain (`a@example.com` → `b@example.com`), skip the external machinery: write the message straight into internal storage, cache, and object store, where the recipient's client can fetch it immediately.
4. **Queue for reliability.** For an external domain, the message goes onto an **outgoing queue** (failures to an error queue). The queue decouples the web servers from the SMTP workers: the web server can confirm "sent" as soon as the message is queued, workers scale independently, and the queue absorbs surges.
5. **SMTP workers.** Dedicated outgoing workers pull from the queue, run heavier spam and virus scanning, and store a copy in the sender's Sent folder.
6. **Delivery.** A worker does a DNS lookup for the recipient domain's **MX (mail exchange) records** to find its mail server, and transmits over SMTP. If the recipient server is down, workers retry with exponential backoff; a growing outgoing queue is the signal to add workers.

## Receiving

![The receive path: incoming SMTP hits a load balancer and SMTP server fleet, early rejection of spam and invalid recipients, large attachments to object storage, an incoming queue drained by processing workers, then storage and real-time or on-demand delivery](../images/email-recieve.png "Receive path: SMTP fleet, early rejection, incoming queue, processing workers, storage.")

1. **Arrival.** Incoming SMTP connections from other servers hit an SMTP load balancer for the domain, spread across an SMTP server fleet.
2. **Early rejection.** At connection level, known spam sources and mail for invalid recipients are bounced immediately, before wasting downstream work.
3. **Attachments.** Very large attachments are offloaded to an attachment store (such as S3) so they don't clog queues and mail stores.
4. **Incoming queue.** Accepted mail goes onto an incoming queue — same decoupling and buffering as the send side.
5. **Processing workers.** These do the heavy lifting: full spam filtering, virus scanning, and user-defined rules ("move mail from X to folder Y").
6. **Storage.** Mail that passes is written to the primary mail store, a cache for recent messages, and an object store for archival and large parts.
7. **Delivery.** If the recipient is online with an active client, the arrival is pushed via real-time servers holding [WebSocket](/citadel/interview/socket-program) connections, so it appears instantly. Otherwise it waits in storage until the client polls — over REST for webmail, or via IMAP/POP servers that read from the store.

## End to end

Alice (Outlook) to Bob (Gmail): Alice's client sends over SMTP to an Outlook mail server; that server does an MX lookup for Gmail and transfers the message over SMTP to a Gmail SMTP server; Gmail processes and stores it; Bob's client later fetches it over IMAP or POP3.

## The takeaway

Every stage that could block is a queue instead: the web server confirms your send the moment the message is durably queued, and the slow parts — remote-server round trips, spam and virus scanning, retries — happen in workers behind it. See [message queues](/citadel/interview/message-queue) for that pattern and [internet routing](/citadel/interview/internet-routing) for the DNS and MX resolution the delivery step depends on.
