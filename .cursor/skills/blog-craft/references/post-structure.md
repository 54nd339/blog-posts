# Post structure

## Anatomy of every post

```
---
frontmatter          # see frontmatter-and-tags.md
---

Lede                 # 2–4 short paragraphs, NO heading. Hook → context → promise.

## Section            # one idea. 150–500 words. Plain-noun or question title. Tracks the arc (see below).
### Sub-step          # optional, for stages of one idea

## Section
...

## <a real ending>    # see "Endings" below. Not "## Conclusion".
```

- **No H1 in the body.** The `title` in frontmatter is the H1; the site renders it. Start the body with the lede.
- **`##` for sections, `###` for sub-steps.** Don't go to `####`. If you need four levels the section is really two sections.
- **`---` horizontal rules** between top-level sections are part of the current house look; keep them if the post already uses them consistently, or drop them all. Don't leave it half-and-half.

---

## The lede (opening 2–4 paragraphs)

The single highest-leverage edit in this migration. Structure:

1. **Hook** — first sentence, ≤ 25 words, concrete, no throat-clearing. Get to the actual subject by sentence three.
2. **Context** — where this sits, why it's worth the reader's next ten minutes.
3. **Promise** — one sentence on what the reader will be able to do or see by the end. Stated plainly, once.

### Openings menu (pick per post — never reuse a formula)

- **The concrete puzzle / surprising fact.** *"A line from the Sun to a planet sweeps equal areas in equal times. That one sentence is a conservation law in disguise."*
- **The question the reader already has.** *"Why does `git rebase` sometimes touch commits you didn't ask it to?"*
- **The friction.** *"Every explanation of monads reaches for burritos. This one won't."*
- **The stakes / where it fits.** *"A Bloom filter is how you answer 'have I seen this before?' when 'this' numbers in the billions and won't fit in RAM."*
- **One line of history, when the history is the insight.** *"In 1965 Gordon Moore drew a straight line through five data points. The industry has been chasing it ever since."*
- **The naive approach and its wall.** *"The obvious way to multiply two n-digit numbers takes n² single-digit multiplies. Karatsuba gets it under n^1.59, and the trick is one subtraction you wouldn't think to make."*

### Banned openings

- "Hey everyone, and welcome back to the blog!"
- "In today's fast-paced digital world..."
- "Have you ever wondered..." (unless the wondering is specific and genuinely the hook)
- "X is one of the most important concepts in Y" (say *why* instead)
- "This post delves into / This exploration delves into"

---

## Endings

End on something the reader keeps. Options, roughly best-first:

- **The one thing to remember** — 2–4 sentences distilling the load-bearing idea. Not a summary of every section; the *keystone*.
- **Where this shows up next** — point forward to the concept this unlocks, or to a sibling post.
- **An honest limitation** — what this model or method doesn't handle, and what you'd reach for instead.
- **A question worth leaving open** — one that sends the reader off thinking, not a fake cliffhanger.

**`## Key takeaways` as a bullet list is allowed only for reference-style posts** (cheatsheets, "X in n minutes", the broad survey posts like `algorithms/Algorithms.md`) where a scannable recap genuinely helps. For a narrative explainer, a bullet dump at the end is filler — cut it and write two real sentences.

Banned endings: "In conclusion", "To sum up", "At the end of the day", "Thanks for reading", "Let me know what you think", any restatement of the intro.

---

## Structure: the arc, not the checklist

Before drafting, write one sentence: **what is the single thing the reader should understand after 8 minutes?** Every section either builds to it or is cut. The post works if the reader closes it thinking *"I finally understand why this works"* — not *"now I know what this is."*

Use the house arc in place of the textbook march (`definition → properties → implementation → complexity → related concepts`):

