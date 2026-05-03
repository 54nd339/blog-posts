---
title: LLD - LRU Cache (and LFU)
description: O(1) get and put with a hash map for lookup and a doubly linked list for recency order. Then generalize eviction behind an interface so LFU, FIFO, and TTL are policies rather than rewrites, and make the whole thing thread-safe.
date: 2026-05-03
draft: false
slug: /system-design/lru-cache
tags:
  - System Design
  - Low-Level Design
  - OOD
---

A fixed-capacity cache: `get(key)` and `put(key, value)`, both O(1), and when it's full, evicting the entry that's been unused the longest. The reason it's a classic is that neither a hash map alone nor a list alone works — the hash map gives O(1) lookup but no order, the list gives order but O(n) lookup. You need both, wired together.

Once that's built, the interesting design question is making the *eviction rule* a plug-in, so LFU (least *frequently* used), FIFO, and TTL are policies, not forks of the code.

## The LRU core

- **Hash map** `key → node` for O(1) lookup.
- **Doubly linked list** of nodes in recency order: most-recently-used at the head, least at the tail. Doubly linked so a node can be unlinked in O(1) given a pointer to it.
- Sentinel `head` and `tail` nodes remove all the null-checks.

```java
class LRUCache<K, V> {
    private final int capacity;
    private final Map<K, Node<K, V>> map = new HashMap<>();
    private final Node<K, V> head = new Node<>(), tail = new Node<>();

    LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail; tail.prev = head;
    }

    V get(K key) {
        Node<K, V> n = map.get(key);
        if (n == null) return null;
        moveToFront(n);                 // touch = becomes most-recently-used
        return n.value;
    }

    void put(K key, V value) {
        Node<K, V> n = map.get(key);
        if (n != null) { n.value = value; moveToFront(n); return; }
        if (map.size() == capacity) {
            Node<K, V> lru = tail.prev;  // least-recently-used
            unlink(lru);
            map.remove(lru.key);
        }
        Node<K, V> fresh = new Node<>(key, value);
        map.put(key, fresh);
        addToFront(fresh);
    }

    private void moveToFront(Node<K, V> n) { unlink(n); addToFront(n); }
    private void unlink(Node<K, V> n)      { n.prev.next = n.next; n.next.prev = n.prev; }
    private void addToFront(Node<K, V> n)  {
        n.prev = head; n.next = head.next;
        head.next.prev = n; head.next = n;
    }
}
```

Every operation is a hash lookup plus a constant number of pointer swaps — O(1).

> Java's `LinkedHashMap` with `accessOrder = true` and an overridden `removeEldestEntry` *is* an LRU cache in a few lines. Know that, and know how to build it by hand — the interview usually wants the hand-built version.

## Generalizing eviction

Extract the "which entry goes next" decision:

```java
interface EvictionPolicy<K> {
    void onAccess(K key);          // get or update
    void onInsert(K key);
    K evictionCandidate();         // who to remove when full
    void onRemove(K key);
}
```

- **LRU policy** — the doubly linked list above; `onAccess`/`onInsert` move to front, `evictionCandidate` returns the tail.
- **FIFO policy** — a plain queue; `onAccess` does nothing (insertion order only), `evictionCandidate` is the queue head.
- **LFU policy** — a frequency count per key plus buckets: `freq → linked set of keys at that frequency`, and a `minFreq` pointer. `onAccess` bumps a key to the next frequency bucket; `evictionCandidate` returns any key from the `minFreq` bucket (ties broken by LRU within the bucket). Still O(1), just more bookkeeping.
- **TTL** — an expiry per entry; `evictionCandidate` prefers an expired key, falling back to the wrapped policy. A background sweep or lazy check on `get` removes stale entries.

The `Cache` class holds a `Map<K,V>` and an `EvictionPolicy`, delegating the ordering decision:

```java
V get(K key) {
    if (!store.containsKey(key)) return null;
    policy.onAccess(key);
    return store.get(key);
}
void put(K key, V value) {
    if (!store.containsKey(key) && store.size() == capacity) {
        K victim = policy.evictionCandidate();
        store.remove(victim); policy.onRemove(victim);
    }
    boolean isNew = !store.containsKey(key);
    store.put(key, value);
    if (isNew) policy.onInsert(key); else policy.onAccess(key);
}
```

## Thread safety

The simplest correct version wraps `get` and `put` in one lock (`synchronized`, or a `ReentrantLock`). Contention is the cost — every read blocks every other read.

Better options:

- **Striped locking** — partition the key space into `S` segments, each with its own map + policy + lock. Independent keys don't contend. This is roughly what `ConcurrentHashMap` does internally and what Guava's cache does.
- **`ConcurrentHashMap` + an approximate recency structure** — real high-performance caches (Caffeine, built on the **W-TinyLFU** policy) don't keep an exact LRU list under a lock; they record accesses in per-thread ring buffers and replay them into the eviction structure in batches, so reads are lock-free and eviction order is very-nearly-LRU. Mention this as the production answer.

## Extensions

- **Size-weighted entries** — capacity in bytes, each entry has a weight; evict until under budget.
- **Loading cache** — `get(key, loader)` computes and caches on miss, with single-flight so concurrent misses for one key call the loader once.
- **Write-through / write-back hooks** — a listener on eviction and on put.
- **Stats** — hit/miss counters for tuning.

## The one idea to keep

An LRU cache is a hash map for O(1) lookup plus a doubly linked list for O(1) recency reordering — touch a key and it jumps to the head; evict from the tail. Make the eviction rule an `EvictionPolicy` interface and LRU, LFU, FIFO, and TTL become interchangeable. For concurrency, one lock is correct but serial; stripe the locks by key segment, and know that production caches (Caffeine/W-TinyLFU) trade exact LRU for lock-free reads.
