# Migration workflow

Goal: bring every category to the house style in `voice-and-style.md` and `post-structure.md`, one category at a time, producing focused top-quality posts while conserving all of the category's information.

## Principles

- **Conserve at the category level.** The invariant is: after migrating a category, the union of its posts still contains every technical claim, derivation, number, edge case, and worked detail it held before — no matter how posts were split, refocused, or had content moved between them. Track this explicitly with a **content map** (below).
- **Per post, wide latitude.** A post may be narrowed to one topic done excellently instead of a sprawling survey. It may be split into a focused series. It may be re-structured and re-voiced freely. Content a post sheds must land in a sibling in the same category (existing or a new split-part).
- **Reshape the maths for quality.** Simplify derivations, reassign/rename variables (consistently), restructure steps, tighten notation, generalise a result with the concrete case kept as an instance. *Cleaner, not shallower* — never skip a step or lose rigor; results and meaning stay identical.
- **Never trade depth for length.** Trimming is hype, throat-clearing, and repetition only. If a topic won't fit one good page, **split it**, don't thin it.
- **Adding is allowed.** Textbook-standard values, landmark results with dates, documented attributed quotes — when they make the analysis more concrete. Never fabricate.
- **The lede is always rebuilt from scratch.** It is the weakest part of every current post.
- **Interlink deliberately.** Wherever a post leans on, continues, or is continued by another (a split sibling, a prerequisite, a deeper treatment), link it with `[text](/citadel/<target's slug tail>)`. Split series get an explicit "part of" line. A reader should never hit a named concept that has its own post without a way to get there.
- **One family, one voice.** Use the matching template in `post-structure.md`.
- **Checkpoint with the user.** Present the category's content map and split plan before executing it; show the first 1–2 restyled posts and confirm direction before doing the rest.

## Category order

Work top to bottom. Rationale: start small to lock the voice and get sign-off, bank the big visible wins next, leave the longest and most math-dense for when the style is proven.

1. `random-thoughts/` (3) — tiny, sets the voice, near-zero risk.
2. `system-design/` (34) — short posts, the "welcome back" formula is thickest here; biggest visible improvement per hour.
3. `tech/` (16) — short, tool-focused.
4. `interview/` (61) — largest count, but short and templated; builds momentum.
5. `data-structures/` (8) then `algorithms/` (21) — code-heavy, medium length.
6. `automata/` (3), `compiler-design/` (8), `parallel-computing/` (3) — small theory.
7. `operating-system/` (9), `computer-architecture/` (9), `computer-networks/` (8), `dbms/` (9) — medium theory.
8. `artificial-intelligence/` (23), `big-data/` (5), `computer-graphics/` (9).
9. `cyber-security/` (10), `pentesting/` (5).
10. `coding-languages/` (10) — all currently `draft: true`.
11. `miscs/` (10) — mixed bag, tag per topic.
12. `maths/` (25) — math-dense, long; style must be solid first.
13. `physics/` (22) — longest (up to 10k words), most hype to strip, several need **splitting** into series. Do last.

The user may override the order. `MIGRATION.md` is the source of truth for progress.

## Refocusing and splitting

A category may leave migration with more posts than it started with, structured differently — as long as nothing is lost when you look at the category as a whole.

### The content map

Before restructuring a category, inventory it:

1. List every post and, under each, its sections / load-bearing content (claims, derivations, key numbers, worked examples, figures).
2. Decide the target shape: which posts stay single-topic as-is, which get refocused, which split, and what new files appear. Name new files and give each a slug `/<category>/<name>`.
3. Write the map as **old location → new location** for every chunk. Every source section must appear on the right-hand side somewhere. This table is the conservation proof.
4. Put the map in `MIGRATION.md` under the category's row (or a linked file for a big category). Keep it updated as you execute.

### Rules

- **Existing `slug`s are immutable** — the route is the slug and links point at it. One part of a split keeps the original file *and its slug*; other parts are new files with new slugs.
- **A new slug must be unique repo-wide.** Check with `grep -rn "slug: /" --include=*.md . | grep <name>` before creating. Format is `/<category>/<name>` matching the file path.
- Inter-post links use `[text](/citadel/<target's slug tail>)` — check the target's `slug:` line; it is often not the filename. See `frontmatter-and-tags.md` → "How the site renders a post".
- A refocused post that keeps its filename must still be substantial and genuinely about what its name/slug says. Don't hollow out a URL.
- Content moved between two existing posts is fine; record it in the map both as removed and as added.
- New split-part posts get full frontmatter per `frontmatter-and-tags.md`, `draft` matching the readiness of their content, and cross-links to their siblings (a short "This is part N of a series on X" line, or inline `[links]`).
- Prefer splitting along the phenomena / sub-topics the post already implies (its `##` sections), not arbitrary word-count halves.
- After executing, re-check the map: open each new/edited file and tick off every right-hand-side entry.

## Per-post procedure

