# AGENTS.md

This repo is a shared content library of ~290 technical blog posts (Markdown + images in `images/`), consumed by the site as a git submodule. Categories are top-level directories (`maths/`, `physics/`, `algorithms/`, `system-design/`, `interview/`, `tech/`, …). Larger categories are further split into **subcategory folders** (`physics/optics/`, `dsa/backtracking/`, …); the site loader walks categories recursively, and the `slug` stays flat (`/<category>/<filename-stem>`) regardless of subfolder — so moving a post between folders never changes its URL.

## Before writing or editing any post

**Invoke the `blog-craft` skill** (`.claude/skills/blog-craft/SKILL.md`). It is the house style, the per-family structure, the frontmatter and tag rules. Don't restyle a post from memory — read the skill's reference files for the task at hand.

## Hard rules

- **Per post: wide latitude.** Refocus a "god blog" post to one topic, split it into a focused series, re-voice and re-structure freely, and reshape the maths (simplify, reassign variables, generalise) — but *cleaner, never shallower*: no skipped steps, no lost rigor. Trimming length is hype and repetition only, never depth.
- **Adding is allowed; fabricating is not.** Textbook-standard facts and real, attributed quotes are fine. Never invent a fact, date, quote, or benchmark — leave a `<!-- TODO -->` and say so.
- **Never change `date`, `slug`, or an existing filename.** The site's routing and inbound links depend on them. New split-part files take `slug: /<category>/<name>`. Moving a post into a subcategory folder (`git mv`, filename and `slug` unchanged) is fine — routing is by `slug`, not path.
- **YAML-safe frontmatter.** Unquoted scalars with `: ` break `js-yaml` and abort the main-site build. Prefer a double-quoted single-line `description` (see `.cursor/rules/yaml-frontmatter.mdc` and `blog-craft` → `frontmatter-and-tags.md`).
- Flip `draft: true → false` only when a post passes the skill's per-post checklist.
- `miscs/markdown-playground.md` is a rendering test, not a post — leave it.
