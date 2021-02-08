---
title: Bloom Filters - Cheap Membership Tests That Never Say No Wrongly
description: A Bloom filter answers "have I seen this?" for a set too large to hold in memory, using a bit array and a few hash functions. It can be wrong in exactly one direction - a false positive - never a false negative, and that asymmetry is what makes it useful.
date: 2021-02-08
draft: false
slug: /data-structures/bloom-filters
tags:
  - Data Structures
  - Probabilistic Structures
  - Hashing
---

You're writing a web crawler. Before fetching a URL you want to know: have I crawled this one already? The set of crawled URLs runs to billions of entries. A hash set of the actual strings won't fit in memory, and a database round-trip per URL is too slow when you're processing thousands a second.

A **Bloom filter**, from Burton Howard Bloom's 1970 paper, gets you most of the way there in a few megabytes. It doesn't store the URLs. It can tell you *"definitely not seen"* with certainty, or *"probably seen"* with a tunable error rate. That one-sided error is the whole trick, and it's exactly the shape a crawler can tolerate.

## The one-sided guarantee

A Bloom filter answers a membership query with one of two results:

- **"Not in the set."** This is always correct. There are **no false negatives** — if the element had been added, the filter would know.
- **"Probably in the set."** This is *usually* correct but sometimes wrong: a **false positive**, where the filter reports an element it never saw. The rate is a parameter you choose.

So a "no" is proof, and a "yes" is a strong hint. Any use where a false "yes" costs you a wasted check but a false "no" would be a correctness bug is a candidate.

## How it works

The structure is a bit array of `m` bits, all `0`, and `k` independent hash functions, each mapping an element to an index in `[0, m)`.

**To add an element:** hash it with all `k` functions, and set the bit at each of the `k` resulting positions to `1`. Bits are shared — different elements often set overlapping positions, and that's fine.

**To query an element:** hash it with the same `k` functions and look at those `k` bits.

- If **any** bit is `0`, the element was never added — a "no". (Adding it would have set that bit.)
- If **all** `k` bits are `1`, report "probably yes". They might all be `1` because this element was added, or because *other* elements collectively set them. The second case is the false positive.

The hash functions need to be **fast** (they run on every add and query) and **uniformly distributed** (to spread bits evenly and keep the false-positive rate down). MurmurHash and xxHash are common choices; in practice you can derive `k` indices from just two hashes with `h1 + i·h2`.

## Sizing it

For `n` elements in `m` bits with `k` hash functions, the false-positive probability is approximately

$$p \approx \left(1 - e^{-kn/m}\right)^k$$

Given a target `p` and expected `n`, the standard results for the optimal parameters are

$$m = -\frac{n \ln p}{(\ln 2)^2}, \qquad k = \frac{m}{n}\ln 2$$

For a 1% false-positive rate that's about 9.6 bits per element and 7 hash functions — independent of how long the elements themselves are. Ten million URLs fit in roughly 12 MB.

```python
import math, hashlib

class BloomFilter:
    def __init__(self, n, false_positive_rate):
        self.m = math.ceil(-n * math.log(false_positive_rate) / (math.log(2) ** 2))
        self.k = max(1, round(self.m / n * math.log(2)))
        self.bits = bytearray((self.m + 7) // 8)

    def _positions(self, item):
        data = item.encode() if isinstance(item, str) else bytes(item)
        h1 = int.from_bytes(hashlib.sha256(data).digest()[:8], "big")
        h2 = int.from_bytes(hashlib.md5(data).digest()[:8], "big") or 1
        for i in range(self.k):                 # derive k indices from two hashes
            yield (h1 + i * h2) % self.m

    def add(self, item):
        for pos in self._positions(item):
            self.bits[pos // 8] |= 1 << (pos % 8)

    def __contains__(self, item):
        return all(self.bits[pos // 8] >> (pos % 8) & 1 for pos in self._positions(item))


bf = BloomFilter(n=100_000, false_positive_rate=0.01)
for i in range(100_000):
    bf.add(f"item-{i}")

assert all(f"item-{i}" in bf for i in range(100_000))     # no false negatives, ever
misses = sum(1 for i in range(100_000, 200_000) if f"item-{i}" in bf)
print(misses / 100_000)                                   # ~0.01, the chosen rate
```

## Two uses

**Deduplicating a crawl frontier.** Keep a Bloom filter of every URL crawled. On discovering a URL, query the filter. "Definitely not seen" → crawl it and add it. "Probably seen" → skip it (accepting that a real false positive occasionally drops a genuinely new page), or, if missing a page is expensive, fall back to the authoritative check only for this small slice. Either way the slow store is spared the overwhelming majority of lookups. This is the core of [deduplicating URLs at scale](/citadel/system-design/dedupe-urls).

**Blocking cache-penetration.** *Cache penetration* is a flood of requests for keys that exist in neither the cache nor the database — each one misses the cache, hits the database, and finds nothing, and enough of them overwhelm the database. Populate a Bloom filter with every valid key. Check it before touching the cache: "definitely not a valid key" → return "not found" immediately; "probably valid" → proceed to cache and database as normal. The database only ever sees requests for keys that plausibly exist. See [caching pitfalls](/citadel/interview/cache-pitfalls).

## The catch: no deletion

A standard Bloom filter can't remove an element. Clearing a bit could break some *other* element that also hashed to it, reintroducing a false negative — the one error the structure promises never to make. A **counting Bloom filter** replaces each bit with a small counter (increment on add, decrement on delete) to support removal, at several times the space.

## When to reach for it

A Bloom filter earns its place when the set is large, queries must be fast, a bounded false-positive rate is acceptable, false negatives are not, and an exact structure would cost too much memory. When you also need approximate *counts* rather than just membership, the neighbouring structures are HyperLogLog and Count-Min Sketch, both in the [advanced data structures](/citadel/algorithms/advanced) roundup.

## The one idea to keep

The Bloom filter trades exactness for space, and it does so in exactly one direction: a "no" is a proof, a "yes" is a probability. That asymmetry is not a limitation to work around — it is the design. Whenever a wrong "yes" only costs a redundant check while a wrong "no" would be a correctness bug (crawl dedup, cache-penetration guards, "might this key be on disk?"), the filter is a few megabytes standing in for a structure that would need gigabytes. Where the shape is reversed — a wrong "yes" is unacceptable — do not reach for it.
