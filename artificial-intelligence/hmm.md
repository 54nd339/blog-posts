---
title: Hidden Markov Models & Tagging - Unraveling Sequences in Language
description: A deep dive into Hidden Markov Models (HMMs), their algorithms like Viterbi and Forward-Backward, and their application to Natural Language Processing tasks like Part-of-Speech (POS) tagging, including various tagging methods and evaluation.
date: 2023-07-07
draft: false
slug: /pensieve/ai-ml/hmm-pos-tagging
tags:
  - AI/ML
  - CS Basics
---

Hello, language detectives and AI enthusiasts! Much of natural language involves sequences – sequences of words forming sentences, sequences of sounds forming speech. How do computers make sense of these sequences and uncover the underlying structure, like identifying the part-of-speech for each word? One of the most powerful statistical tools for this is the **Hidden Markov Model (HMM)**.

Today, we'll explore:
* The basics of **Markov Models** and how they lead to **HMMs**.
* Key HMM algorithms like **Viterbi** and **Forward-Backward**.
* The crucial NLP task of **Tagging** (especially Part-of-Speech tagging).
* Various methods for tagging, from rules to statistical models like HMMs and Maximum Entropy taggers.
* How we evaluate the performance of these tagging systems.

Let's unravel the secrets hidden in linguistic sequences!

---
## Understanding Sequences: Markov Models

Before diving into HMMs, let's understand their simpler predecessor: **Markov Models** (or Markov Chains).
A Markov Model describes a sequence of possible events where the probability of each event depends *only* on the state attained in the previous event. This is known as the **Markov property** or memorylessness – the past is irrelevant given the present state.
* **States**: A finite set of possible states the system can be in.
* **Transitions**: Movements between states.
* **Transition Probabilities**: The probability of moving from one state to another.

For example, a simple weather model might have states {Sunny, Rainy} and probabilities of transitioning from Sunny to Rainy, Sunny to Sunny, etc.

## Hidden Markov Models (HMMs): When States Are Unseen ️‍

A **Hidden Markov Model (HMM)** is a statistical model where the system being modeled is assumed to be a Markov process with **unobserved (hidden) states**. We don't directly see the states; instead, we see **observations** that are probabilistically emitted by these hidden states.

HMMs are incredibly useful for sequence labeling tasks in NLP. For example, in Part-of-Speech (POS) tagging:
* The **hidden states** are the POS tags (Noun, Verb, Adjective, etc.).
* The **observations** are the actual words in the sentence.
We observe the words, and we want to infer the most likely sequence of hidden POS tags that generated them.

**Components of an HMM**:
An HMM $\lambda$ is defined by:
1.  **A set of $N$ hidden states ($Q = \{q_1, q_2, \dots, q_N\}$)**: For POS tagging, these would be the tags in the tag set.
2.  **A set of $M$ observation symbols ($V = \{v_1, v_2, \dots, v_M\}$)**: For POS tagging, these are the words in the vocabulary.
3.  **State Transition Probability Distribution ($A = \{a_{ij}\}$)**:
    $a_{ij} = P(state_t = q_j | state_{t-1} = q_i)$
    This is the probability of transitioning from hidden state $q_i$ at time $t-1$ to hidden state $q_j$ at time $t$.
4.  **Observation Emission Probability Distribution ($B = \{b_j(k)\}$)**:
    $b_j(o_k) = P(observation_t = v_k | state_t = q_j)$
    This is the probability of emitting (observing) symbol $v_k$ when the system is in hidden state $q_j$.
5.  **Initial State Distribution ($\pi = \{\pi_i\}$)**:
    $\pi_i = P(state_1 = q_i)$
    This is the probability that the sequence starts in hidden state $q_i$.

**Three Fundamental Problems for HMMs**:
1.  **Likelihood Problem**: Given an HMM $\lambda = (A, B, \pi)$ and an observation sequence $O = (o_1, o_2, \dots, o_T)$, what is the probability of observing this sequence, $P(O|\lambda)$? (Solved by the **Forward algorithm**).
2.  **Decoding Problem**: Given an HMM $\lambda$ and an observation sequence $O$, what is the most likely sequence of hidden states $Q = (q_1, q_2, \dots, q_T)$ that produced these observations? (Solved by the **Viterbi algorithm**).
3.  **Learning Problem**: Given an observation sequence $O$ (and possibly corresponding state sequences in supervised learning), how do we adjust the model parameters $A, B, \pi$ to maximize $P(O|\lambda)$? (Solved by direct counting for supervised learning, or the **Forward-Backward (Baum-Welch) algorithm** for unsupervised learning).

