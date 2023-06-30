---
title: Secure Coding - Building It Right Instead of Patching It Later
description: Most vulnerabilities are a handful of mistakes repeated. Validate input at the boundary, encode output for its destination, fail closed, run with least privilege, and never trust data as code. The habits and the tooling that catch the rest before it ships.
date: 2023-06-30
draft: false
slug: /cyber-security/secure-coding
tags:
  - Security
  - Blue Team
  - Software Engineering
---

The [web application security](/citadel/cyber-security/web-app-security) post catalogues the vulnerability classes; this one is the developer's side — the coding habits that keep them out in the first place, because finding a bug in review or a scanner is cheaper than finding it in production and far cheaper than an attacker finding it.

The good news is that the mistakes are repetitive. A small set of principles, applied consistently, prevents most of them, and a small set of tools catches much of the rest.

## Principles

- **Validate input at the trust boundary.** The moment untrusted data enters your system (an HTTP request, a queue message, a file upload, a parameter from another service), check it against what you expect — type, length, range, format, allowed set. Prefer an **allowlist** (accept only what matches a known-good pattern) over a **denylist** (reject known-bad, and miss the rest). Validation reduces attack surface; it is *not* the fix for injection (that's parameterization) but it stops a lot of nonsense earlier.
- **Encode output for its sink.** Data that's safe in a database is dangerous in HTML; safe in HTML is dangerous in a shell. Encode/escape at the point of use, contextually: HTML-encode into HTML, JS-encode into script, shell-quote... actually, never build a shell string — pass an argument array. The bug is treating data as code at the boundary where the interpreter changes.
- **Fail closed.** On an error or an unexpected condition, deny. An access-control check that throws should result in "denied," not "allowed because the check didn't complete." Default to the restrictive branch.
- **Least privilege.** Every component runs with the minimum rights it needs — the service account, the database user (a read-only connection for read paths), the container capabilities, the API token's scopes, the IAM role. So a compromise is contained.
- **Complete mediation.** Check authorization on *every* access to a protected resource, at the server, every time — not once at login, not only in the UI. Centralize the check so no endpoint forgets it.
- **Defence in depth.** Assume each layer will sometimes fail; add another. Parameterized queries *and* least-privilege DB user *and* input validation *and* monitoring.
- **Keep it simple.** Complexity hides bugs. The smallest design that meets the requirement has the smallest attack surface.

## By vulnerability class

- **Injection** — parameterized queries / prepared statements for SQL; ORM query builders (but watch raw fragments); argument arrays for `exec`; parameterized APIs for LDAP/XPath; strict schema validation for anything parsed.
- **XSS** — a templating engine with contextual auto-escaping on by default; treat any place you build HTML by string concatenation as suspect; a **Content Security Policy** that forbids inline script and restricts sources; `HttpOnly` and `Secure` on session cookies; sanitize rich-text input with a vetted library (DOMPurify), never a regex.
- **Access control / IDOR** — scope every data lookup to the authenticated principal (`WHERE owner_id = :me`); never let a client-supplied ID imply permission; a policy layer every request passes through.
- **Deserialization** — don't deserialize untrusted input. Use JSON with a strict schema; if a native format is unavoidable, sign the payload and verify before deserializing, and restrict allowed types to a tight allowlist.
- **Prototype/mass-assignment pollution** — reject the keys `__proto__`, `constructor`, `prototype` in any recursive merge; bind request bodies to an explicit DTO with only the fields you mean to accept, not straight onto your model.
- **SSRF** — an allowlist of destinations; resolve the hostname and block link-local/private ranges *after* resolution; disable redirects; no ambient cloud credentials on the fetcher.
- **Path traversal** — canonicalize the path and verify it's still inside the intended directory; better, don't take filenames from users at all — use an ID that maps to a path server-side.
- **TOCTOU / race conditions** — the check and the use must be atomic. Open the file and operate on the descriptor rather than re-resolving the path; use `O_CREAT|O_EXCL` for temp files; do the "is it available?" check and the "reserve it" write as one conditional database update (`UPDATE ... WHERE status='free'`), not a read then a write. Stage uploads outside the web root and `rename()` atomically into place.

## Secrets and configuration

- No secrets in source, in config files committed to git, in logs, or in error messages. Use a secrets manager / vault; inject at runtime; rotate.
- Safe defaults: TLS on, debug off, verbose errors off, directory listing off, permissive CORS off. Ship the hardened config, don't rely on ops to apply it.
- Secure the config *itself* — file permissions, encrypted at rest, access-controlled.

## Cryptography

Don't implement it. Use a high-level library (libsodium, `cryptography` for Python, Tink) that gives you AEAD (AES-GCM, ChaCha20-Poly1305), a real password hash (Argon2id / scrypt / bcrypt with a proper cost), and a CSPRNG (`secrets`, `crypto.randomBytes`, `/dev/urandom` — never `Math.random`, `rand()`, or a language's default non-crypto RNG). Nonces: unique per key, and for GCM never reused. Keys in a KMS/HSM. See [applied cryptanalysis](/citadel/cyber-security/applied-cryptanalysis) for what happens when these are wrong.

## Dependencies

Most of your code is dependencies. Lockfiles with hashes; **SCA** scanning (`Dependabot`, `Snyk`, `Trivy`, `osv-scanner`) in CI failing the build on known-vulnerable versions; an SBOM; a patch cadence; and removing packages you don't need (each is attack surface and a supply-chain risk — typosquatting, dependency confusion, a maintainer account takeover).

## Process and tooling

- **SAST** (`Semgrep`, CodeQL, SonarQube) — scans source for taint flows and dangerous patterns. Fast, noisy; tune the ruleset; run on every PR.
- **DAST** (`ZAP`, `Burp` scans) — attacks the running app. Finds real exploitable issues; slower; run against staging.
- **SCA** — dependency vulnerabilities (above).
- **Secret scanning** (`gitleaks`, `trufflehog`) — pre-commit hook *and* CI, plus history scanning.
- **Fuzzing** — for parsers and anything handling untrusted bytes; see [fuzzing](/citadel/pentesting/fuzzing).
- **Threat modelling** — during design, walk the data flows and ask what could go wrong at each (STRIDE: spoofing, tampering, repudiation, info disclosure, denial of service, elevation of privilege). Write **abuse cases** alongside user stories.
- **Code review with a security lens** — a checklist for the reviewer: any new input? any new query, template, file path, deserialization, external call? any auth check? any secret?
- **Security regression tests** — when a vulnerability is fixed, add a test that fails on the vulnerable code, so it can't come back.

## The one idea to keep

Most vulnerabilities are the same few mistakes repeated, so a short list of habits prevents the bulk of them: validate input at the boundary (allowlist), encode output for its exact destination, fail closed, run every component at least privilege, check authorization on every access, and never let data become code — parameterize queries, don't deserialize untrusted input, don't build shell strings. Then let SAST, SCA, secret scanning, and fuzzing in CI catch the ones that slip through, and add a regression test for every bug you fix.
