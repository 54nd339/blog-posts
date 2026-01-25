---
title: Words, Classes, and Lexicons - The Atoms of Natural Language Processing
description: A deep dive into how NLP systems understand and organize words - exploring word classes, lexicography, statistical measures of word association like Mutual Information, their application in NLP tasks, and the complexities involved.
date: 2023-07-06
draft: false
slug: /pensieve/ai-ml/lexicon
tags:
  - AI/ML
  - CS Basics
---

Hey language lovers and tech explorers! We've explored some high-level Natural Language Processing (NLP) tasks, but what are the fundamental building blocks that enable computers to "understand" language? It all starts with **words** – their properties, how they group together, and how we can build structured knowledge about them.

Today, we're zooming into the fascinating world of words and lexicons in NLP. We'll cover:
* **Word Classes** (like Parts of Speech) and the art of **Lexicography**.
* Statistical measures like **Mutual Information**, the **t-score**, and the **Chi-square test** to understand word associations.
* The role of these concepts in various **NLP tasks**.
* How we estimate parameters for models using word features.
* Algorithms for automatically grouping words into classes.
* The inherent complexities and some handy programming tips.

Let's dissect the atoms of language!

---
## Word Classes and Lexicography: Organizing the Vocabulary
Words are not just arbitrary strings of characters; they carry rich syntactic and semantic information. NLP systems need to organize this information to process language effectively.

### Word Classes (Parts of Speech - POS)

One of the most fundamental ways to categorize words is by their **Part of Speech (POS)**. These classes group words based on their grammatical properties and how they function in sentences. You're likely familiar with the basic ones:
* **Nouns (N, NN, NNS, NNP)**: Representing entities (e.g., *cat, snow, IBM, Italy*).
* **Verbs (V, VB, VBD, VBG, VBN, VBZ)**: Representing actions or states (e.g., *see, registered, joining*).
* **Adjectives (JJ)**: Describing nouns (e.g., *old, independent*).
* **Adverbs (RB)**: Modifying verbs, adjectives, or other adverbs (e.g., *slowly, never*).

Word classes can be broadly divided into:
* **Open Classes**: Nouns, verbs, adjectives, adverbs. New words are constantly added to these classes (e.g., new tech terms, slang).
* **Closed Classes (Function Words)**: Determiners (e.g., *the, a, an*), pronouns (e.g., *she, he, I*), prepositions (e.g., *on, under, to*), conjunctions (e.g., *and, or*). These classes have a relatively fixed and small number of members.

**Why is POS Tagging Important?**
Many words are ambiguous and can belong to multiple POS classes. For example, "back" can be:
* An adjective (JJ): *The **back** door*.
* A noun (NN): *On my **back***.
* An adverb (RB): *Win the voters **back***.
* A verb (VB): *Promised to **back** the bill*.
**POS tagging** is the process of assigning the correct POS tag to each word in a sentence given its context. This is crucial for many downstream NLP tasks like parsing, named entity recognition, and information extraction, as it helps resolve ambiguities and understand the grammatical structure.

### Lexicography and Lexicons in NLP

**Lexicography** is the art and science of compiling dictionaries. In NLP, this translates to creating and using **lexicons**, which are digital repositories of words and their associated linguistic information. A lexicon might store:
* The word itself (lemma).
* Its possible POS tags.
* Word senses or meanings.
* Morphological variants.
* Other syntactic or semantic features.

A prime example of a large-scale lexical database extensively used in NLP is **WordNet**. WordNet groups English words into sets of synonyms called **synsets**, provides short definitions (glosses), and records various semantic relations between these synsets (like hyponymy/hypernymy – ISA relations).

---
## Measuring Word Associations: Beyond Simple Co-occurrence
To understand language, we often need to know how strongly words are associated with each other. Are they synonyms? Do they frequently appear together (collocations)? Are they related to a particular topic? Statistical measures help quantify these associations.

### Pointwise Mutual Information (PMI)

**Pointwise Mutual Information (PMI)** is a measure of association used in information theory and statistics. It quantifies how much more two events (or in NLP, two words or a word and a context) co-occur than if they were independent.

The PMI between two words, $word_1$ and $word_2$, is defined as:
$PMI(word_1, word_2) = \log_2 \frac{P(word_1, word_2)}{P(word_1)P(word_2)}$

