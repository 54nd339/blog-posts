---
title: Design Twitter - Merge K Timelines With a Heap
description: A mini social feed where getNewsFeed returns the 10 most recent tweets from a user and everyone they follow — each user's tweets are already newest-first, so it is a k-way merge.
date: 2021-11-15
draft: false
slug: /dsa/design-twitter
tags:
  - Competitive Programming
  - Heap
  - Design
---

Each user's own tweets are stored newest-first. The feed is the 10 most recent across the followee set — exactly [Merge K Sorted Lists](/citadel/dsa/merge-k-sorted-lists), where the "lists" are per-user tweet histories and a global timestamp is the sort key.

## Description

Implement `Twitter` with:

- `postTweet(userId, tweetId)` — post a tweet.
- `getNewsFeed(userId)` — the 10 most recent tweet ids from `userId` and their followees, newest first.
- `follow(followerId, followeeId)` / `unfollow(followerId, followeeId)`.

**Example**

```
postTweet(1, 5); getNewsFeed(1) -> [5]
follow(1, 2); postTweet(2, 6); getNewsFeed(1) -> [6, 5]
unfollow(1, 2); getNewsFeed(1) -> [5]
```

**Constraints**

- $1 \le \text{userId}, \text{tweetId} \le 10^4$; at most `3 \times 10^4` calls.
- A user cannot follow themselves; the feed always includes the user's own tweets.

## Prerequisites

- Hash maps of lists / sets.
- Min-heap `k`-way merge ([Merge K Sorted Lists](/citadel/dsa/merge-k-sorted-lists)).

## Approach 1: Sorting

### Intuition

Store each tweet with a global increasing timestamp. For the feed, gather every relevant tweet, sort by timestamp descending, take 10.

### Algorithm

1. `time` counter; `tweets: user -> list of (time, tweetId)`; `following: user -> set`.
2. `postTweet`: append `(time, tweetId)`; `time += 1`.
3. `getNewsFeed`: collect tweets from `userId` and every followee; sort by `time` descending; return the first 10 ids.
4. `follow` / `unfollow`: add/discard in the set.

```python
from collections import defaultdict

class Twitter:
    def __init__(self):
        self.time = 0
        self.tweets = defaultdict(list)
        self.following = defaultdict(set)

    def postTweet(self, userId: int, tweetId: int) -> None:
        self.tweets[userId].append((self.time, tweetId))
        self.time += 1

    def getNewsFeed(self, userId: int) -> list[int]:
        users = self.following[userId] | {userId}
        all_tweets = [t for u in users for t in self.tweets[u]]
        all_tweets.sort(reverse=True)
        return [tid for _, tid in all_tweets[:10]]

    def follow(self, followerId: int, followeeId: int) -> None:
        self.following[followerId].add(followeeId)

    def unfollow(self, followerId: int, followeeId: int) -> None:
        self.following[followerId].discard(followeeId)
```

### Complexity

- **Time:** $O(N \cdot m + t \log t)$ per `getNewsFeed` (`N` followees, `m` tweets each, `t` total); $O(1)$ for the rest.
- **Space:** $O(\text{total tweets} + \text{follow edges})$.

## Approach 2: Heap (k-way merge)

### Intuition

Do not sort everything. Each user's tweet list is already newest-last, so its newest tweet is at the end. Push the newest tweet of each followee into a max-heap keyed by timestamp; pop the newest, then push that user's next-newest. Stop after 10.

### Algorithm

1. Same maps, but `postTweet` uses a *decreasing* counter so a min-heap yields newest first (or negate the timestamp).
2. `getNewsFeed`: add `userId` to the followee set. For each followee with tweets, push `(timestamp, tweetId, userIndex, listIndex)` for its last tweet.
3. Pop up to 10 times: record the id, and if that user has an earlier tweet, push it.

