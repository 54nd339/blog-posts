---
title: LLD - Notification Service
description: One send() call, many channels. The design is a Channel strategy per medium, a preferences check, a template renderer, and a retry-with-backoff wrapper - the object model behind the notification system, sized for a single service rather than a distributed one.
date: 2026-06-10
draft: false
slug: /system-design/notification-service
tags:
  - System Design
  - Low-Level Design
  - OOD
---

A dozen places in an application need to notify a user — order shipped, password changed, someone commented. They shouldn't each know about email providers, SMS gateways, and push tokens. They call `notify(user, event)` and a notification service works out which channels to use, renders the message from a template, sends it, and retries on failure.

This is the class-level design (the [system-level version](/citadel/system-design/notifications) covers scale and delivery infrastructure). The pieces: a `Channel` per medium, a preferences check, a template renderer, and a retry policy.

## Scope

- `send(Notification)` across channels: email, SMS, push, in-app.
- Per-user channel preferences and quiet hours.
- Templates with variable substitution and localization.
- Retries with backoff; deduplication; delivery status.
- Out of scope: the queue, the worker fleet, provider sharding (mention them).

## Model

```java
enum ChannelType { EMAIL, SMS, PUSH, IN_APP }
enum Priority { LOW, NORMAL, HIGH, CRITICAL }

record Notification(String id,           // idempotency key
                    String userId,
                    String templateKey,
                    Map<String, Object> params,
                    Priority priority) {}

interface Channel {
    ChannelType type();
    DeliveryResult send(User user, RenderedMessage msg);   // may throw TransientException
}

interface TemplateRenderer {
    RenderedMessage render(String templateKey, ChannelType channel,
                           Locale locale, Map<String, Object> params);
}
```

`Channel` is the Strategy: `EmailChannel` wraps an SMTP/SES client, `SmsChannel` wraps Twilio, `PushChannel` wraps APNs/FCM, `InAppChannel` writes a row. Each is also an **Adapter** — it maps the generic `RenderedMessage` onto that provider's API.

## The dispatcher

```java
class NotificationService {
    private final Map<ChannelType, Channel> channels;
    private final PreferenceStore prefs;
    private final TemplateRenderer renderer;
    private final RetryPolicy retryPolicy;
    private final DedupStore dedup;
    private final RateLimiter rateLimiter;

    void send(Notification n) {
        if (!dedup.firstSeen(n.id())) return;                 // already processed

        User user = users.get(n.userId());
        List<ChannelType> targets = resolveChannels(user, n);

        for (ChannelType ct : targets) {
            if (!rateLimiter.allow(user.id(), ct)) { record(n, ct, SKIPPED_RATE_LIMIT); continue; }
            RenderedMessage msg = renderer.render(n.templateKey(), ct, user.locale(), n.params());
            dispatchWithRetry(channels.get(ct), user, msg, n, ct);
        }
    }

    private List<ChannelType> resolveChannels(User user, Notification n) {
        Preferences p = prefs.forUser(user.id());
        return p.enabledChannels(n.templateKey()).stream()
            .filter(ct -> n.priority() == Priority.CRITICAL || !p.inQuietHours(ct))
            .toList();
    }
}
```

`resolveChannels` is where preferences and quiet hours apply — except a `CRITICAL` notification (security alert) overrides quiet hours. The user's per-template opt-outs ("no email for comments, but yes for security") are checked here.

## Retry with backoff

```java
interface RetryPolicy {
    boolean shouldRetry(int attempt, Throwable last);
    Duration backoff(int attempt);          // e.g. base * 2^attempt + jitter
}

private void dispatchWithRetry(Channel ch, User u, RenderedMessage m,
                               Notification n, ChannelType ct) {
    int attempt = 0;
    while (true) {
        try {
            DeliveryResult r = ch.send(u, m);
            record(n, ct, r.status());        // DELIVERED / QUEUED / FAILED
            return;
        } catch (TransientException e) {
            if (!retryPolicy.shouldRetry(++attempt, e)) {
                deadLetter(n, ct, e); return;
            }
            schedule(() -> dispatchWithRetry(ch, u, m, n, ct), retryPolicy.backoff(attempt));
            return;                            // hand off to scheduler, don't sleep here
        } catch (PermanentException e) {       // bad address, unsubscribed
            record(n, ct, FAILED_PERMANENT); return;
        }
    }
}
```

Distinguish **transient** (provider 5xx, timeout — retry with exponential backoff + jitter) from **permanent** (invalid number, hard bounce — don't retry, mark the channel bad for this user). After `maxAttempts`, dead-letter for inspection.

## Provider failover

A channel can have multiple providers (primary SES, fallback SendGrid). Wrap them: `EmailChannel` tries the primary; on a transient failure or an open **circuit breaker**, it uses the fallback. The circuit breaker (see [resilience patterns](/citadel/interview/resilience-patterns)) stops hammering a provider that's down.

## Batching and digests

Low-priority notifications can be **coalesced**: instead of five "someone commented" emails, one "5 new comments" digest sent on a schedule. A `DigestBuffer` per user per template accumulates entries and a scheduled job flushes them. High-priority notifications skip the buffer.

## Concurrency and idempotency

- `send` is called from many threads and from a queue consumer; the `DedupStore` (a set of seen `Notification.id`s with a TTL) makes reprocessing a no-op — important because the upstream queue is at-least-once.
- Per-user, per-channel rate limits use an atomic counter (see [rate limiting](/citadel/interview/rate-limiting)).
- Channels are stateless; providers handle their own concurrency.

## Extensions

- **Scheduling** — `sendAt(notification, instant)`; hands to a [scheduler](/citadel/system-design/distributed-scheduler).
- **Localization** — templates per locale; `render` picks by `user.locale()`, falls back to default.
- **Unsubscribe** — a per-template, per-channel toggle in `Preferences`, plus a one-click link in emails.
- **Webhooks** — a `WebhookChannel` for machine recipients; delivery-status callbacks update `record`.

## The one idea to keep

A notification service is a `Channel` strategy per medium (each also an adapter onto a provider's API), fronted by a dispatcher that resolves the target channels from the user's preferences and quiet hours — with `CRITICAL` overriding quiet hours. Wrap each send in a retry policy that separates transient failures (exponential backoff) from permanent ones (stop, mark bad), dedupe on the notification ID because the upstream is at-least-once, and coalesce low-priority messages into digests.
