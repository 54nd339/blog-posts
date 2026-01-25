---
name: blog-craft
description: >-
  House writing style, post structure, frontmatter rules, and the
  category-by-category migration workflow for this blog-posts repo. Use whenever
  writing a new post, editing or restyling an existing one, migrating a category,
  or answering any question about the blog's voice, tone, length, headings,
  openings, endings, frontmatter, or tags.
---

# blog-craft

This repo holds ~290 technical posts across ~24 categories: maths, physics, CS theory (algorithms, automata, compilers, architecture, networks, OS, DBMS, AI), systems and interview prep, tech tools, and a few reflective notes. They were migrated from an older portfolio site ("pensieve") and carry two consistent problems:

- **Formulaic openings.** `"Hey everyone, and welcome back to the blog!"` opens 110 files. `"Let's dive in"` appears in 112. Maths posts open with `"This exploration delves into..."`.
- **Tone that misses in both directions.** Maths reads like a dry textbook; physics reads like a hype reel (exclamation marks every paragraph, section titles like *"Growth (and Decay) on Steroids"*, sign-offs like *"genius!"*).

There is also mechanical damage in a few files: Cyrillic mojibake (e.g. `physics/atoms.md` has `классика physics`), a stray trailing `"` in some titles (`algorithms/Algorithms.md`), and a misspelled filename (`computer-networks/newtwork-layer.md`).

This skill is the house style and the process for fixing all of it, one category at a time.

## When to use

- Writing a new post in any category.
- Restyling or migrating an existing post or a whole category.
- Any question about voice, length, structure, headings, frontmatter, tags, or "does this read right".

## The style in one paragraph

Write for one curious reader who met the prerequisites once and forgot the details. Open with the concrete thing — a number, a specific case, a question they already have — then generalise. Earn every piece of jargon by glossing it in plain words the first time it appears. Intuition first, formal statement second, derivation or code third, caveats last. Keep the warmth and cut the caffeine: a dry aside every few sections, not an exclamation mark every paragraph. Close with something worth keeping, not "In conclusion".

## Reference files

Read the ones the task needs; don't load all of them for a small edit.

| File | Read it when |
| --- | --- |
| [references/voice-and-style.md](references/voice-and-style.md) | Every edit. Reader model, person/tense, intuition-first method, jargon rule, humour budget, sentence mechanics, the banned-phrase list, AI-slop tells, before/after examples from this repo. |
| [references/post-structure.md](references/post-structure.md) | Shaping a post. The house arc (problem → surprising observation → intuition → technique → implementation → edge cases → deeper implication), the requirement→constraint→decision→tradeoff variant for systems/interview, the three length tiers (quick 800–1,500 / core 1,500–2,500 / deep dive 2,500–4,000+), openings menu, endings menu, heading rules, per-domain conventions, images and tables. |
| [references/frontmatter-and-tags.md](references/frontmatter-and-tags.md) | Touching frontmatter. Key schema, title format, `description` rules, the tag taxonomy with a directory→tag map, slug and date policy. |
| [references/migration-workflow.md](references/migration-workflow.md) | Migrating. Category order, the per-post checklist, editing without rewriting, commit convention, tracking in `MIGRATION.md`. |
| [references/reference-blogs.md](references/reference-blogs.md) | Background. The blogs this style is drawn from and what each one contributes. |

## Migration in brief

1. Pick a category. Open [MIGRATION.md](../../../MIGRATION.md) and take the next unchecked one. Before restructuring, build the category's **content map** (see `references/migration-workflow.md` → "Refocusing and splitting").
2. For each post, run the per-post checklist in [references/migration-workflow.md](references/migration-workflow.md).
3. **Per post, wide latitude.** Refocus a sprawling "god blog" post to one topic done well; split it into a focused series; simplify, re-notate, reassign variables (consistently), restructure or generalise the maths — whatever yields top-quality content. "Simplify" means *cleaner*, never *shallower* or *fewer steps*. Re-voice and re-structure freely.
4. **The conserved unit is the category, not the post.** Every technical claim, derivation, number, edge case, and worked detail the category held before migration must still be findable somewhere in that category afterwards — across however many parts a post was split into. Content a post sheds lands in a sibling (existing or a new split-part), never dropped. You *may* add textbook-standard facts and documented, attributed quotes; never fabricate; mark a real gap `<!-- TODO: ... -->`.
5. Keep `date`, `slug`, and existing **filenames** unchanged (site routing). New split-part files take `slug: /<category>/<name>`. Flip `draft: true` to `false` only when the post passes the checklist.
6. Commit per post or per small batch: `blog: restyle <category>/<file>`; for a split, `blog: split <category>/<file> into <n> parts` listing the new files.
7. Update `MIGRATION.md`: the count, and the category's content map.

Do not migrate a whole category in one pass without first showing the user one or two restyled posts and getting a thumbs up on the direction.

## Do not touch

- `miscs/markdown-playground.md` — a Markdown rendering test, not a post.
- Filenames — renaming breaks the site's slugs and inbound links. Fix a broken filename only if the user asks and accepts the slug change.
