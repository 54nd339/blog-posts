---
title: Natural Language Processing - Ambiguity, Language Models, and the Noisy Channel
description: Why human language is hard for computers and how statistics helps - the four levels of ambiguity, the chain rule and perplexity, n-gram language models and the smoothing they need, the noisy channel model, the EM algorithm for hidden variables, text normalisation, and the two views of syntax.
date: 2023-03-17
draft: false
slug: /artificial-intelligence/nlp
tags:
  - Artificial Intelligence
  - NLP
  - Probabilistic Models
---

"I saw a man on a hill with a telescope." Who has the telescope — you, the man, or the hill? A person barely notices the ambiguity; a computer sees five valid parses and no way to choose. Human language is ambiguous at every level, and machines lack the world knowledge that lets us resolve it instantly. **Natural language processing** copes by going statistical: learn the probabilities of language from large amounts of text, and let those probabilities break the ties.

This post covers the ambiguity NLP has to fight, the probability tools it fights with — language models, the noisy channel, EM — and the preprocessing and syntactic structure underneath.

## What NLP does

NLP sits where computer science, AI, and linguistics meet, with the goal of getting computers to process, understand, and generate human language usefully. It's already everywhere: question answering, information extraction (calendar entries from emails), machine translation, sentiment analysis, spam detection, part-of-speech tagging, named-entity recognition.

## Why it's hard: ambiguity

Language is ambiguous at four levels:

- **Lexical** — words have multiple senses. "Bank" is a riverside or a financial institution. In "Fed raises interest rates", is "raises" a verb or a noun, "interest" a noun or an adjective?
- **Syntactic** — one sentence, many grammatical structures. The telescope sentence above.
- **Semantic** — the meaning is unclear even when the syntax is fixed.
- **Discourse (anaphora)** — pronouns depend on context. "Carter told Mubarak he shouldn't run again." Who is "he"?

Headlines exploit this by accident — "crash blossoms" like "Teacher Strikes Idle Kids". And ambiguity isn't the only obstacle: non-standard text (tweets, texting), languages without spaces between words (Chinese, Japanese) or with long compounds (German `Lebensversicherungsgesellschaftsangestellter`), the enormous amount of unstated world knowledge, idioms ("get cold feet"), and entity names that look like ordinary phrases ("Where is *A Bug's Life* playing?").

## Probability and information theory for NLP

The **chain rule of probability** gives the joint probability of a word sequence:

$$P(w_1, w_2, \dots, w_n) = \prod_{i=1}^{n} P(w_i \mid w_1, \dots, w_{i-1})$$

**Entropy** measures the uncertainty of a random variable — here, how hard the next word is to predict. **Perplexity** is the standard intrinsic score for a language model: the inverse probability the model assigns to a test set, normalised by length,

$$PP(W) = P(w_1 \dots w_N)^{-\frac{1}{N}}$$

Lower is better. It reads as the model's average branching factor — the effective number of equally likely choices it faces at each word. (More on entropy and coding in [information theory](/citadel/miscs/information-theory).)

## Language models and n-grams

A **language model** assigns a probability to a word sequence, or predicts the next word given the previous ones. Computing the full chain rule is infeasible — you'd need to have seen every long sequence in training. The fix is the **Markov assumption**: the next word depends only on the previous $n - 1$:

$$P(w_i \mid w_1, \dots, w_{i-1}) \approx P(w_i \mid w_{i-n+1}, \dots, w_{i-1})$$

A **unigram** model ($n = 1$) treats words as independent; a **bigram** conditions on one previous word; a **trigram** on two. Probabilities are estimated by **maximum likelihood** from a corpus — for a bigram:

$$P(w_i \mid w_{i-1}) = \frac{\text{count}(w_{i-1}, w_i)}{\text{count}(w_{i-1})}$$

## Smoothing

MLE gives any n-gram not seen in training a probability of zero, which zeros out the probability of every test sentence containing it. **Smoothing** moves probability mass from seen n-grams to unseen ones:

