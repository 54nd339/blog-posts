---
title: Lexicalised Parsing and Statistical Machine Translation
description: Two places statistics reshaped NLP - lexicalised PCFGs that condition grammar rules on head words to capture what specific verbs expect, and statistical machine translation built on the noisy channel model, with word alignment learned by EM through the IBM models.
date: 2023-03-30
draft: false
slug: /artificial-intelligence/machine-translation
tags:
  - Artificial Intelligence
  - NLP
  - Probabilistic Models
---

A plain [PCFG](/citadel/artificial-intelligence/grammar) treats every verb the same: the probability of `VP → V NP NP` doesn't depend on whether the verb is "gave" (which wants two objects) or "slept" (which wants none). That's too strong an independence assumption, and fixing it — **lexicalised parsing** — was one leap in parsing accuracy in the late 1990s. The same statistical instinct, wrapped around a noisy channel, produced the first machine translation that worked. This post covers both.

## Lexicalised PCFGs

The fix is to annotate every non-terminal in a parse tree with its **lexical head** — the single most important word in the phrase it covers. The NP "the big red car" has head "car", so it becomes `NP(car)`. Rules are then lexicalised: `S → NP VP` becomes $S(h_S) \rightarrow NP(h_{NP})\ VP(h_{VP})$, and rule probabilities are conditioned on those heads, e.g. $P(S(\text{saw}) \rightarrow NP(\text{man})\ VP(\text{saw}) \mid S(\text{saw}))$.

This captures dependencies between specific words: verb subcategorisation (what complements a verb takes) and prepositional-phrase attachment preferences. The effect on accuracy is large — from roughly 73% to 88% labelled F1 on standard datasets.

**Charniak's 1997 model** is the well-known example. It conditions each rule expansion on the head word of the parent non-terminal and the head of the constituent being expanded, and separately models the probability of choosing a head word given the parent's head. The conditioning makes probabilities far more informative:

- $P(VP \rightarrow V\ NP\ NP) = 0.00151$ — the general rule probability.
- $P(VP \rightarrow V\ NP\ NP \mid \text{head} = \textit{said}) = 0.00001$.
- $P(VP \rightarrow V\ NP\ NP \mid \text{head} = \textit{gave}) = 0.01980$.

The cost is **data sparsity**: lexicalisation multiplies the number of parameters — a rule per head-word combination — so most lexicalised rules have few or no counts in training. The remedy is heavy **smoothing**, typically linear interpolation (shrinkage) that blends the specific lexicalised estimate with the more general, less lexicalised one.

## Statistical machine translation

**Statistical MT** learns to translate from **bilingual parallel corpora** — the same text in two languages.

### The noisy channel model

Frame the source sentence $f$ (say, French) as a corrupted version of an intended target sentence $e$ (English) that came out of a [noisy channel](/citadel/artificial-intelligence/nlp). Recover the most probable $e$:

$$\hat{e} = \arg\max_e P(e \mid f) = \arg\max_e P(f \mid e)\,P(e)$$

by Bayes' rule, dropping the constant $P(f)$. This splits the problem in two:

- **Translation model** $P(f \mid e)$ — how well $f$ is a translation of $e$: word and phrase correspondences, reordering. It's happy with disfluent English as long as the words match.
- **Language model** $P(e)$ — how fluent $e$ is as English. An [n-gram model](/citadel/artificial-intelligence/nlp) trained on a large monolingual English corpus. It knows nothing about $f$; it just enforces good English.

The system searches for an $e$ that scores well on both.

### Word alignment

Building $P(f \mid e)$ hinges on **word alignment** — knowing which source words correspond to which target words. For "le chat noir" ↔ "the black cat": le↔the, chat↔cat, noir↔black. Alignments aren't one-to-one — they can be one-to-many, many-to-one, or map a word to NULL — and they're **hidden variables**, not marked in the corpus.

The **IBM Models** (1 through 5) are the classic family for learning word alignments and lexical translation probabilities $t(f_j \mid e_i)$ from a parallel corpus. Because alignments are unobserved, they're trained with the [EM algorithm](/citadel/artificial-intelligence/hmm): the E-step computes expected alignment counts given the current translation probabilities, the M-step re-estimates the translation probabilities from those counts, and the loop repeats until convergence.

### Putting it together

The **language model** parameters come from MLE with smoothing (Kneser-Ney, or stupid backoff at web scale) on monolingual target text. The **translation model** parameters come from EM on the sentence-aligned parallel corpus. At translation time, a **decoder** searches the vast space of possible target sentences for the $\hat{e}$ maximising $P(f \mid e)\,P(e)$.

## Where it went

Neural machine translation — encoder-decoder [RNNs](/citadel/artificial-intelligence/generalisation) and then Transformers — has largely replaced statistical MT. But the pieces carried over: a language model still scores fluency, alignment became the attention mechanism, and everything is still fit by statistical estimation from parallel text.

## The one idea to keep

Both halves add word-specific knowledge to a model that was too coarse without it. A lexicalised PCFG conditions grammar rules on head words, so "gave" and "slept" get different structures; statistical MT factors translation into a fluency model and a fidelity model and learns the fidelity part's hidden alignments with EM. In both cases the price of the extra specificity is sparsity, paid down with smoothing.