---
## Decoding with HMMs: The Viterbi Algorithm ️

The most common task in NLP using HMMs is **decoding**: finding the most probable sequence of hidden states (e.g., POS tags) given a sequence of observations (e.g., words). The **Viterbi algorithm** accomplishes this efficiently using dynamic programming.

Imagine a **trellis** (a grid-like diagram) where columns represent time steps (positions in the observation sequence) and rows represent the possible hidden states. Each path through this trellis from start to end represents a possible sequence of hidden states. The Viterbi algorithm finds the single path (state sequence) through this trellis that has the highest probability.

**Viterbi Algorithm Steps**:
1.  **Initialization**: For each state $q_i$ at time $t=1$:
    $V_{1,i} = \pi_i \cdot b_i(o_1)$ (probability of starting in state $q_i$ and emitting the first observation $o_1$)
    Store path: $ptr_{1,i} = 0$
2.  **Recursion**: For each time step $t = 2, \dots, T$ and for each state $q_j$:
    $V_{t,j} = \max_{i} [V_{t-1,i} \cdot a_{ij}] \cdot b_j(o_t)$
    This calculates the maximum probability of any path ending in state $q_j$ at time $t$ and emitting observation $o_t$.
    Store backpointer: $ptr_{t,j} = \arg\max_{i} [V_{t-1,i} \cdot a_{ij}]$ (the previous state $i$ that maximized this probability).
3.  **Termination**:
    $P(\text{best path}) = \max_i V_{T,i}$
    $q_T^* = \arg\max_i V_{T,i}$ (the best final state)
4.  **Path (State Sequence) Backtracking**: Starting from $q_T^*$, follow the backpointers $ptr_{t,j}$ backward to $t=1$ to reconstruct the most likely sequence of hidden states.

The Viterbi algorithm guarantees finding the most probable sequence of hidden states.

---
## Learning HMM Parameters: The Forward-Backward Algorithm
To use an HMM, we first need to define its parameters: $A$ (transition probabilities), $B$ (emission probabilities), and $\pi$ (initial state probabilities).

* **Supervised Learning**: If we have a training corpus where both the observation sequences and their corresponding hidden state sequences are labeled, estimating HMM parameters is straightforward using **Maximum Likelihood Estimation (MLE)**. We simply count the occurrences of transitions, emissions, and initial states and normalize them:
    * $a_{ij} = \frac{\text{Count}(q_i \rightarrow q_j)}{\text{Count}(q_i)}$
    * $b_j(o_k) = \frac{\text{Count}(o_k \text{ emitted from } q_j)}{\text{Count}(q_j)}$
    * $\pi_i = \frac{\text{Count(sequence starts with } q_i)}{\text{Total number of sequences}}$

* **Unsupervised Learning (Baum-Welch Algorithm)**: Often, we only have observation sequences (e.g., raw text) without the hidden state sequences. In this case, we use the **Baum-Welch algorithm**, which is a specific instance of the **Expectation-Maximization (EM) algorithm**, to iteratively estimate the HMM parameters. It involves:
    * **Forward Probability ($\alpha_t(i)$)**: $P(o_1, o_2, \dots, o_t, \text{state at } t \text{ is } q_i | \lambda)$. This is the probability of seeing the partial observation sequence up to time $t$ and ending in state $q_i$.
    * **Backward Probability ($\beta_t(i)$)**: $P(o_{t+1}, o_{t+2}, \dots, o_T | \text{state at } t \text{ is } q_i, \lambda)$. This is the probability of observing the remainder of the sequence, given that we are in state $q_i$ at time $t$.
    The algorithm alternates between an E-step (calculating expected counts of transitions and emissions using $\alpha$ and $\beta$ values based on current parameters) and an M-step (re-estimating parameters A, B, and $\pi$ using these expected counts) until the parameters converge.

