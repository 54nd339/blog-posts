---
title: Webhooks - Getting Notified Instead of Polling
description: Instead of your app repeatedly asking an external service "done yet?", the service calls a URL you registered when the event happens. The workflow, and the four things — missed callbacks, fast handling, authenticity, idempotency — that make it robust.
date: 2024-11-21
draft: false
slug: /interview/webhook
tags:
  - System Design
  - Interview Prep
  - APIs
---

Your app kicks off a payment at an external provider and needs to know the outcome. The naive way is **polling**: send "is it done yet?" every few seconds until the answer changes. A **webhook** flips that — you give the provider a URL, and it calls *you* when the event happens.

## Why not poll

Short polling an external service for a status has three problems:

- **Wasteful** — every poll costs a request on your side and load on theirs, and most polls return "not yet".
- **Laggy** — you learn about an event only at the next poll, so there is always a delay of up to one interval.
- **Awkward** — frequent outbound status checks mean more firewall rules and connections than the job needs.

## How webhooks work

A webhook is an HTTP callback: the service makes a request to your application when it has something to report. Sometimes called a "reverse API" or "push API" because the direction is inverted.

1. **Register.** You give the external service a publicly reachable URL — "when event X happens, POST the details here".
2. **Event occurs** at the service (payment succeeded, payment failed).
3. **Callback.** The service sends an HTTP `POST` to your URL with a payload describing the event.
4. **Act.** Your endpoint receives it, processes the payload, and updates state or notifies the user.

Your service never polls; it waits to be called.

## Making it robust

- **Handle missed callbacks.** The service's callback can be lost to a network fault on its end. Run a scheduled job that reconciles — for any payment with no webhook after a timeout, poll the provider once for its status. Webhooks as the fast path, polling as the backstop.
- **Respond fast.** The calling service is waiting on your `200 OK`. Validate the payload, drop the event onto an internal [queue](/citadel/interview/message-queue) for asynchronous processing, and acknowledge immediately — do not do the slow work inline.
- **Verify authenticity.** The URL is public, so anyone can `POST` to it. Confirm the request is really from the provider: an HMAC signature over the body using a shared secret (the common method), or an IP allowlist. See [securing a web API](/citadel/interview/secure-web-api).
- **Be idempotent.** The service will retry if it does not get a timely `200`, so the same event can arrive more than once. Key handling on the event ID so a duplicate is a no-op, not a double charge. This is the same discipline as [retry handling](/citadel/interview/data-systems).

Also: keep the registered URL correct and live — a stale URL silently drops every notification.

## Webhooks are polling, moved to the sender

The event still has to travel from producer to consumer; the webhook just makes the producer responsible for pushing it when it is ready, instead of the consumer asking on a timer. That removes the waste and the lag — at the cost of a public endpoint you must secure, a handler you must keep fast, and a reconciliation job for when the push does not arrive. For a steady internal stream rather than sporadic third-party events, a [message queue](/citadel/interview/message-queue) is usually the better shape.