* If $PMI > 0$: The words co-occur more often than expected by chance (positively associated).
* If $PMI < 0$: The words co-occur less often than expected (negatively associated or they tend to avoid each other).
* If $PMI = 0$: The words co-occur as expected by chance (independent).

Probabilities like $P(word_1)$, $P(word_2)$, and $P(word_1, word_2)$ are typically estimated from their frequencies in a large text corpus. For instance, $P(word_1, word_2)$ can be estimated by the count of $word_1$ occurring in the context of $word_2$ .

**Positive PMI (PPMI)**: Since negative PMI values can be problematic for some applications (indicating repulsion rather than attraction), often only positive PMI values are used, or negative values are set to zero:
$PPMI(word_1, word_2) = \max(0, PMI(word_1, word_2))$

PMI and PPMI are widely used in NLP for tasks like finding collocations, measuring word similarity based on distributional contexts, and feature weighting in sentiment analysis (e.g., how much more does a phrase co-occur with "excellent" versus "poor").

### The t-score

The t-score is another statistical measure often used in NLP to identify significant word associations, particularly for discovering collocations (like "strong coffee" or "kick the bucket"). It assesses whether the observed co-occurrence frequency of two words is significantly higher than what would be expected if the words were independent, taking into account the sample size and variance. Essentially, it's a variant of the t-test applied to word co-occurrence data.

### The Chi-square ($\chi^2$) Test

The Chi-square ($\chi^2$) test is a statistical hypothesis test commonly used in NLP for feature selection and assessing independence. For word associations, it can determine if the observed co-occurrence of two words (or a word and a feature/class) is significantly different from the co-occurrence expected if they were independent. A higher $\chi^2$ value typically indicates a stronger association (or dependence). It is frequently used in text categorization to find words that are good indicators of a particular category.

---
## Word Classes for NLP Tasks: Practical Applications ️

Word classes, especially POS tags, are fundamental features that fuel a wide range of NLP applications:
* **Parsing**: POS tags are essential input for syntactic parsers, which aim to determine the grammatical structure of sentences. Knowing that "book" is a noun helps determine its role in "I book a flight" vs. "I read a book."
* **Named Entity Recognition (NER)**: Identifying sequences of words that constitute named entities (like persons, organizations, locations) often relies on POS tags as features. For example, sequences of proper nouns (NNP) often indicate named entities.
* **Information Extraction (IE)**: Extracting specific pieces of information or relations often uses patterns defined over POS tags and words. For instance, extracting `PERSON, POSITION of ORG` might use patterns involving POS tags.
* **Sentiment Analysis**: Adjectives (JJ) are strong indicators of sentiment. Identifying adjectives and their polarity is a common strategy.
* **Language Modeling**: Instead of (or in addition to) word N-grams, language models can use POS tag N-grams or mixed N-grams (word + POS tag) to achieve better generalization, especially for rare words.
* **Machine Translation**: POS tags help in disambiguating words and ensuring grammatical correctness in the target language.

Beyond manually defined POS tags, **induced word classes** (clusters of words grouped based on similar distributional behavior in a corpus) can also serve similar purposes, providing a data-driven way to categorize words.

---
## Parameter Estimation for Models with Word Features/Classes

Many NLP models are probabilistic, meaning their behavior is governed by parameters (e.g., probabilities) that need to be estimated from training data.
* **Maximum Likelihood Estimation (MLE)** is a common principle for estimating these parameters. It involves choosing parameter values that make the observed training data most probable.
    * For **N-gram language models**, the MLE for $P(w_i|w_{i-1})$ is $\frac{C(w_{i-1}, w_i)}{C(w_{i-1})}$.
    * For **Naïve Bayes text classifiers**, the MLE for $P(word|class)$ (the likelihood of a word given a class) is typically $\frac{\text{count}(word, class)}{\text{count}(class) + \alpha |\text{Vocabulary}|}$ with smoothing, or $\frac{\text{count}(word, class)}{\sum_{w' \in V} \text{count}(w', class)}$ for the basic MLE without smoothing.
* **Smoothing**: A critical aspect of parameter estimation, especially with sparse data (common in NLP). When an event (like a specific word or N-gram) doesn't appear in the training data, MLE assigns it a zero probability. Smoothing techniques (like Add-one/Laplace, Good-Turing, Interpolation, and Kneser-Ney) reallocate some probability mass from seen events to unseen ones to avoid these zero probabilities.