```python
import heapq
from collections import defaultdict

class Twitter:
    def __init__(self):
        self.time = 0
        self.tweets = defaultdict(list)          # user -> [(time, tweetId), ...] oldest first
        self.following = defaultdict(set)

    def postTweet(self, userId: int, tweetId: int) -> None:
        self.time += 1
        self.tweets[userId].append((self.time, tweetId))

    def getNewsFeed(self, userId: int) -> list[int]:
        heap = []
        users = self.following[userId] | {userId}
        for u in users:
            if self.tweets[u]:
                i = len(self.tweets[u]) - 1
                t, tid = self.tweets[u][i]
                heapq.heappush(heap, (-t, tid, u, i - 1))
        res = []
        while heap and len(res) < 10:
            negt, tid, u, i = heapq.heappop(heap)
            res.append(tid)
            if i >= 0:
                t2, tid2 = self.tweets[u][i]
                heapq.heappush(heap, (-t2, tid2, u, i - 1))
        return res

    def follow(self, followerId: int, followeeId: int) -> None:
        self.following[followerId].add(followeeId)

    def unfollow(self, followerId: int, followeeId: int) -> None:
        self.following[followerId].discard(followeeId)
```

### Complexity

- **Time:** $O(N \log N)$ per `getNewsFeed` (seed `N` followees, then $\le 10$ pop/push).
- **Space:** $O(N)$ for the heap.

## Approach 3: Heap (Optimal, cap per user)

### Intuition

Two bounds keep it near-constant: store at most the 10 newest tweets per user (older ones can never enter any feed), and if a user follows more than 10 people, first pick the 10 followees whose latest tweet is newest (a max-heap), then merge only those.

### Algorithm

1. `postTweet`: append, then trim the user's list to its last 10 entries.
2. `getNewsFeed`: if `len(followees) > 10`, use a max-heap on each followee's latest timestamp to keep the top 10 followees; otherwise take all. Then run the k-way merge of Approach 2 over those (at most 11) lists.

```python
import heapq
from collections import defaultdict

class Twitter:
    def __init__(self):
        self.time = 0
        self.tweets = defaultdict(list)
        self.following = defaultdict(set)

    def postTweet(self, userId: int, tweetId: int) -> None:
        self.time += 1
        self.tweets[userId].append((self.time, tweetId))
        if len(self.tweets[userId]) > 10:
            self.tweets[userId].pop(0)

    def getNewsFeed(self, userId: int) -> list[int]:
        users = list(self.following[userId] | {userId})
        users = [u for u in users if self.tweets[u]]
        if len(users) > 10:
            users = heapq.nlargest(10, users, key=lambda u: self.tweets[u][-1][0])

        heap = []
        for u in users:
            i = len(self.tweets[u]) - 1
            t, tid = self.tweets[u][i]
            heapq.heappush(heap, (-t, tid, u, i - 1))

        res = []
        while heap and len(res) < 10:
            negt, tid, u, i = heapq.heappop(heap)
            res.append(tid)
            if i >= 0:
                t2, tid2 = self.tweets[u][i]
                heapq.heappush(heap, (-t2, tid2, u, i - 1))
        return res

    def follow(self, followerId: int, followeeId: int) -> None:
        self.following[followerId].add(followeeId)

    def unfollow(self, followerId: int, followeeId: int) -> None:
        self.following[followerId].discard(followeeId)
```

### Complexity

- **Time:** $O(N)$ per `getNewsFeed` with the caps; $O(1)$ for the rest.
- **Space:** $O(\text{users} \cdot 10 + \text{edges})$.

## Common Pitfalls

- **Forgetting the user follows themselves for the feed.** Add `userId` to the followee set inside `getNewsFeed`, not in `follow` (the problem forbids self-follow).
- **`follow` when already following, or `unfollow` a non-followee.** A `set` with `add` / `discard` makes both idempotent.
- **Heap direction.** `heapq` is a min-heap; negate the timestamp (or use a decreasing counter) so the newest tweet pops first.
- **Not advancing within the same user's list.** After popping a user's tweet, push their *next older* tweet so the merge stays correct.

## The keystone

A feed over many sorted-by-time sources is a k-way merge. Cap each source (only recent items matter) and cap the number of sources (a heap picks the most promising), and a potentially huge merge becomes $O(1)$-ish per query.
