---
title: Designing a Notification System - The Decision Logic and Push Delivery
description: "Two problems in one: deciding whether a given event is worth interrupting a user for, and delivering the alert to their device through Apple's or Google's push service. The decision tree, the device-token dance, and the reliability gaps to design around."
date: 2025-07-23
draft: false
slug: /system-design/notifications
tags:
  - System Design
  - Interview Prep
  - Messaging
---

A notification system has two jobs that pull in opposite directions: deliver timely, relevant alerts, and don't send so many that the user turns them off. The first is a decision problem — is this event worth an interruption? The second is a delivery problem — get the payload to a specific device through a platform push service. This covers both.

Notifications come as **push** (server to device via APNS or FCM, app closed), **in-app**, **email**, and **SMS**. The focus here is push.

## Deciding whether to notify

Before anything is sent, a decision tree runs — the same kind [Slack publishes](/citadel/system-design/slack), and a good example of a "simple" feature carrying a lot of hidden logic. The inputs:

- **Global preference** — has the user muted this app entirely?
- **Channel or topic preference** — muted this specific conversation?
- **Do Not Disturb** — in a DnD window, and does anything override it?
- **Event type** — direct message, `@channel` mention, keyword highlight, or a routine update?
- **Presence** — already active on desktop, making a phone push redundant or worth delaying?
- **Content** — does it contain a word the user asked to be alerted on?
- **Subscription** — is the user following this thread or item?

## The components

- **Client app** — registers for push and displays notifications.
- **Application backend** — detects the event and decides a notification is warranted.
- **Notification service** — a dedicated service that composes, manages, and dispatches.
- **Push notification service (PNS)** — the OS vendor's system: **APNS** (Apple), **FCM** (Google, also web), WNS (Windows), SMS gateways.
- **Database** — device tokens, user preferences, sent history.
- **Message queue** (Kafka, RabbitMQ, SQS) — decouples the backend from the notification service and buffers peaks.
- **Worker pool** — consumes requests, builds platform-specific payloads, calls the PNS.

## The workflow

![The push flow: the app registers with APNS/FCM for a device token stored in the backend, an event publishes to a queue, workers compose payloads and dispatch to the PNS, which delivers to the device](../images/notifications.png "Registration for a device token, then event to queue to worker to PNS to device.")

1. **Device registration** (once per install). On first launch or permission grant, the app registers with APNS or FCM. The PNS returns a **device token** — the address for that app instance on that device. The app sends the token to the backend, which stores it against the user ID and their preferences.
2. **Event trigger.** Something happens (a DM arrives, a transcode finishes). The backend picks the target users and content and publishes a request to the message queue.
3. **Notification service.** Workers consume the request and **compose** the payload — fetch the user's language for localization, look up their device tokens, add sound, badge count, and a deep link (which opens a specific screen when tapped). Payload shape differs between APNS and FCM. Then **dispatch** to the right PNS per device.
4. **Delivery by PNS.** Apple's and Google's globally distributed systems hold connections to devices and deliver the payload.
5. **User interaction.** The device OS shows the notification per the user's settings; a tap opens the app, which handles the deep link.

For users already **in the app**, skip the PNS: a [WebSocket](/citadel/interview/socket-program) to a real-time server lets the backend push in-app updates directly.

## Design considerations

- **Scalability** — millions of tokens and high send volume at peak; scale the queue, notification service, and workers.
- **Reliability** — a PNS makes a best effort, not a guarantee (offline devices). Handle PNS feedback for invalid or unregistered tokens by pruning them, and use persistent queues with retries on both hops.
- **Latency** — transactional notifications must arrive quickly to matter.
- **Security** — protect device tokens, TLS everywhere, authenticate send requests to prevent spam and spoofing.
- **Throttling and batching** — cap how many notifications a user gets in a window, and batch to protect the PNS and SMS gateways.
- **Analytics** — track delivery, open, and error rates and pipeline latency.

## The takeaway

The delivery pipeline is standard queue-and-worker plumbing; the part that decides whether an app is pleasant or muted is the decision tree at the front. Both halves lean on [a message queue](/citadel/interview/message-queue) to keep the sending backend responsive while workers deal with the slow, unreliable last hop to Apple and Google.
