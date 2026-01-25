# Voice and style

The house voice sits between two traditions that the best technical writers share:

- **The explainer tradition** — Better Explained (Kalid Azad), 3Blue1Brown (Grant Sanderson), Quanta Magazine. Intuition first, formalism second. Analogy, picture, concrete case, then the symbols.
- **The engineering tradition** — Julia Evans (jvns.ca), Phil Eaton (eatonphil), Dan Luu, Bartosz Ciechanowski. Show the real thing. Complete code, real numbers, honest about tradeoffs and about what you don't know.

Maths and physics posts lean explainer. Algorithms, systems, tools, and interview posts lean engineering. Every post uses both to some degree.

---

## 1. The reader

Write for **one** person: a curious undergraduate (or self-taught equivalent) who has seen the prerequisites once and forgotten the details. They are smart, short on time, and allergic to being talked down to.

Consequences:

- Don't explain what they already know (what a variable is, what a function is).
- Don't skip the step that actually trips people up. Julia Evans's edge is "remembering what it felt like to not understand something." Find that step and slow down there.
- Pick one level and hold it. A post that serves beginners, mid, and experts at once serves no one and runs three times too long.
- No "as everyone knows", "simply", "just", "obviously", "trivially", "of course". If it were obvious the reader wouldn't be here.

---

## 2. Person, tense, distance