---
## Inducing Word Classes: Partitioning Algorithms (Clustering)
While POS tags are often pre-defined (though they can also be learned), NLP also benefits from **inducing word classes** automatically from large text corpora. This is an unsupervised learning task where words are grouped based on their **distributional similarity** – the idea that words appearing in similar contexts tend to have similar meanings or grammatical functions.

1.  **Representing Words as Vectors**: Each word can be represented as a vector based on the contexts it appears in. These contexts can be:
    * Neighboring words (e.g., in a fixed window).
    * Syntactic dependencies (e.g., "object of verb 'eat'").
    The vector components can be raw co-occurrence counts, tf-idf scores, or Pointwise Mutual Information (PMI)/PPMI values.

2.  **Clustering Algorithms**: Once words are represented as vectors, clustering algorithms can be applied to partition them into groups (classes).
    * **K-Means Clustering**: A common partitioning algorithm that groups data points (word vectors) into K clusters by minimizing the sum of squared distances to cluster centroids. This can group words that are used in similar ways distributionally.
    * Other clustering methods like hierarchical clustering (agglomerative or divisive) can also be used to discover a taxonomy of word classes.

These induced classes can then be used as features in various NLP tasks, similar to how POS tags are used, potentially capturing semantic nuances not covered by traditional POS tags.

---
## Complexity Issues of Word Classes
Working with word classes, whether pre-defined like POS tags or automatically induced, comes with its own set of complexities:

* **Ambiguity**: The most significant challenge. A single word form can belong to multiple POS classes (e.g., "book" as a noun or a verb). Resolving this ambiguity correctly is crucial for downstream processing. About 40% of word tokens in the Brown corpus are ambiguous.
* **Granularity of Classes**: Determining the right level of detail for word classes is difficult.
    * A small set of coarse POS tags might be too general and miss important distinctions.
    * A very large set of fine-grained tags (like the full Penn Treebank tagset with around 45 tags) can capture more nuances but also leads to data sparsity issues and makes the learning task harder.
* **Unknown Words (Out-of-Vocabulary - OOV)**: How should an NLP system assign a class to a word it has never encountered during training? This requires robust methods for handling OOV words, often involving morphological analysis, looking at word shapes, or assigning a special `<UNK>` token probability.
* **Computational Cost**:
    * POS tagging, especially with sophisticated sequence models, can be computationally intensive.
    * Inducing word classes from large corpora using clustering algorithms can also be very resource-demanding.
    * Using word classes (especially fine-grained ones) as features in other NLP models can increase the dimensionality and complexity of those models.

---
## Programming Tricks & Tips for Lexical NLP
When working with words and lexicons in NLP applications, here are a few practical tips:

* **Master Regular Expressions**: They are indispensable for text preprocessing, tokenization, pattern matching (e.g., for Hearst patterns in IS-A extraction), and extracting simple lexical features from words.
* **Choose Efficient Data Structures**:
    * Use hash maps (dictionaries in Python) for storing lexicons, word counts, and N-gram models for fast lookups.
    * Tries (prefix trees) can be efficient for storing vocabularies and supporting prefix-based searches.
    * For very large N-grams, consider specialized structures or pruning (e.g., storing only N-grams with count > threshold).
* **Consistent Text Normalization is Key**: Establish a clear pipeline for text normalization early on. This includes decisions about case folding, handling of punctuation, and whether to use stemming or lemmatization. Consistency here prevents many downstream issues.
* **Handle Large Scale**: For very large vocabularies or corpora:
    * Map words to unique integer IDs to save space and speed up comparisons.
    * Consider techniques like quantizing probabilities or Huffman coding for efficient storage if dealing with web-scale N-grams.
* **Leverage Existing Libraries & Toolkits**: Don't reinvent the wheel! Libraries like NLTK, spaCy, and Stanford CoreNLP offer robust, pre-built tools for common lexical tasks such as tokenization, POS tagging, lemmatization, and NER. This saves development time and often ensures high-quality processing.

---
## Conclusion: The Foundation of Understanding

Words and their organized representations in lexicons form the very foundation upon which more complex Natural Language Processing tasks are built. By understanding word classes, measuring associations between words using statistical techniques like PMI, and employing methods to automatically induce or utilize these categories, we empower machines to move closer to a nuanced understanding of human language. While challenges like ambiguity and scale persist, the continuous development of new models and computational techniques ensures that NLP will keep unlocking new frontiers in human-computer interaction and information access.