```
problem                 — the concrete thing that doesn't work yet, or the question with no clean answer
surprising observation  — the fact that makes the reader lean in: a bound that shouldn't be beatable,
                          a symmetry, a cost that vanishes, a case where the obvious method is wrong
intuition               — why the surprise is true, in plain words + one small picture, before formalism
technique               — the actual method / theorem / mechanism, now that the reader wants it
implementation          — runnable code, or the worked derivation, or the real architecture
edge cases              — where it breaks, what it assumes, the bug you'd actually hit
deeper implication      — what this changes about how you see the wider topic; the forward door
```

Not every post needs all seven beats and they can merge — but the *order* holds: **earn the technique before you present it.** A post that opens with a formal definition has skipped the first three beats and will read like a reference entry.

This is the editorial job: **explain the thing textbooks make unnecessarily hard.** Find the step everyone glosses — the "it can be shown that", the notation hiding a simple idea, the derivation line with no motivation — and spend your words exactly there.

### Systems, system-design, interview, tech posts

Same arc, specialised. Do **not** write `technology → what it does`. Write:

```
requirement             — the concrete demand: 10M writes/sec, p99 < 50ms, survive an AZ loss
constraint              — what that rules out, and the physical or economic reason it does
design decision         — the choice the constraint forces, stated as a choice among options
tradeoff                — what the choice costs, and what you'd have picked under different numbers
```

Every architectural claim traces back to a requirement. "It uses a message queue" is not an explanation. "A 30× write spike at lunch would drop requests if the DB took them synchronously, so a queue absorbs the spike — at the cost of read-your-write consistency" is.

## Length: three tiers

Match the tier to what the topic *is*, not to the directory.

| Tier | Words | Read | What belongs here |
|---|---|---|---|
| **Quick explanation** | 800–1,500 | 4–7 min | one algorithm, one data structure, one interview concept, one tool, one contest problem. `dsa/`, `cp/`, `algorithms/`, most `interview/`, most `system-design/`, `tech/`, `random-thoughts/` |
| **Core technical article** | 1,500–2,500 | 7–11 min | a mechanism with several moving parts, a subsystem, a design-heavy system-design post. `data-structures/`, `automata/`, `compiler-design/`, `computer-architecture/`, `computer-networks/`, `operating-system/`, `dbms/`, `artificial-intelligence/`, `parallel-computing/`, `big-data/`, `computer-graphics/`, `cyber-security/`, `coding-languages/`, `pentesting/`, `miscs/` |
| **Deep dive / foundational** | 2,500–4,000+ | 12+ min | a topic the rest of a field is built on: most of `maths/` and `physics/`, hub/overview posts, `cyber-security/maths.md`. No upper cap; split only when it's genuinely two topics. |

- **The band is a floor to reach, not only a cap.** A quick-explanation post at 550 words is thin — it is missing the surprising observation, or the edge cases, or the worked example. Add the real thing.
- **Never hit a number by cutting a derivation step, a caveat, an edge case, or a worked example.** If de-hyping took a post below its band, the compression went too far; restore the explanatory prose.
- **Growth must be real content**: intuition, a fuller derivation, a concrete example with numbers, a failure mode, a "why textbooks get this wrong" aside. Not restatement, not a longer intro, and never invented facts — use `<!-- TODO -->` for a real gap (see `voice-and-style.md`).
- A post still long after every bit of hype is gone is either the right length, or two posts. Split it (see `migration-workflow.md`).

## Section shape

- `##` for sections, `###` for sub-steps. One idea per section, 150–500 words.
- Section titles track the arc, in plain nouns or questions: `## The problem`, `## Why the obvious fix fails`, `## The idea`, `## Walking one example through`, `## Where it breaks`, `## What this buys you elsewhere`. Never `## Introduction`, `## Overview`, `## Conclusion`.

## Domain conventions

**Maths** (`maths/`, `cyber-security/maths.md`, `miscs/{game-theory,information-theory,randomised-algorithms}.md`)
- Inline `$...$`, display `$$...$$`, KaTeX syntax. Define every symbol in the sentence that introduces it.
- A derivation is prose with equations *in* it, not a wall of numbered lines. Between two display lines say what happened: "Divide both sides by $|\vec a|^2$:". Call out the non-obvious move: "The trick is to add and subtract the same term:".
- State assumptions before the derivation. Always include one full worked example with real numbers.