- **Second person.** "You pass the function a pointer." Address the reader.
- **"We" only for a shared walk-through** you are genuinely doing together: "we'll derive the closed form", "let's trace one request". Not "we" as a royal we.
- **Present tense** for how things work. Past tense only for history and for a specific past event ("Discord started on MongoDB in 2015").
- **Active voice.** "The scheduler preempts the thread", not "the thread is preempted by the scheduler".
- **Contractions are fine** (it's, you'll, don't). They read like a person.
- First person singular ("I") is allowed in reflective posts (`random-thoughts/`, `miscs/goals.md`) and for a genuine opinion or experience elsewhere. Use it sparingly; it is a spice, not a base.

---

## 3. Intuition before formalism (the ADEPT order)

Never open a section with a definition or an equation. Move blurry-to-sharp:

1. **Analogy** — what is this like that the reader already knows? Analogies are allowed to be imperfect; they are scaffolding, say so if it matters.
2. **Diagram / picture** — a figure, or a vivid verbal picture if no figure exists.
3. **Example** — one concrete case with real values. Concrete before abstract: a specific 3×3 matrix before "an n×n matrix", `n = 8` before "for all n".
4. **Plain-English statement** — what's true, in words.
5. **Technical statement** — now the formal definition, the notation, the equation.

For a derivation or proof, still lead with *why we'd want it* and *what the result will say* before the algebra.

**Before** (`maths/logarithms.md`, current):

> The mathematical concepts of exponential and logarithmic functions are fundamental to describing and understanding a vast array of phenomena... This exploration delves into their definitions, fundamental laws...

**After:**

> A logarithm answers one question: *what power do I raise this base to, to get that number?* `log₁₀(1000) = 3` because `10³ = 1000`. Everything else — earthquake scales, pH, the way your ear hears loudness, the running time of binary search — is that question wearing different clothes.

---

## 4. Jargon

Every technical term gets a plain-English gloss **the first time it appears**, in the same sentence.

- Bad: "This uses a Bloom filter." (reader who doesn't know is now lost)
- Good: "This uses a *Bloom filter* — a small bit-array that can tell you 'definitely not seen' or 'probably seen', trading exactness for space."

After the first gloss, use the term freely. Don't re-explain it every time.

Bold a term on **first definition only**. The current posts bold every key noun on every appearance; that trains the eye to ignore bold. One bold per concept.

---

## 5. Humour budget

Humour helps — it buys attention and makes dense material approachable — but the research consensus (and the failure mode in this repo's physics posts) is that it must be **relevant, subtle, sparing, strategic**. Rules:

- **Exclamation marks: budget one per ~1000 words, and zero is a fine number.** The current physics posts average one every few lines. Cut them. A sentence that needs `!` to land usually needs rewriting instead.
- **Wry, not zany.** A dry understatement or a well-placed aside beats "Bam!", "Boom!", "genius!", "mind-blowing", "buckle up", "strap in", "hold onto your hats".
- **At most one playful section title per post**, and only if it still tells the reader what the section covers. `"The Two-Body Tango"` is fine as a flourish on top of a clear subject; `"Growth (and Decay) on Steroids"` as the whole title is not.
- **Never** joke inside: a definition, a theorem statement, a result, a security warning, a complexity bound, a "gotcha" callout. Precision content is a joke-free zone.
- No memes, no internet slang, no "the physics police will come for me", no forced pop-culture. The audience is international and includes non-native English readers; keep humour translatable.
- Delete emoji from body text. (Frontmatter never has emoji — see `frontmatter-and-tags.md`.)

**Before** (`physics/atoms.md`, current):

> Hey there, curious minds! Ever pondered what makes up, well, *everything*? From the phone in your hand to the stars in the sky, it all boils down to fantastically tiny, incredibly mighty building blocks called **atoms**. But our understanding of these little guys didn't just pop into existence! ... So, buckle up, science adventurers!

**After:**

> Every model of the atom has been wrong, and each one was wrong in a more interesting way than the last. Democritus guessed at an uncuttable speck. Dalton made it a billiard ball. Thomson stuffed electrons into a positive pudding. Then Rutherford fired helium nuclei at gold foil and a few bounced straight back — and the pudding was done. This post follows that chain of wrong answers to the quantum picture we use now.

The rewrite keeps the personality (a point of view, a through-line, a bit of wit in "the pudding was done"). It drops the caffeine.

---

## 6. Sentences and paragraphs

- **Paragraphs: 2–4 sentences, one idea each.** A screen-height paragraph is a bug.
- **Vary sentence length.** A short one after two long ones lands. If every sentence is 25+ words, the reader drowns.
- **Front-load the point.** First sentence of a paragraph says what the paragraph is about.
- **Lists earn their place.** Use a bulleted list for genuinely parallel items (3–7 of them). Don't bullet prose that should flow. Don't nest past two levels.
- **One idea, one sentence.** Split a sentence with three "and"s and a "which".
- **Cut throat-clearing.** "It is important to note that", "It's worth mentioning that", "As we can see", "Basically", "Essentially" at the start of a sentence — delete, the sentence is stronger without.

---

## 7. Banned and smell-list phrases

Hard-banned (search and destroy on every migration):

- `Hey everyone, and welcome back to the blog` and every "welcome back" variant.
- `Let's dive in` / `Let's dive into` / `dive deep` / `deep dive` as filler (the phrase is fine when you are literally describing depth).
- `buckle up`, `strap in`, `hold onto your hats`, `hang tight`.
- `In today's fast-paced world`, `In this day and age`, `In our increasingly digital lives`, `In the world of X`.
- `delve`, `unleash`, `unlock the power of`, `harness the power of`, `the realm of`, `navigating the world of`, `a testament to`.
- `game-changer`, `revolutionary` / `revolutionize` (unless the thing was literally a revolution and you say why), `seamless`, `cutting-edge`, `state-of-the-art` as filler.
- Hype interjections: `Bam!`, `Boom!`, `Voila!`, `genius!`, `mind-blowing`, `mind-boggling`, `wild ride`, `epic`, `insane` (as praise).
- `In conclusion`, `To sum up`, `To wrap up`, `At the end of the day`, `Without further ado`.
- `Thanks for reading`, `let me know in the comments`, `hit like and subscribe`, `stay tuned` (the site has no comments and no feed loyalty mechanic).

Smell list (allowed rarely, with intent — if you use one, you should be able to say why):

- Rhetorical `Have you ever wondered...?` as an opener. Fine only if the wondering genuinely *is* the hook and it's specific.
- `simply`, `just`, `merely`, `all you have to do is` — usually condescending.
- Em dash as a default connector. One or two per post for a real dramatic pause; not one per paragraph.
- Starting three consecutive sentences with the same word.

---

## 8. Don't sound machine-generated

Current AI-writing tells, all to avoid:

- **"It's not just X, it's Y"** and "isn't merely about X". Rephrase as a direct claim.
- **Rhetorical triplets everywhere** — "faster, cheaper, and more reliable", "the what, the why, and the how". One list of three is fine; three per section is a tic.
- **Empty rhetorical questions** that you immediately answer: "So what does this mean? It means...". Just say what it means.
- **Sentence-initial "Moreover", "Furthermore", "Additionally"** stacked paragraph after paragraph.
- **The wrap-up that restates the intro** in the same words with "In conclusion" bolted on.
- **Vague uplift**: "This opens up exciting possibilities", "The future looks bright", "possibilities are endless".

The defense is specificity: a real number, a named tool, a concrete failure you can describe, a genuine opinion. Write from particular knowledge, not from the shape of an essay.

---

## 9. Accuracy and honesty

- **The conserved unit is the category, not the post.** Every technical claim, derivation, number, edge case, and worked detail a category held before migration must still be findable somewhere in that category afterwards — across however many parts a post was split into. Within that, an individual post can be refocused, split, rewritten, and have content relocated to a sibling. Nothing lands on the floor.
- **Losing depth is a worse failure than being long.** Trimming comes from cutting hype, throat-clearing, and repetition — never from dropping analysis, a derivation step, a caveat, or a case. If a topic is too big for one good page, **split it into a focused series**; don't thin it.
- **You may reshape the maths for quality.** Simplify a derivation, reassign or rename variables (consistently across the post), restructure the steps, tighten the notation, or state a result in a more general form with the concrete case kept visible as an instance. Guardrail: *cleaner, not shallower* — never skip a step, hand-wave a "it can be shown", or lose rigor. The result and its meaning stay identical.
- **You may add well-established facts and real, attributed quotes** — a standard measured value (Mercury's 43″/century), a landmark result and its date (Eddington 1919, GW150914), a documented quote from a scientist or paper — when it makes the analysis more concrete or complete. Bar: textbook-standard or checkable, stated accurately, attributed where a reader would want the source. Still **never fabricate** a number, date, quote, citation, or benchmark, and never dress a guess as a fact. A genuine gap you can't fill stays a `<!-- TODO -->`.
- If a claim looks wrong, flag it to the user — don't silently "fix" it and don't silently keep it.
- **State tradeoffs.** Every design or method has a cost; name it. "Bloom filters never give a false negative but do give false positives, at a rate you tune with the bit-array size."
- **Admit the edge.** "This derivation assumes the collisions are perfectly elastic; real gases aren't, which is why the next section exists." Honesty about limits builds trust and is a hallmark of every blog in `reference-blogs.md`.
