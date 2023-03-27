---
title: Hidden Markov Models and POS Tagging - Decoding Hidden Sequences
description: Recovering a hidden state sequence from observations - the Markov property, the components of an HMM, its three fundamental problems, the Viterbi decoding algorithm with runnable code, Baum-Welch learning, and how HMMs and their discriminative rivals do part-of-speech tagging.
date: 2023-03-27
draft: false
slug: /artificial-intelligence/hmm
tags:
  - Artificial Intelligence
  - NLP
  - Probabilistic Models
---

You see the words of a sentence; you want its parts of speech. The words are what you *observe*; the tags are hidden states that produced them, each tag emitting a word and handing off to the next tag. That structure is a **hidden Markov model**, and the **Viterbi algorithm** recovers the most likely tag sequence from the words. This post covers the model, the three questions you can ask of it, and how it — and its discriminative competitors — do part-of-speech tagging.

## Markov models

A **Markov model** describes a sequence in which each event depends only on the immediately preceding state — the **Markov property**, or memorylessness. It has a set of **states**, and **transition probabilities** for moving between them. A toy weather model has states `{Sunny, Rainy}` and probabilities like $P(\text{Rainy} \mid \text{Sunny}) = 0.3$.

## Hidden Markov models

In an HMM the states are **not observed**. Instead each state probabilistically **emits** an observation, and you only see the observations. For POS tagging: the hidden states are the tags, the observations are the words. An HMM $\lambda$ has:

- $N$ hidden states $Q = \{q_1, \dots, q_N\}$ — the tag set.
- $M$ observation symbols $V = \{v_1, \dots, v_M\}$ — the vocabulary.
- **Transition probabilities** $A = \{a_{ij}\}$ with $a_{ij} = P(\text{state}_t = q_j \mid \text{state}_{t-1} = q_i)$.
- **Emission probabilities** $B = \{b_j(k)\}$ with $b_j(v_k) = P(\text{obs}_t = v_k \mid \text{state}_t = q_j)$.
- **Initial distribution** $\pi = \{\pi_i\}$ with $\pi_i = P(\text{state}_1 = q_i)$.

## Three problems

1. **Likelihood** — given $\lambda$ and an observation sequence $O$, compute $P(O \mid \lambda)$. Solved by the **Forward algorithm**.
2. **Decoding** — given $\lambda$ and $O$, find the most likely hidden state sequence. Solved by the **Viterbi algorithm**.
3. **Learning** — given observation sequences, set $A$, $B$, $\pi$ to maximise $P(O \mid \lambda)$. Solved by counting (supervised) or the **Forward-Backward / Baum-Welch algorithm** (unsupervised).

## Viterbi

Viterbi is dynamic programming over a **trellis** — a grid with time steps as columns and states as rows. $V_{t, j}$ holds the probability of the best state path that ends in state $q_j$ at time $t$ and emits the observations seen so far.

- **Initialisation** — $V_{1, i} = \pi_i\,b_i(o_1)$.
- **Recursion** — $V_{t, j} = \max_i\bigl[V_{t-1, i}\,a_{ij}\bigr]\,b_j(o_t)$, storing a backpointer to the $i$ that won the max.
- **Termination** — the best final state is $\arg\max_i V_{T, i}$.
- **Backtrack** — follow the backpointers from that state to $t = 1$ to read off the sequence.

In code, working in log space (see the numerical note below):

```python
import math

def viterbi(obs, states, start_p, trans_p, emit_p):
    log = math.log
    V = [{s: log(start_p[s]) + log(emit_p[s][obs[0]]) for s in states}]
    back = [{s: None for s in states}]
    for t in range(1, len(obs)):
        V.append({}); back.append({})
        for s in states:
            prev = max(states, key=lambda p: V[t-1][p] + log(trans_p[p][s]))
            V[t][s] = V[t-1][prev] + log(trans_p[prev][s]) + log(emit_p[s][obs[t]])
            back[t][s] = prev
    last = max(states, key=lambda s: V[-1][s])
    path = [last]
    for t in range(len(obs) - 1, 0, -1):
        path.append(back[t][path[-1]])
    return path[::-1]

states  = ["DT", "NN", "VB"]
start_p = {"DT": 0.6, "NN": 0.3, "VB": 0.1}
trans_p = {"DT": {"DT": 0.1, "NN": 0.8, "VB": 0.1},
           "NN": {"DT": 0.1, "NN": 0.3, "VB": 0.6},
           "VB": {"DT": 0.5, "NN": 0.4, "VB": 0.1}}
emit_p  = {"DT": {"the": 0.9, "dog": 0.05, "runs": 0.05},
           "NN": {"the": 0.05, "dog": 0.8,  "runs": 0.15},
           "VB": {"the": 0.05, "dog": 0.15, "runs": 0.8}}

print(viterbi(["the", "dog", "runs"], states, start_p, trans_p, emit_p))
# ['DT', 'NN', 'VB']
```