- **Add-one (Laplace)** — add 1 to every count: $P(w_i \mid w_{i-1}) = \frac{C(w_{i-1}, w_i) + 1}{C(w_{i-1}) + V}$ with $V$ the vocabulary size. Simple, too blunt for serious language modelling but fine for text classification.
- **Good-Turing** — use the number of n-grams seen exactly once to estimate the total mass for unseen n-grams, adjusting an n-gram of count $c$ to roughly $(c+1)\,N_{c+1}/N_c$.
- **Interpolation** — a fixed weighted mix of unigram, bigram, and trigram estimates, with weights $\lambda_i$ summing to 1, learned on held-out data.
- **Backoff** — use the highest-order n-gram that has a non-zero count; if it doesn't, "back off" to the next lower order, scaled by a backoff weight. "Stupid backoff" is a cheap version for web-scale data.

## The noisy channel model

A general framework for spelling correction, speech recognition, and translation. An intended message $w$ passes through a "noisy channel" — a typist, a speaker in a loud room, a translation process — and is observed as a possibly-corrupted $x$. Given $x$, recover the most probable $w$:

$$\hat{w} = \arg\max_w P(w \mid x) = \arg\max_w P(x \mid w)\,P(w)$$

by Bayes' rule, dropping the constant $P(x)$. Two components: $P(w)$ is a **language model** (which messages are plausible), and $P(x \mid w)$ is a **channel model** (how $w$ tends to get corrupted into $x$ — the probability of a given typo, mis-hearing, or translation). [Statistical machine translation](/citadel/artificial-intelligence/machine-translation) is built on exactly this decomposition.

## The EM algorithm

**Expectation-Maximisation** finds maximum-likelihood (or MAP) parameters when the model has **hidden variables** you never observe. It alternates:

1. **E-step** — estimate the hidden variables' distribution, given the current parameters and the observed data.
2. **M-step** — re-estimate the parameters to maximise the expected log-likelihood from the E-step.

Repeat until the parameters converge. In NLP, EM trains [HMMs for POS tagging](/citadel/artificial-intelligence/hmm) (the true tags are hidden), learns word alignments in machine translation, and fits PCFGs with latent annotations.

## Preparing text

Before any analysis, text is normalised:

- **Tokenisation** — split a character stream into tokens. Handle punctuation, hyphens, contractions ("what're" → "what", "are"), possessives ("Finland's"). Languages without word separators — Chinese, Japanese, Thai — need a word-segmentation step, often maximum matching.
- **Case folding** — lowercase everything so "The" and "the" match. Standard for information retrieval; sometimes harmful elsewhere (it destroys "US" vs "us").
- **Morphology** — words are built from **morphemes**: a **stem** carrying the core meaning plus **affixes** carrying grammatical information. **Lemmatisation** reduces a word to its dictionary form ("cars" → "car"; "am", "are", "is" → "be"), needing a dictionary and often the POS. **Stemming** just chops affixes to a common stem, which may not be a real word ("automate", "automatic", "automation" → "automat"); Porter's algorithm is the standard rule-based English stemmer.

## Syntax: two views

Syntactic structure is described in one of two ways:

- **Constituency (phrase structure)** — words group into nested constituents: noun phrases, verb phrases, prepositional phrases. Formalised with context-free grammars and their probabilistic version, PCFGs. *Headed* phrase structure marks the most important word (the head) of each phrase.
- **Dependency** — structure is a set of binary directed links from a **head** word to its **dependents**. In "The boy put the tortoise", "put" is the head, with "boy" as subject and "tortoise" as object. Dependency structures are usually trees and can be *non-projective* (crossing links when words are laid out in order), which some phrase-structure formalisms can't represent.

Both, and the algorithms that build them, are covered in [grammars and parsing](/citadel/artificial-intelligence/grammar).

## The one idea to keep

NLP's core move is turning "understand this sentence" into "assign a probability to it". A language model scores fluency, a channel model scores corruption, and Bayes combines them; EM fits the models when part of the structure is hidden. Smoothing is the unglamorous necessity that keeps a single unseen word from setting the whole probability to zero.