---
## Implementation Issues for HMMs

* **Underflow**: When multiplying many small probabilities (as in the Forward, Backward, and Viterbi algorithms), the resulting values can become extremely small and exceed the precision of standard floating-point numbers, leading to underflow (becoming zero).
    * **Solution**: Perform calculations using **log probabilities**. Instead of multiplying probabilities, sum their logarithms. $\log(P_1 \cdot P_2) = \log P_1 + \log P_2$.
* **Scaling**: In the Forward-Backward algorithm, the raw $\alpha$ and $\beta$ values can also become very small. To prevent underflow and maintain numerical stability, **scaling factors** are introduced at each time step during their computation. These scaling factors are then used to correctly normalize the re-estimated parameters.

---
## Tagging: Assigning Labels to Words in Text ️

**Tagging** is the process of assigning a category (or "tag") from a predefined set to each token (usually a word) in a text. The most common tagging task is **Part-of-Speech (POS) Tagging**.
* **Example**: `The/DT cat/NN sat/VBD on/IN the/DT mat/NN ./.`
  Here, DT (determiner), NN (noun, singular), VBD (verb, past tense), IN (preposition), and . (punctuation) are POS tags.

### Tag Sets

A **tag set** is the collection of labels used for a particular tagging task. For POS tagging English:
* **Penn Treebank Tag Set**: Widely used in NLP research, contains about 45-48 tags (e.g., NN, NNS, NNP, VB, VBD, VBG, VBN, VBP, VBZ, JJ, RB, IN, DT).
* **Brown Corpus Tag Set**: An older, larger set with more fine-grained distinctions.

The choice of tag set affects the complexity and granularity of the tagging task.

### Morphology and Lemmatization in Tagging

Understanding word structure (**morphology**) and identifying the base or dictionary form of a word (**lemma**) can significantly aid tagging, especially for unknown words or words with multiple inflectional forms.
* **Morphological clues**: Suffixes like "-ing," "-ed," "-s," "-ly" often indicate specific POS tags (e.g., VBG, VBD, NNS/VBZ, RB respectively).
* **Lemmatization**: Reducing words like "running," "ran," "runs" to the lemma "run" can help generalize statistical models.

### Tagging Methods: Diverse Approaches

1.  **Manually Designed Rules and Grammars (Rule-Based Taggers)**:
    * Early POS taggers were often rule-based. Linguists would write rules like: "If a word ends in '-ed' and is preceded by a pronoun, it's likely a past tense verb."
    * Can be accurate if rules are meticulously crafted.
    * **Drawbacks**: Labor-intensive to create, brittle (fail on unseen patterns), hard to maintain and scale to new languages or domains.

2.  **Statistical Methods**: These methods learn statistical patterns from large, (often manually) tagged corpora.
    * **HMM Tagging**: A very common and effective statistical approach.
        * **Hidden States**: The POS tags.
        * **Observations**: The words in the sentence.
        * **Transition Probabilities $P(Tag_j | Tag_i)$**: The probability that tag $j$ follows tag $i$. Estimated from sequences of tags in the training corpus.
        * **Emission Probabilities $P(Word_k | Tag_j)$**: The probability that word $k$ is generated given tag $j$. Estimated from word-tag pairings in the training corpus.
        * **Supervised HMM Tagging**: Parameters $(A, B, \pi)$ are estimated from a hand-tagged corpus using MLE. Given a new sentence, the Viterbi algorithm finds the most likely sequence of tags.
        * **Unsupervised HMM Tagging**: Attempts to learn parameters using the Forward-Backward (Baum-Welch) algorithm from untagged text. Generally less accurate than supervised methods.

3.  **Statistical Transformation Rule-Based Tagging (e.g., Brill Tagger)**:
    * This is a hybrid approach that learns a set of transformation rules.
    * **Process**:
        1.  Start with an initial (often simple) tagging of the text (e.g., assign each word its most frequent tag from a dictionary).
        2.  Iteratively learn transformation rules that correct the most errors on a training corpus. Rules are typically of the form: "Change tag X to tag Y if context C holds" (e.g., "IF previous_tag is DT AND current_tag is NN THEN change current_tag to VB").
        3.  Rules are ranked by their error-reduction score, and the best rules are applied.
    * It combines some of the linguistic insight of rule-based systems with the data-driven learning of statistical models.
    *(This is a known method; its explicit detailed algorithm is not present in the provided PDFs but the concept fits under statistical NLP methods.)*

