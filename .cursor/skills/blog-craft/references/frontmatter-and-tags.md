# Frontmatter and tags

The repo is consumed as the `content/blogs` git submodule of the site
(`main-site/`, a Next.js app — a checkout lives at `main-site/` inside this repo,
gitignored). See **How the site renders a post** at the bottom before changing
structure.

The loader (`main-site/lib/content/blog.ts`) reads `title`, `date`, `tags`,
`description`, `draft`, and `slug`. **The route is derived from `slug`**:
`slug: /physics/vectors` → `/citadel/physics/vectors`. So `slug` is
load-bearing — never change it — and **filenames match the slug tail**
(`physics/vectors.md` ↔ `slug: /physics/vectors`).

## Schema

```yaml
---
title: Concept - the angle on it
description: >-
  One to three sentences. What the post covers and why it's worth reading.
  A standalone blurb (used as the card text and meta description).
date: 2023-01-07          # original publish date — never change
draft: false              # false only once the post passes the checklist
slug: /maths/vectors            # /<category>/<name> — never change
tags:
  - Mathematics           # primary field tag, Title Case, first
  - Vector Algebra        # 1–3 topic tags
---
```

Every post has these six keys, in this order. No extra keys (no `author`, no `updated`, no `cover` unless the site later calls for it — ask first).

## `title`

Format: **`Concept - the angle`**, one separator, spaced hyphen (` - `).

- Concrete noun first. The subject, then the framing.
- Aim for ≤ 70 characters; hard cap ~75.
- No trailing punctuation. **Strip stray quotes** — `algorithms/Algorithms.md` currently ends with a `"`.
- No "A Deep Dive into", "Everything You Need to Know About", "The Ultimate Guide to", "Demystifying", "Understanding" as the lead word.
- One playful clause is allowed in the angle half if it still says what the post is. Not a pun that hides the subject.

Good:

- `Bloom Filters - Cheap Membership Tests at Web Scale`
- `Kepler's Laws - Why Orbits Are Ellipses`
- `Two-Factor Authentication - How TOTP Codes Actually Work`

Rework:

- `Cosmic Choreography - From Kepler's Waltz to Einstein's Spacetime Symphony!` → `Astrodynamics - From Kepler's Ellipses to Einstein's Curved Spacetime`
- `Atoms Unmasked - From Billiard Balls to Quantum Chaos Parties!` → `Atomic Models - From Billiard Balls to the Electron Cloud`

## `description`

- 1–3 sentences, roughly **25–50 words**.
- Says what's covered **and** why a reader would want it. Concrete nouns.
- Written as a blurb about the post, not an announcement: no "This post delves into", "In this article we", "Join me as we explore", "Ever wondered".
- No emoji. Ends with a period.
- The current descriptions are mostly serviceable but wordy and "delve"-heavy; tighten rather than replace wholesale.

Before:

> Explore the fundamental world of vectors, the mathematical objects that describe quantities with both magnitude and direction. This post delves into coordinate systems, vector algebra, dot and cross products, and their applications in geometry and physics, providing a comprehensive guide to their properties and operations.

After:

> Vectors carry both size and direction, which is why they describe force, velocity, and displacement where plain numbers can't. A tour of the algebra: components, dot and cross products, triple products, and what each one means geometrically.

## `date`

The original publish date. Never change it during migration. There is no `updated` key; don't add one without asking.

## `draft`

`draft: true` on 38 posts today. The site now honours it: draft posts are hidden from listings, search, the feed, the sitemap, and 404 on their route **in production**, but stay visible when running `next dev` (so you can preview a migration before publishing). Flip to `false` only when the post passes the per-post checklist in `migration-workflow.md`. If a draft is genuinely unfinished (missing sections, `TODO`s in the body), migrate the prose that exists, leave `draft: true`, and tell the user what's missing.

## `slug`

**Load-bearing.** Format: `slug: /<category>/<name>`, matching the file path
(`category/name.md`). Served by the `/citadel/[...slug]` catch-all — e.g.
`slug: /physics/rotations` lives at `/citadel/physics/rotations`. Never change
an existing post's `slug`; it is what inbound links point at.

- Slug tail **is** the filename stem: `physics/rotations.md` → `/physics/rotations`.
- New split-part files: `slug: /<category>/<name>` with `<name>` a clean kebab-case stem (it *is* the URL now). Confirm it's unique repo-wide: `grep -rl "slug: /<category>/<name>" .`.
- Renaming a file means changing its slug too (and breaking inbound links) — only do that when asked.

## `tags`

**2–4 tags. Title Case. Primary field tag first**, then 1–3 topic tags.

Current tags are inconsistent (`Mathematics` vs `system-design` vs `SDE` vs `DSA`). Normalise to this map.