1. **Read the whole post.** Identify: family (A–E in `post-structure.md`), current word count (`wc -w`), `draft` status, and where it sits in the category content map (stay / refocus / split).
2. **Inventory what must survive.** Jot the load-bearing claims, equations, numbers, derivation steps, edge cases, code blocks, citations. This list is your conservation contract — every item lands somewhere in the category (this post or a named sibling).
3. **Rebuild the lede** using the openings menu. Hook (≤ 25 words) → context → one-line promise. No heading.
4. **Walk the sections:**
   - Reorder to intuition-first where a section opens cold on a definition or equation.
   - Rewrite section titles to plain noun phrases or questions (kill "...on Steroids", "SHOCKER!", trailing `!`).
   - De-hype: strip exclamation marks (budget ~1 per 1000 words), hype interjections, "buckle up", emoji.
   - Tighten: split screen-height paragraphs, cut throat-clearing, de-bullet prose that should flow, reduce bold to first-definition-only.
   - Gloss every term at first use.
   - Keep every equation, number, and code block; motivate each before it appears.
5. **Fix the ending** — replace "Conclusion"/"Key Takeaways" filler with a real close (see `post-structure.md` → Endings). Keep a `## Key takeaways` bullet list only for reference-style posts.
6. **Fix frontmatter** per `frontmatter-and-tags.md`: title format, description tightening, tag normalisation. Leave `date` and `slug` untouched.
7. **Mechanical scan:**
   - `grep -nED 'welcome back|dive in|buckle up|delve|In conclusion|game-changer' <file>`
   - non-ASCII / mojibake: `grep -nP '[^\x00-\x7F]' <file>` then check each hit is legitimate (a real accented name, a real math symbol in prose) and not corruption like `классика`.
   - images have alt text, not the filename.
   - math still renders (balanced `$`/`$$`, no broken `\` escapes).
8. **Check length** against the family guidance. Over by a lot → first strip hype and repetition; if it's still long because the content is genuinely big, split into a series (one topic per post), never thin the analysis. Under → the post is probably missing a section the template calls for; add it from the source material (or a textbook-standard fact), or mark `<!-- TODO -->`.
9. **Read it back, start to finish.** Does it sound like one person explaining something they understand? If not, another pass.
10. **`draft: true` → `false`** only if every checklist item passes.

## Per-post checklist (the gate)

```
Frontmatter
[ ] title: "Concept - angle", one spaced-hyphen separator, <= ~70 chars,
    no trailing punctuation or stray quotes, concrete noun first
[ ] description: 25-50 words, concrete, no "delve"/"this post", ends with a period
[ ] tags: 2-4, Title Case, primary field tag first, from the taxonomy map
[ ] existing file: date, slug, filename all unchanged; new split-part file: slug is /<category>/<name>
[ ] draft flipped to false only if everything below passes

Lede
[ ] hook in sentence 1, <= 25 words, concrete, no throat-clearing
[ ] no "welcome back", no "in today's world", no "have you ever wondered"
[ ] context + one-line promise present
[ ] <= 4 short paragraphs, no heading

Body
[ ] every section is one idea; titles are plain (no "...on Steroids", no "!")
[ ] intuition / example before every definition and equation
[ ] every jargon term glossed at first use; bold only on first definition
[ ] paragraphs 2-4 sentences; no screen-height blocks; lists only for parallel items
[ ] exclamation marks <= 1 per ~1000 words; no hype interjections; no emoji
[ ] all source equations / numbers / derivation steps / edge cases / code preserved; each motivated before it appears
[ ] any added fact/quote is textbook-standard or checkable, accurate, and attributed
[ ] code blocks language-tagged and complete enough to run
[ ] images have descriptive alt text; needed-but-missing figures marked TODO
[ ] no banned phrases (voice-and-style.md §7); no AI-slop tells (§8)
[ ] no mojibake / encoding artifacts
[ ] inter-post links written as /citadel/<target slug tail>; every target slug exists
[ ] MDX-safe: no bare < > { } in prose (inside $…$ or backticks), math on its own $$ lines
[ ] related posts are actually linked where the text refers to a sibling topic

Ending
[ ] a real close (keystone idea / forward pointer / honest limit / open question)
[ ] "## Key takeaways" bullets only if the post is reference-style
[ ] no "In conclusion", no "thanks for reading"

Maths (if the post has derivations)
[ ] every step still present; nothing replaced by "it can be shown"
[ ] variable names consistent within the post; each symbol defined at first use
[ ] any generalisation keeps the concrete case visible as an instance
[ ] results and their physical/mathematical meaning unchanged from the source

Whole
[ ] length reflects the content: hype/repetition gone, no analysis or depth dropped
[ ] if it can't fit one page without thinning, it was split into a focused series
[ ] reads aloud like one person who understands the topic
[ ] category content map updated; every chunk this post shed is logged against a named sibling
[ ] category-lens conservation: nothing the category held is now missing from the category
```

## Commits

- One commit per post, or one per small batch within a category.
- Message: `blog: restyle <category>/<file>` (e.g. `blog: restyle system-design/discord.md`).
- For a split: `blog: split physics/big-bang.md into <n>-part series` and list the new files in the body.
- Don't commit `MIGRATION.md` separately every time — fold the tick into the post's commit.

## Tracking

`MIGRATION.md` at the repo root has a row per category with a checkbox and a `done / total` count. Update the count as you finish posts; check the box when the category is fully migrated and the user has signed off.
