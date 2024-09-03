---
title: Monorepo vs Microrepo - One Codebase or Many
description: Put every project in one repository, or give each its own. The trade runs through code sharing, dependency management, build times, and team autonomy — and both approaches scale, as Google and Amazon show.
date: 2024-09-03
draft: false
slug: /interview/repo
tags:
  - System Design
  - Interview Prep
  - Version Control
---

Every organisation decides how to lay out its code: everything in one giant repository, or split across many small ones. The split is real — Google, Meta, Uber, and Airbnb run monorepos; Amazon, Netflix, LinkedIn, and Oracle lean to many repos. Both scale to enormous size. The choice is a set of trade-offs, not a right answer.

## Monorepo

All projects, libraries, and services in a single repository. Each lives in its own directory with its own build config and an `OWNERS` file for permissions, but the history and the checkout are one.

- **Code sharing** — a shared utility is just an import; a change to it and all its callers can land in one atomic commit.
- **Dependencies** — everything is on one version of each third-party library, upgraded for all consumers at once.
- **Standards and discovery** — one place to enforce tooling, style, and review bar; every team's code is visible and searchable.
- **Tooling cost** — at scale, Git strains and builds balloon, so monorepos need specialised build systems (Bazel, Buck) and VCS tricks (sparse checkout, shallow clone).

## Microrepo

Each service or library in its own repository, with its own CI pipeline, permissions, and dependency versions.

- **Autonomy** — a team upgrades a library on its own schedule and picks its own stack.
- **Shared code** — published and versioned as packages, which is more ceremony than an import.
- **Standard tooling** — ordinary Git, Maven, npm; no custom build system required.
- **Divergence risk** — quality and practice can drift between repos without active governance.

## The trade, dimension by dimension

| | Monorepo | Microrepo |
| --- | --- | --- |
| Code sharing | import directly; atomic cross-project refactor | publish/version packages; more overhead |
| Dependencies | one version for all; big coordinated upgrades | per-team schedules; risk of version sprawl |
| Build/test | slow without "build only what changed" tooling | fast per service; hard cross-repo integration tests |
| VCS scale | strains Git; needs sparse/shallow strategies | simple per repo |
| Team autonomy | low; strong shared-codebase culture | high; strong per-repo ownership |
| Discoverability | high; all code visible | lower; needs docs and search tools |
| Large refactors | atomic and codebase-wide | coordinated across many repos |
| Governance | easy to standardise | standards can diverge |

## A concrete scaling story

Pinterest's monorepo *Pinboard* is 20 GB with 350,000 commits and 60,000 Git pulls a day. CI clones took **40 minutes**, because the build fetched all refs — over 2,500 branches — even with shallow clone. The fix was a one-line change adding a Git `refspec` to fetch only the `master` branch that build needed, cutting clone time by **99%, to 30 seconds**. The lesson is not "monorepos are slow" but "monorepos at scale demand build and VCS engineering that microrepos get for free".

## The decision is organisational, not technical

Both models work at the largest scale — the deciding factors are company size and culture, how tightly teams need to share code, whether services are coupled or independent, and whether you have the resources to build monorepo tooling. Startups often start with a monorepo for the easy sharing; large orgs either invest heavily in monorepo infrastructure (Google) or embrace microrepos to match a [microservices](/citadel/interview/cloud-native) structure (Amazon). The [CI/CD](/citadel/interview/ci-cd) setup follows directly from the choice.
