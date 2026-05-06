---
title: LLD - Rate Limiter
description: The class-design version of the interview favourite. A RateLimiter interface, the algorithm behind a Strategy, per-key state in a concurrent map, and a Store seam so the same classes run single-node or backed by Redis.
date: 2026-05-06
draft: false
slug: /system-design/rate-limiter-lld
tags:
  - System Design
  - Low-Level Design
  - OOD
---

`rateLimiter.tryAcquire(key)` returns `true` or `false` in microseconds, based on how many calls `key` has made recently. The [algorithms](/citadel/interview/rate-limiting) — token bucket, sliding window — are covered elsewhere; the [distributed design](/citadel/system-design/rate-limiter) is its own post. This one is the object model: how to structure it so the algorithm is swappable, the per-key state is thread-safe, and moving from an in-memory counter to a Redis-backed one doesn't touch the algorithm code.

## Scope

- `boolean tryAcquire(String key)` and `boolean tryAcquire(String key, int permits)`.
- Configurable limit and window per key (or per key pattern).
- Thread-safe, low overhead.
- Pluggable algorithm; pluggable backing store (local vs Redis).

## The interface and strategy

```java
interface RateLimiter {
    boolean tryAcquire(String key, int permits);
    default boolean tryAcquire(String key) { return tryAcquire(key, 1); }
}

interface RateLimitAlgorithm {
    /** @return true if `permits` are allowed for `key` right now */
    boolean allow(String key, int permits, Rule rule, Store store, long nowMillis);
}

record Rule(long limit, Duration window, long burst) {}
```

The algorithm is a Strategy. Each implementation reads and writes its state through `Store`, never a concrete map — that's the seam that lets the same class run local or distributed.

## Token bucket implementation

```java
class TokenBucketAlgorithm implements RateLimitAlgorithm {
    public boolean allow(String key, int permits, Rule rule, Store store, long now) {
        double refillPerMs = (double) rule.limit() / rule.window().toMillis();
        return store.compute(key, state -> {           // atomic read-modify-write on the store
            double tokens = state == null ? rule.burst() : state.tokens();
            long last     = state == null ? now : state.timestamp();
            tokens = Math.min(rule.burst(), tokens + (now - last) * refillPerMs);
            boolean ok = tokens >= permits;
            if (ok) tokens -= permits;
            return new AllowResult(ok, new State(tokens, now));
        }).allowed();
    }
}
```

`SlidingWindowCounterAlgorithm`, `FixedWindowAlgorithm`, `LeakyBucketAlgorithm` implement the same method with their own state shape. The key requirement each imposes on `Store`: the read-modify-write must be **atomic per key**, or two threads both see 1 remaining token and both take it.

## The Store seam

```java
interface Store {
    /** Atomically apply `fn` to the state for `key` and return its result. */
    <R> R compute(String key, Function<State, StateAnd<R>> fn);
}

class LocalStore implements Store {
    private final ConcurrentHashMap<String, State> map = new ConcurrentHashMap<>();
    public <R> R compute(String key, Function<State, StateAnd<R>> fn) {
        var holder = new Object() { R result; };
        map.compute(key, (k, cur) -> {                 // ConcurrentHashMap.compute is atomic per key
            StateAnd<R> out = fn.apply(cur);
            holder.result = out.result();
            return out.newState();
        });
        return holder.result;
    }
}

class RedisStore implements Store {
    // compute() ships the whole read-modify-write as one Lua script, so it's
    // atomic across every process sharing this Redis. Same algorithm classes,
    // different Store.
}
```

`LocalStore` gets atomicity from `ConcurrentHashMap.compute` (which locks only that key's bin). `RedisStore` gets it from a Lua script Redis runs atomically. The `TokenBucketAlgorithm` code above is identical either way.

## Wiring it up

```java
class ConfigurableRateLimiter implements RateLimiter {
    private final RateLimitAlgorithm algorithm;
    private final Store store;
    private final RuleResolver rules;      // key or key-pattern -> Rule
    private final Clock clock;             // injected for testability

    public boolean tryAcquire(String key, int permits) {
        Rule rule = rules.resolve(key);
        return algorithm.allow(key, permits, rule, store, clock.millis());
    }
}
```

Injecting `Clock` matters: tests set a fake clock and assert "5 requests in the first second pass, the 6th fails, one advances the clock 200 ms and a request passes again" without real sleeps.

## Composed limits

A request may be subject to several rules at once — per user *and* per IP. Model that as a `CompositeRateLimiter` that consults each and admits only if **all** allow. Order matters slightly (fail fast on the cheapest / most-likely-to-reject check) and, on rejection, you should still not consume permits from the limiters that would have allowed it — so either check-all-then-commit, or accept a small over-count.

## Concurrency summary

- Per-key atomicity comes from the `Store` (`ConcurrentHashMap.compute` or a Lua script), not from a lock in the algorithm.
- No global lock — different keys never contend.
- The rule table is read-mostly; `CopyOnWriteArrayList` or an immutable snapshot swapped on reload.

## Extensions

- **Blocking acquire** — `acquire(key)` that waits (up to a timeout) for a permit instead of returning `false`; compute the wait from the token deficit and the refill rate.
- **Cost-weighted requests** — expensive endpoints pass `permits > 1`.
- **Metrics hook** — an `onDecision(key, allowed)` callback for dashboards.
- **Hierarchical limits** — per-endpoint under per-user under per-tenant, evaluated as a composite.

## The one idea to keep

Put the algorithm behind a `RateLimitAlgorithm` strategy and the per-key state behind a `Store` interface whose `compute` is atomic per key. Then `ConcurrentHashMap.compute` and a Redis Lua script are just two `Store` implementations, and the token-bucket code is written once and runs single-node or distributed unchanged. Inject the `Clock` so the whole thing is testable without sleeping.
