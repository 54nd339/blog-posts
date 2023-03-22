---
title: Words and Lexicons - Word Classes and Association Measures
description: The word-level groundwork under NLP - open and closed word classes and why part-of-speech matters, lexicons and WordNet, the statistical measures of word association (pointwise mutual information, the t-score, chi-square), estimating model parameters from counts, and inducing word classes by distributional clustering.
date: 2023-03-22
draft: false
slug: /artificial-intelligence/lexicon
tags:
  - Artificial Intelligence
  - NLP
  - Probabilistic Models
---

Before a machine can parse a sentence it needs to know what its words *are*: which grammatical class each belongs to, what senses it can carry, and which other words it tends to keep company with. That knowledge is the **lexicon**, and building and using it is the subject here — word classes, the dictionary resources like WordNet, and the statistical tests that measure how strongly two words are associated.

## Word classes

A **part of speech** (POS) groups words by grammatical behaviour. Two broad families:

- **Open classes** — nouns, verbs, adjectives, adverbs. New members arrive constantly (tech jargon, slang).
- **Closed classes** (function words) — determiners, pronouns, prepositions, conjunctions. Small and stable.

POS matters because words are ambiguous. "Back" is an adjective (*the back door*), a noun (*on my back*), an adverb (*win them back*), and a verb (*back the bill*). **POS tagging** — assigning the right tag in context — resolves this, and downstream tasks (parsing, named-entity recognition, information extraction) take POS tags as input. The tagging methods, tag sets, and accuracy figures are in [HMMs and POS tagging](/citadel/artificial-intelligence/hmm).

## Lexicons and WordNet

**Lexicography** is dictionary-making; in NLP a **lexicon** is a machine-readable store of words with their lemmas, possible POS tags, senses, morphological variants, and other features.

**WordNet** is the large-scale example. It groups English words into **synsets** — sets of synonyms — each with a short definition (gloss), and records semantic relations between synsets, most importantly **hyponymy/hypernymy** (the IS-A hierarchy: a *dog* is a *canine* is a *mammal*).

## Measuring word association

To know whether two words are genuinely related — a collocation, a topic pair — or just both common, you need a statistical test.

**Pointwise mutual information** (PMI) compares observed co-occurrence to what independence would predict:

$$\text{PMI}(w_1, w_2) = \log_2 \frac{P(w_1, w_2)}{P(w_1)\,P(w_2)}$$

Positive means the words co-occur more than chance; negative means they avoid each other; zero means independence. Since negative values are noisy for rare pairs, **positive PMI** clips them: $\text{PPMI} = \max(0, \text{PMI})$. Probabilities come from corpus counts. PMI and PPMI drive collocation discovery, distributional similarity, and feature weighting in sentiment analysis.

**The t-score** asks whether an observed co-occurrence frequency is significantly above the independence expectation, accounting for sample size — a t-test applied to co-occurrence data, good at surfacing collocations like "strong coffee".

**The chi-square ($\chi^2$) test** is an independence test: a high $\chi^2$ between a word and a class means strong association, which is why it's used in text categorisation to find the words that best indicate a category.

## Word classes at work

POS tags — and automatically induced word classes — feed a range of tasks:

- **Parsing** — tags are the terminal-level input to a syntactic parser.
- **Named-entity recognition** — runs of proper nouns often mark entities.
- **Information extraction** — extraction patterns are written over POS tags and words.
- **Sentiment analysis** — adjectives carry most of the polarity signal.
- **Language modelling** — POS n-grams (or mixed word/POS n-grams) generalise better for rare words than pure word n-grams.
- **Machine translation** — tags help disambiguate words and keep target grammar correct.

## Estimating parameters from counts

Probabilistic NLP models get their numbers from **maximum likelihood** — the choice of parameters that makes the training corpus most probable. For [n-gram language models](/citadel/artificial-intelligence/nlp), $P(w_i \mid w_{i-1}) = \frac{C(w_{i-1}, w_i)}{C(w_{i-1})}$. For a [Naive Bayes](/citadel/artificial-intelligence/ml) text classifier, $P(\text{word} \mid \text{class})$ comes from word-class counts, with smoothing. Because raw counts assign zero to unseen events, **smoothing** (Laplace, Good-Turing, interpolation, Kneser-Ney) reallocates some probability mass to them — a recurring necessity, covered in [NLP](/citadel/artificial-intelligence/nlp).

## Inducing word classes

You don't have to take word classes as given. The **distributional hypothesis** — words appearing in similar contexts have similar meaning or function — lets you learn them:

1. Represent each word as a **context vector**: counts of the words in its neighbourhood, or its syntactic-dependency contexts. The components can be raw counts, tf-idf scores, or PMI values.
2. **Cluster** the vectors — [k-means](/citadel/artificial-intelligence/unspervised-learning) or hierarchical clustering — to partition the vocabulary into classes.

These induced classes serve as features much like POS tags, sometimes capturing semantic distinctions POS tags miss.

## The hard parts

- **Ambiguity** — about 40% of word *tokens* in the Brown corpus are POS-ambiguous. Resolving them is the whole point of contextual tagging.
- **Granularity** — coarse tag sets lose useful distinctions; fine ones (the ~45-tag Penn Treebank set) cause data sparsity and make learning harder.
- **Out-of-vocabulary words** — a word unseen in training still needs a class, assigned from its morphology, its shape ("Xx-xxxx"), or a fallback `<UNK>` probability.
- **Computational cost** — sequence tagging and large-corpus clustering are both resource-heavy, and fine-grained class features inflate the models that use them.

## Practical notes

Regular expressions do most of the preprocessing and simple feature extraction. Hash maps hold counts and n-gram tables; [tries](/citadel/data-structures/trie) suit vocabularies and prefix search; for web-scale n-grams, prune to counts above a threshold. Map words to integer IDs to save space and speed comparisons. And don't reimplement tokenisation, tagging, or lemmatisation — NLTK, spaCy, and Stanford CoreNLP are well-tested.

## The one idea to keep

The lexicon is where NLP's statistics start. A word's class, its senses, and its co-occurrence profile are all estimated by counting in a corpus, and the association measures — PMI, t-score, chi-square — are just different ways of asking whether a count is bigger than chance would explain. Whether you use the hand-built POS tags or cluster your own word classes, the underlying signal is the same: words that behave alike appear in alike contexts.
