---
title: Password Salting - Why a Hash Alone Isn't Enough
description: Hashing a password is reversible in practice with a precomputed table. Adding a unique random salt per password makes those tables useless and forces an attacker to crack every hash individually.
date: 2024-06-09
draft: false
slug: /interview/salting
tags:
  - System Design
  - Interview Prep
  - Security
---

Storing passwords in plaintext is obviously wrong — anyone with database access reads them all. The fix seems to be hashing: run each password through a one-way function and store the result. But a bare hash is not enough either, because of **rainbow tables** — giant precomputed lookups of hash values for common passwords. Steal a database of unsalted hashes, match them against the table, and the weak passwords fall in seconds.

**Salting** closes that gap.

## What a salt is

Per OWASP: a salt is a unique, randomly generated string added to each password before hashing. It is **not secret** — it is stored in plaintext right next to the hash. Its value comes entirely from being *unique per password*.

## What it buys

- **Rainbow tables stop working.** Each password is hashed with its own salt, so a precomputed table would have to exist for every possible salt — computationally infeasible.
- **Identical passwords get different hashes.** If Alice and Bob both pick `password123`, then `hash(alice_salt + "password123")` and `hash(bob_salt + "password123")` are unrelated. An attacker cannot even tell that two users share a password.

## Storing a password

![Two flows. Store a password: a client-provided password is combined with a randomly generated salt, hash(password + salt) is computed, and both the salt and the hash are written to the database keyed by user id. Validate a password: the client-provided password is combined with the salt fetched from the database, hash(password + salt) is recomputed, and it is compared to the stored hash for equality.](../images/salting.png "Store: save the random salt and hash(password + salt). Validate: fetch the salt, recompute the hash the same way, compare. Source: system-design illustration.")

1. Generate a fresh cryptographically random salt for this password.
2. Combine it with the plaintext password.
3. Hash the combination with a strong, deliberately slow function — bcrypt, scrypt, or Argon2.
4. Store the salt (plaintext) and the resulting hash.

## Validating a login

1. The user submits username and password.
2. Fetch that user's salt from the database.
3. Combine the submitted password with the salt exactly as at signup, and hash it.
4. Compare to the stored hash. Equal means authenticated.

## Why this is harder to attack

Even with the whole database — every hash and every salt — an attacker cannot bulk-crack. Each password must be attacked on its own: take its salt, guess passwords, hash each guess with that salt, compare. The slow hash function makes every guess expensive, and the unique salt means work done against one account teaches nothing about any other.

## Salting is the floor, not the ceiling

Salting is non-negotiable, but it is one layer. It is paired with a slow hash (bcrypt/Argon2, so guessing costs milliseconds not nanoseconds) and sometimes a **pepper** — a secret value, held outside the database, mixed in as well, so a database leak alone is not enough. Note what salting is *not*: it is not [encryption or tokenization](/citadel/interview/encoding), which are reversible by design — a password hash is meant to be a dead end. It fits into the broader [identity management](/citadel/interview/identity-management) picture as the "something you know" factor done safely.
