---
title: Linked List - Pointers, Dummies, and Two Speeds
description: A guide to the Linked List section of NeetCode 150 — three idioms cover almost everything, namely the three-pointer reversal, the dummy head, and fast/slow pointers for the middle, the k-th from end, and cycles.
date: 2021-08-06
draft: false
slug: /dsa/linked-list
tags:
  - Competitive Programming
  - Linked List
  - Two Pointers
---

Linked-list problems are pointer-manipulation exercises. Three idioms do most of the work, and once they are reflexes the eleven problems in this section are variations and compositions of them.

## The three idioms

**The three-pointer reversal.** `prev`, `curr`, `nxt`: save `curr.next`, redirect `curr.next` to `prev`, slide all three forward. This is [Reverse a Linked List](/citadel/dsa/reverse-linked-list), and a subroutine inside [Reorder List](/citadel/dsa/reorder-list) and [Reverse Nodes in K-Group](/citadel/dsa/reverse-nodes-in-k-group).

**The dummy head.** Build any list from a sentinel node and return `dummy.next`. The "is this the first node?" special case disappears. Used in [Merge Two Sorted Lists](/citadel/dsa/merge-two-sorted-lists), [Add Two Numbers](/citadel/dsa/add-two-numbers), [Remove Nth Node From End of List](/citadel/dsa/remove-nth-node-from-end-of-list), [Merge K Sorted Lists](/citadel/dsa/merge-k-sorted-lists).

**Fast and slow pointers.** Two pointers at different speeds (or a fixed gap):

- Gap of `n` → [Remove Nth Node From End of List](/citadel/dsa/remove-nth-node-from-end-of-list): when the leader ends, the follower is `n` from the end.
- Speed 1 vs 2 → the middle node (used inside [Reorder List](/citadel/dsa/reorder-list)).
- Speed 1 vs 2 → cycle detection ([Linked List Cycle](/citadel/dsa/linked-list-cycle)); reset one pointer to the head and step together → the cycle entrance ([Find the Duplicate Number](/citadel/dsa/find-the-duplicate-number)).

## The rest

- [Copy List with Random Pointer](/citadel/dsa/copy-list-with-random-pointer) — a graph clone: allocate all nodes, then wire all pointers, with a hash map (or an interleaving trick) linking old to new.
- [LRU Cache](/citadel/dsa/lru-cache) — hash map plus doubly linked list, so both "lookup by key" and "reorder by recency" are $O(1)$.

## Recognising the tools

- Need to walk from the end, or find the middle, without a length count → **fast/slow**.
- Building or splicing a new list → **dummy head**.
- Reversing all or part of a list → **three-pointer loop**.
- Detecting or entering a cycle → **Floyd's**.
- Copying a structure with arbitrary internal pointers → **map old node to new node**.

## Where this goes next

Fast/slow pointers reappear in array problems recast as functional graphs ([Find the Duplicate Number](/citadel/dsa/find-the-duplicate-number)). The recursive shape of "process this node, recurse on the rest" is the bridge to [Trees](/citadel/dsa/trees), where the same recursion just branches two ways.