**Physics** (`physics/`)
- Every equation is motivated before it appears — never "the equation for this is". Build it from the picture.
- Units on every physical quantity; ballpark numbers make it real ("about $10^{-11}$ s — a hundred-billionth of a second").
- Keep personality and good analogies; cut exclamation marks and hype (see `voice-and-style.md` §5). Historical framing welcome but accurate — don't embellish quotes or dates.

**CS / algorithms** (`algorithms/`, `dsa/`, `cp/`, `data-structures/`, `automata/`, `compiler-design/`, `computer-architecture/`, `computer-networks/`, `operating-system/`, `dbms/`, `artificial-intelligence/`, `parallel-computing/`, `big-data/`, `computer-graphics/`)
- Show working code, not pseudocode-shaped prose. Complete enough to run, language-tagged fences. Over ~30 lines: show the core, trim or link the rest.
- Trace one concrete input all the way through before generalising. Complexity claims carry a reason: "each of the n items is pushed and popped once, so O(n)".

**Systems / interview** (`system-design/`, `interview/`, `tech/`, `random-thoughts/{latency-number,online-payments}.md`)
- Real numbers wherever possible: p99 latency, request rates, data sizes, dollar cost. That is what separates these from a Wikipedia summary.
- Name the real systems (Discord's ScyllaDB migration, Twitter Snowflake IDs) and attribute to their engineering blogs/talks; don't overstate what's public.
- Still a real explanation, not flashcards — the reader should understand the thing, not just recite it.

**Reflective** (`random-thoughts/`, `miscs/goals.md`)
- First person is fine. Keep it personal and specific — an argument or a story, no template rigidity, no takeaways bullets, no forced universality.

---

## Images and diagrams

- Reference images as `../images/<name>` (relative from the category directory — the established path). `images/` holds ~100 files, incl. Wikimedia Commons figures pulled in for physics. The main-site serves these via `/blog-images/<name>` (route handler); a plain relative path in the Markdown is all a post needs.
- **Every image gets alt text**, and it describes the figure: `![why the cache misses](../images/cache-issues.png)`, not `![cache-issues](...)`.
- **Caption** goes in the Markdown *title* (the quoted string after the path), not the alt: `![alt](../images/x.png "Caption shown under the figure. Source: …")`. Cite the source for anything not drawn in-house.
- Pulling a figure from Wikimedia Commons: `curl -sL "https://commons.wikimedia.org/wiki/Special:FilePath/<Filename>?width=1100" -o images/<name>.png` — with `?width` it returns a PNG raster even for `.svg` originals, so save it `.png`. Verify it's real (`file images/<name>.png`) and view it before writing a caption. Prefer local files over hotlinks (the site has no `remotePatterns`; hotlinked images do not load).
- **Mermaid** diagrams work: a ```mermaid fenced block renders as an SVG (flowchart / sequence / etc.). Use for a genuine flow, tree, or timeline — a decay chain, a classification tree, a pipeline of definitions. Keep node labels quoted (`A["…"]`) so units and symbols don't break the parser. Not for anything a table or a sentence does better.
- If a diagram would genuinely help and neither an image nor mermaid fits, insert `<!-- TODO: diagram — <what it should show> -->` and mention it to the user. Don't fake a description of one.
- Don't add decorative images. A figure has to carry information.

## Tables

- For genuine comparisons and lookups (complexity classes, HTTP codes, protocol fields). Not for prose that should be paragraphs.
- Max ~5 columns; keep cell text short. The surrounding prose still states the conclusion.

## Callouts / blockquotes

- Use a blockquote for a genuine gotcha, a key caveat, or a short pulled quote.
- Sparingly — one or two per post. A page of blockquotes is a page with no emphasis.
- Never put a joke in a callout.