The recursion does $O(N^2 T)$ work — $N$ candidate predecessors for each of $N$ states at each of $T$ steps — versus the $N^T$ of enumerating every path.

## Learning the parameters

**Supervised** — with a corpus tagged by hand, MLE is just counting:

$$a_{ij} = \frac{\text{Count}(q_i \to q_j)}{\text{Count}(q_i)}, \qquad b_j(o_k) = \frac{\text{Count}(o_k \text{ from } q_j)}{\text{Count}(q_j)}, \qquad \pi_i = \frac{\text{Count(starts with } q_i)}{\text{sequences}}$$

**Unsupervised** — with only observation sequences, the **Baum-Welch** algorithm, a special case of [EM](/citadel/artificial-intelligence/nlp), iteratively re-estimates the parameters. It uses the **forward** probability $\alpha_t(i) = P(o_1 \dots o_t, \text{state}_t = q_i \mid \lambda)$ and the **backward** probability $\beta_t(i) = P(o_{t+1} \dots o_T \mid \text{state}_t = q_i, \lambda)$. The E-step computes expected transition and emission counts from $\alpha$ and $\beta$; the M-step re-normalises them into new parameters. It converges to a local optimum and is generally less accurate than supervised training.

## Numerical issues

Multiplying many probabilities below 1 quickly underflows to zero. The fix is to work with **log probabilities** — $\log(P_1 P_2) = \log P_1 + \log P_2$, so products become sums (as in the code above). The Forward-Backward algorithm can't take logs directly (it sums probabilities), so it introduces per-time-step **scaling factors** and undoes them at the end.

## POS tagging

A tagged sentence: `The/DT cat/NN sat/VBD on/IN the/DT mat/NN`. The **tag set** is the label inventory — the **Penn Treebank** set has about 45 tags; the **Brown** corpus set is larger and finer. Morphology helps: suffixes like `-ing`, `-ed`, `-s`, `-ly` cue tags, and [lemmatisation](/citadel/artificial-intelligence/nlp) lets a model generalise across inflected forms.

## Tagging methods

- **Rule-based** — linguists write rules ("if a word ends in `-ed` and follows a pronoun, tag it as a past-tense verb"). Accurate with enough effort, but brittle and labour-intensive.
- **HMM** — states are tags, transitions are $P(\text{tag}_j \mid \text{tag}_i)$, emissions are $P(\text{word} \mid \text{tag})$; Viterbi decodes a sentence. Trained supervised by MLE, or unsupervised by Baum-Welch.
- **Transformation-based (Brill)** — start from a baseline (assign each word its most frequent tag), then learn a ranked list of error-correcting rules of the form "change tag X to Y in context C", ordered by how many errors each fixes.
- **Maximum-entropy / MEMM** — a *discriminative* model of $P(\text{tag}_i \mid \text{tag}_{i-1}, \text{full observation})$, which can use rich overlapping features without independence assumptions.
- **Feature-based** — the current and neighbouring words, prefixes and suffixes, capitalisation, word shape ("Xx-xxxx"), and previously assigned tags, fed to a classifier. **Conditional random fields** are the related discriminative sequence model.

## Evaluation

**Accuracy** — correctly tagged words over total — is the headline metric; **per-tag precision and recall** matter for rare or task-critical tags. State-of-the-art supervised taggers reach about **96–97%** on the English Penn Treebank. **Unknown words** are the main differentiator between taggers, handled by morphology and word-shape features. Performance drops for **morphologically rich** languages (Turkish, Finnish, Arabic — many forms per lemma) and for low-resource languages where large annotated corpora don't exist; the tag set's granularity also affects both difficulty and score.

## The one idea to keep

An HMM separates *what generates the sequence* (hidden states with transition and emission probabilities) from *what you see* (the observations), and Viterbi is dynamic programming that finds the single best state path in $O(N^2 T)$ instead of exponential time. The discriminative taggers — MEMM, CRF — drop the generative story and model the tags given the words directly, which is what lets them throw in every feature they can think of.