4.  **Maximum Entropy (MaxEnt) Methods**:
    * **Maximum Entropy Tagging (MEMMs - Maximum Entropy Markov Models)**: These are **discriminative** sequence models, as opposed to HMMs which are generative.
    * MEMMs directly model the conditional probability of the current tag given the previous tag(s) and the entire observation sequence: $P(tag_i | tag_{i-1}, observation\_sequence)$.
    * This allows them to incorporate a rich set of overlapping **features** from the observation sequence (e.g., current word, surrounding words, prefixes, suffixes, capitalization, word shape) without strong independence assumptions between these features.

5.  **Feature-Based Tagging**:
    * Modern statistical taggers, especially discriminative ones like MEMMs and Conditional Random Fields (CRFs, another powerful sequence model not detailed here but related), heavily rely on **feature engineering**.
    * Features can include:
        * Current word identity.
        * Previous and next words.
        * Prefixes and suffixes (e.g., first 3 letters, last 3 letters).
        * Capitalization patterns (e.g., Is_Capitalized, Is_All_Caps).
        * Word shape (e.g., "Xx-xxxx" for "Varicella-zoster", "dddd" for a 4-digit number).
        * Previously assigned tags in the sequence.
    * These features are used by a classifier (like MaxEnt) to predict the tag for each word.

---
## Evaluation Methodology for Tagging
How do we measure the performance of a POS tagger?
* **Accuracy**: The most common metric. It's the percentage of words that are tagged correctly.
    $Accuracy = \frac{\text{Number of correctly tagged words}}{\text{Total number of words in the test set}}$
* **Precision and Recall (Per Tag)**: For specific tags, especially rare ones or those critical for a particular application, precision and recall can provide more insight than overall accuracy.
    * For a tag $T_k$:
        * **Precision($T_k$)**: Of all words the system tagged as $T_k$, what fraction were actually $T_k$?
        * **Recall($T_k$)**: Of all words that are truly $T_k$ in the gold standard, what fraction did the system correctly tag as $T_k$?
    * F1-score can then be calculated for each tag.

**Examples from Tagging Performance**:
* State-of-the-art supervised POS taggers for English typically achieve accuracies in the range of **96-97%** on standard datasets like the Penn Treebank.
* Performance on **unknown words** (words not seen during training) is a key challenge and a differentiator between taggers. Good morphological analysis and feature engineering help here.

---
## Results on Tagging and Various Natural Languages
* While English POS tagging is a relatively well-solved problem with high accuracies, tagging performance can vary significantly for other languages.
* **Morphologically rich languages** (e.g., Turkish, Finnish, Arabic) pose greater challenges due to the large number of word forms for each lemma.
* The availability of large, accurately **annotated corpora** is a major factor. Languages with fewer resources generally have lower tagging accuracies.
* The choice of **tag set** also influences complexity and performance. Some languages might require more fine-grained tag sets to capture important grammatical distinctions.
* Despite these challenges, HMMs, MEMMs, CRFs, and increasingly deep learning models (like RNNs and Transformers) have been successfully applied to POS tagging for a wide variety of natural languages.

---
## Conclusion: Decoding Language's Sequential Secrets

Hidden Markov Models provide a robust and mathematically elegant framework for modeling sequential data, making them a cornerstone of statistical NLP, especially for tasks like Part-of-Speech tagging. The Viterbi algorithm allows us to efficiently find the most likely sequence of hidden states, while the Forward-Backward algorithm enables learning HMM parameters even from unlabeled data.

Tagging, by assigning grammatical labels to words, is a fundamental first step in deeper language understanding. From classic rule-based systems to sophisticated statistical and feature-rich models like HMMs and MEMMs, the field has made remarkable progress in accurately and efficiently performing this task across many languages, paving the way for more advanced NLP applications.