| Directory | Primary tag | Typical topic tags |
| --- | --- | --- |
| `algorithms/` | `Algorithms` | `Divide and Conquer`, `Dynamic Programming`, `Graphs`, `Greedy`, `Backtracking`, `Complexity` |
| `artificial-intelligence/` | `Artificial Intelligence` | `Machine Learning`, `Deep Learning`, `NLP`, `Search`, `Probabilistic Models` |
| `automata/` | `Theory of Computation` | `Automata`, `Formal Languages`, `Computability` |
| `big-data/` | `Big Data` | `Distributed Systems`, `Data Engineering`, `Hadoop` |
| `coding-languages/` | `Programming Languages` | `<language name>`, `Type Systems`, `Web` |
| `compiler-design/` | `Compilers` | `Parsing`, `Code Generation`, `Optimization`, `Static Analysis` |
| `computer-architecture/` | `Computer Architecture` | `CPU`, `Memory`, `Pipelining`, `Parallelism` |
| `computer-graphics/` | `Computer Graphics` | `Rendering`, `Image Processing`, `Compression`, `Multimedia` |
| `computer-networks/` | `Networking` | `TCP/IP`, `Routing`, `Wireless`, `Network Security` |
| `cp/` | `Competitive Programming` | `Algorithms`, `Problem Solving` |
| `cyber-security/` | `Security` | `Cryptography`, `Public-Key`, `Digital Signatures`, `Network Security` |
| `data-structures/` | `Data Structures` | `Trees`, `Hashing`, `Probabilistic Structures`, `Balanced Trees` |
| `dbms/` | `Databases` | `Relational Model`, `Transactions`, `Query Languages`, `Distributed Databases` |
| `interview/` | `System Design` | `Interview Prep`, plus the real topic (`Caching`, `Networking`, `Auth`, ...) |
| `maths/` | `Mathematics` | `Calculus`, `Linear Algebra`, `Probability`, `Number Theory`, `Geometry`, `Algebra` |
| `miscs/` | (topic-led, no fixed primary) | `Distributed Systems`, `Game Theory`, `Information Theory`, `Randomized Algorithms`, `Mobile Computing` |
| `operating-system/` | `Operating Systems` | `Processes`, `Memory Management`, `Scheduling`, `File Systems` |
| `parallel-computing/` | `Parallel Computing` | `Parallel Algorithms`, `Architecture` |
| `pentesting/` | `Security` | `Offensive Security`, `Exploitation`, `Tooling` |
| `physics/` | `Physics` | `Mechanics`, `Electromagnetism`, `Quantum Mechanics`, `Thermodynamics`, `Astrophysics`, `Optics`, `Waves` |
| `random-thoughts/` | `Notes` | the real topic (`Databases`, `Payments`, `Latency`) |
| `system-design/` | `System Design` | the real topic (`Databases`, `Messaging`, `Payments`, `Case Study`) |
| `tech/` | `Tools` | the real topic (`Containers`, `Databases`, `CI/CD`, `Linux`) |

Rules:

- Drop `SDE`, `DSA`, `Cheatsheet` as tags — encode "cheatsheet" by keeping the post reference-style, not by tagging it.
- Never use the lowercase directory name as a tag (`system-design` → `System Design`).
- A tag should be something a reader might plausibly click to find sibling posts. If a tag would only ever apply to this one post, it's a bad tag.
- The site has **no category browsing** — tags are the only taxonomy. The primary field tag (`Physics`, `Mathematics`, …) is what does the work of a category, and `getRelatedPosts` / search rank on tag overlap. Consistent tags matter more here than they would on a category-first site.

---

## How the site renders a post

`main-site/` (Next.js 16, `next-mdx-remote`). What this means for content:

- **Routing is by `slug`.** URL = `slug` under `/citadel`, served by `app/citadel/[...slug]/` (a catch-all). `slug: /physics/rotations` → `/citadel/physics/rotations`. Changing a `slug` changes the URL; that's why slugs are frozen.
- **Slugs must be unique repo-wide** (the catch-all resolves the first post whose slug matches). Format is always `/<category>/<filename-stem>`, so uniqueness follows the file tree.
- **A cross-link target's path is its slug.** `[rolling](/citadel/physics/rotations)` for `slug: /physics/rotations`.
- **Content is MDX**, not plain Markdown. Bare `<`, `>`, `{`, `}` in prose can break compilation. Keep every inequality and brace inside `$…$`/`$$…$$` or a backtick span (`` `x < 5` ``). Fenced code blocks are safe.
- **Math**: `remark-math` + `rehype-katex`. `$…$` inline, `$$…$$` as its own line for display. This already works; don't change delimiters.
- **Code**: `rehype-pretty-code` (Shiki, `github-dark`). Always language-tag fences.
- **Headings**: `rehype-slug` + `rehype-autolink-headings` turn `##`/`###` text into anchor IDs and a table of contents. Clean heading text = clean anchors — another reason to kill the mojibake/pictographs.
- **Images**: rendered through `next/image` from the raw `src`. Existing posts use `../images/<file>.png` (resolves to `content/blogs/images/`). Keep that path shape; always give real alt text.
- **Reading time**: computed from raw content (LaTeX included), so length edits are reflected automatically; no field to maintain.
- **Inter-post links**: write `[text](/citadel/<category>/<name>)` matching the target's `slug: /<category>/<name>`. A shim in `MarkdownRenderer.tsx` also rewrites legacy `/pensieve/<path>` → `/citadel/<path>` and routes internal links through `next/link`.
- **`draft`**: honoured (hidden in prod, visible in `next dev`) as of the main-site changes on branch `blog-content-compat`.

If a content need isn't served by the current site, improving `main-site/` is in scope — it's the same author's repo. Keep such changes small, typed, and idiomatic to that codebase.
