---
title: NLP Grammars & Parsing Algorithms - Understanding Sentence Structure
description: Explore how Natural Language Processing deciphers sentence structure using grammars and parsing algorithms, from foundational concepts like CFGs to probabilistic parsing with PCFGs and evaluation using treebanks.
date: 2023-07-08
draft: false
slug: /pensieve/ai-ml/grammar-parsing
tags:
  - AI/ML
  - CS Basics
---

Hey language aficionados and AI explorers! We've seen how NLP systems handle words and their individual properties. But how do machines make sense of the way words combine to form phrases and sentences? This is the domain of **syntactic parsing**, a core task in NLP that aims to uncover the grammatical structure of text.

Today, we'll delve into:
* The basics of **parsing** and why it's crucial.
* **Generative Grammars** like Context-Free Grammars (CFGs) that define language rules.
* An overview of early **non-statistical parsing algorithms**.
* The role of **Treebanks** in developing and evaluating parsers.
* The shift towards **Probabilistic Parsing** and Probabilistic Context-Free Grammars (PCFGs) to handle ambiguity.

Let's break down the architecture of language!

---
## Introduction to Parsing
**Parsing** (or syntactic analysis) is the process of analyzing a string of symbols—typically words in a sentence—according to the rules of a formal grammar. The goal is to determine the grammatical structure of the sentence, often represented as a **parse tree**.

**Why is parsing important in NLP?**
Understanding the syntactic structure of a sentence is a crucial step towards understanding its meaning. It helps in:
* **Information Extraction**: Identifying who did what to whom.
* **Machine Translation**: Ensuring grammatically correct translations by understanding source sentence structure.
* **Question Answering**: Interpreting questions and finding relevant information in structured text.
* **Sentiment Analysis**: Understanding how different parts of a sentence contribute to overall sentiment.

In NLP, there are two main ways to view syntactic structure:
1.  **Constituency (Phrase Structure)**: Organizes words into nested groups or "constituents" like Noun Phrases (NP), Verb Phrases (VP), etc.
2.  **Dependency Structure**: Shows which words depend on (modify or are arguments of) which other words through directed links.

Today, we'll primarily focus on constituency parsing.

---
## Generative Grammars: The Rules of Language Construction
A **generative grammar** is a formal system of rules that aims to define all and only the grammatically "correct" sentences of a language. It provides a mechanism to generate an infinite number of sentences from a finite set of rules.

### Context-Free Grammars (CFGs)

The most common type of generative grammar used in NLP for describing phrase structure is the **Context-Free Grammar (CFG)**. A CFG is formally defined as a 4-tuple $G = (T, N, S, R)$:
* **$T$**: A set of **terminal symbols** (e.g., the words of the language like "cat", "saw", "the").
* **$N$**: A set of **non-terminal symbols** (e.g., syntactic categories like `S` for sentence, `NP` for noun phrase, `VP` for verb phrase).
* **$S \in N$**: A designated **start symbol** (usually `S`).
* **$R$**: A set of **rules** or **productions** of the form $X \rightarrow \gamma$, where $X \in N$ is a non-terminal, and $\gamma$ is a sequence of symbols from $(N \cup T)^*$. This means the right-hand side can be a mix of non-terminals and terminals.

**Example CFG rules**:
* `S → NP VP` (A sentence can be a Noun Phrase followed by a Verb Phrase)
* `NP → DT N` (A Noun Phrase can be a Determiner followed by a Noun)
* `VP → V NP` (A Verb Phrase can be a Verb followed by a Noun Phrase)
* `DT → the`
* `N → cat`
* `V → chased`

These rules allow us to generate sentences like "the cat chased the dog" and assign a tree structure to them. In NLP, we often distinguish between grammatical rules (expanding non-terminals) and lexical rules (expanding pre-terminals like `N` to terminals like "cat").

---
## Properties of Grammars: Regular vs. Context-Free
Different types of grammars have different expressive powers and are suited for different tasks.

* **Regular Grammars (and Regular Languages)**:
    * These are the simplest form in the Chomsky hierarchy.
    * They can be recognized by **finite automata** and described by **regular expressions**.
    * Regular grammars are suitable for simpler NLP tasks like tokenization, morphological analysis (recognizing word patterns like prefixes/suffixes), or simple pattern matching in text.
    * **Limitation**: They cannot handle nested structures or long-range dependencies common in natural language (e.g., center-embedded clauses like "The mouse the cat chased ran.").

* **Context-Free Grammars (CFGs)**:
    * More powerful than regular grammars.
    * They can describe **nested structures** (e.g., `NP` containing another `PP` which contains an `NP`) effectively, making them suitable for representing the phrase structure of most natural languages.
    * CFGs form the basis for most syntactic parsing algorithms in NLP.
    * **Limitations**: While powerful, standard CFGs still face challenges with some complex linguistic phenomena, such as cross-serial dependencies (found in languages like Swiss German) and ensuring context-sensitive agreement (e.g., subject-verb number agreement, which often requires additional mechanisms or features beyond pure CFG rules).

---
## Overview of Non-statistical Parsing Algorithms: Early Approaches

Before statistical methods became dominant, parsing was primarily done using algorithms based directly on grammar rules. These non-statistical parsers aim to find *any* valid parse for a sentence according to a given grammar.

### Simple Top-Down Parser with Backtracking

* **Approach**: Starts with the grammar's start symbol (e.g., `S`) and tries to expand it using grammar rules to eventually derive the input sentence.
* **Process**:
    1.  Begin with the start symbol.
    2.  Recursively apply rules to expand the leftmost non-terminal in the current sentential form.
    3.  If a terminal symbol is generated, match it against the current input word. If it matches, advance the input pointer.
    4.  If a non-terminal has multiple expansion rules, try them one by one.
    5.  **Backtracking**: If a chosen rule leads to a dead end (e.g., cannot match an input word, or no more rules apply to complete the parse), the parser "backtracks" to the last choice point and tries an alternative rule.
* **Challenges**:
    * **Left Recursion**: Rules like `NP → NP PP` can cause infinite loops in simple top-down parsers.
    * **Inefficiency**: Can perform a lot of repeated work on sub-problems and explore many failing paths.
    * **Ambiguity**: If multiple parses exist, it might find one and stop, or it might explore all, which can be very slow.

### Shift-Reduce Parser (Bottom-Up)

* **Approach**: Starts with the input sentence (sequence of words) and tries to apply grammar rules in reverse to reduce the sequence to the start symbol.
* **Components**: Uses a **stack** to hold processed symbols and an **input buffer** for the remaining words.
* **Main Operations**:
    1.  **Shift**: Move the next token from the input buffer onto the top of the stack.
    2.  **Reduce**: If the top symbols on the stack match the right-hand side (RHS) of a grammar rule, replace these symbols on the stack with the rule's left-hand side (LHS) non-terminal.
    3.  **Accept**: If the stack contains only the start symbol and the input buffer is empty, the parse is successful.
    4.  **Error**: If no action can be performed.
* **Challenges**:
    * **Conflicts**: Ambiguity in the grammar can lead to:
        * **Shift/Reduce Conflict**: The parser doesn't know whether to shift another token or reduce the symbols already on the stack.
        * **Reduce/Reduce Conflict**: The top of the stack matches the RHS of multiple rules.
    * Requires strategies to resolve these conflicts (often by looking ahead in the input or using precedence rules).

*(While the provided `NLP.pdf` focuses more on statistical parsing techniques, these classical algorithms form the conceptual basis for understanding parsing challenges like ambiguity and search.)*

---
## Treebanks and Treebanking: Data for Parsing
The advent of large, annotated linguistic resources, known as **corpora**, revolutionized NLP. For parsing, **Treebanks** are particularly crucial.
* **What is a Treebank?** A treebank is a text corpus in which each sentence has been manually or semi-automatically annotated with its syntactic structure, typically in the form of a parse tree.
* **Example**: The **Penn Treebank (PTB)**, containing parsed sentences from the Wall Street Journal, is a widely used resource in NLP research.

**Importance of Treebanks**:
* **Training Data**: They provide the supervised data needed to train statistical parsers.
* **Evaluation Data**: They serve as a "gold standard" against which the output of parsers can be evaluated.
* **Linguistic Research**: They are invaluable for studying language phenomena, such as the frequency of different grammatical constructions or word argument structures.
* **Broad Coverage**: They help in developing parsers that can handle a wide variety of sentences found in real-world text, rather than just a limited set of textbook examples.
* **Frequencies and Distributional Information**: They provide empirical data on the probabilities of different grammar rules and lexical items, which is essential for probabilistic parsing.

---
## Evaluation of Parsers: How Good is the Parse?
To measure the performance of constituency parsers, we compare the parse trees they produce for test sentences against the "gold standard" trees from a treebank. Common metrics include:

* **Labeled Precision (LP)**: The percentage of constituents (brackets with labels like `NP`, `VP`) in the parser's output that exactly match a constituent in the gold standard tree.
    $LP = \frac{\text{Number of correct constituents in parser output}}{\text{Total number of constituents in parser output}}$
* **Labeled Recall (LR)**: The percentage of constituents in the gold standard tree that were correctly identified by the parser.
    $LR = \frac{\text{Number of correct constituents in parser output}}{\text{Total number of constituents in gold standard tree}}$
* **Labeled F1-Score**: The harmonic mean of Labeled Precision and Labeled Recall, providing a single measure of performance.
    $F1 = \frac{2 \times LP \times LR}{LP + LR}$

Additionally, **Tagging Accuracy** (the accuracy of the POS tags assigned to the words) is often reported as pre-terminal nodes are part of the parse tree.

---
## Probabilistic Parsing: Handling Ambiguity with Probabilities

Natural language is inherently ambiguous. A single sentence can often have multiple valid syntactic interpretations (parses) according to a given grammar. For example, "Fed raises interest rates 0.5% in effort to control inflation" had 36 parses with a minimal grammar and 592 with a simple 10-rule grammar in one experiment. Classical parsers might struggle to choose the correct one or get bogged down exploring too many options.

**Probabilistic parsing** addresses this by assigning probabilities to different parse trees for a sentence and selecting the most probable one.

### Probabilistic Context-Free Grammars (PCFGs)

The most common framework for probabilistic parsing is the **Probabilistic Context-Free Grammar (PCFG)**. A PCFG is a CFG where each rule $A \rightarrow \beta$ is associated with a conditional probability $P(\beta | A)$ – the probability of expanding non-terminal $A$ into the sequence $\beta$.
* For any non-terminal $A$, the sum of probabilities for all rules expanding $A$ must be 1: $\sum_{\beta} P(A \rightarrow \beta) = 1$.

**Probability of a Parse Tree ($P(t)$)**:
The probability of a specific parse tree $t$ is the product of the probabilities of all the grammar rules used to generate that tree.
$P(t) = \prod_{rule (A \rightarrow \beta) \in t} P(A \rightarrow \beta)$

**Probability of a String ($P(s)$)**:
The probability of a string (sentence) $s$ is the sum of the probabilities of all possible parse trees $t$ that have $s$ as their yield (i.e., their terminal symbols spell out $s$):
$P(s) = \sum_{t \text{ such that yield}(t)=s} P(t)$

**The Task: Finding the Best Parse**
Given a sentence $s$, the goal of a probabilistic parser is usually to find the parse tree $t^*$ that is most probable:
$t^* = \arg\max_t P(t|s)$
Using Bayes' rule, $P(t|s) = \frac{P(s|t)P(t)}{P(s)}$. Assuming $P(s|t)$ is 1 if $yield(t)=s$ and 0 otherwise (the tree must generate the string exactly), this simplifies to finding the tree $t^*$ that maximizes $P(t)$ for the given sentence $s$.

**Example of PCFG probabilities for "people fish tanks with rods"**:
A sentence like "people fish tanks with rods" can have multiple parses (e.g., "fish" as a verb or "fish tanks" as a compound noun). A PCFG assigns probabilities to each rule (e.g., `VP → V NP` might have a higher probability than `NP → N N` for certain words). By multiplying these rule probabilities, we can find the probability of each complete parse tree. For example, one parse $t_1$ might have $P(t_1) = 0.0008232$ and another $t_2$ might have $P(t_2) = 0.00024696$. The parser would select $t_1$ as the more likely parse.

### The CKY Algorithm for PCFGs (Cocke-Kasami-Younger)

The **CKY algorithm** is a dynamic programming algorithm widely used for parsing with PCFGs (and non-probabilistic CFGs). To use CKY with PCFGs for finding the most probable parse (the Viterbi parse):
1.  The PCFG must be converted to **Chomsky Normal Form (CNF)**. In CNF, all grammar rules are of the form $A \rightarrow B C$ (a non-terminal expands to two non-terminals) or $A \rightarrow w$ (a non-terminal expands to a terminal word).
2.  The algorithm builds a table (often called a chart) indexed by `[start_position][end_position][NonTerminal]`. Each cell `score[i][j][A]` stores the maximum probability that non-terminal `A` spans the substring of the input sentence from word $i$ to word $j$.
3.  **Initialization**: For each word $w_i$ at position $i$ in the sentence, if there's a rule $A \rightarrow w_i$, then `score[i][i+1][A] = P(A \rightarrow w_i)`. This fills the diagonal of the chart.
4.  **Iteration**: For increasing span lengths (from 2 up to the sentence length $N$):
    For each possible starting position `begin`:
    For each possible split point `split` between `begin` and `end`:
    For each rule $A \rightarrow B C$:
    `prob = score[begin][split][B] * score[split][end][C] * P(A \rightarrow B C)`
    If `prob` is greater than the current `score[begin][end][A]`, update the score and store backpointers indicating that $A$ spanning `begin` to `end` was formed by $B$ (spanning `begin` to `split`) and $C$ (spanning `split` to `end`).
5.  Unary rules (like $A \rightarrow B$) can be handled within the loops by checking if applying a unary rule increases the probability of a non-terminal spanning a given substring.
6.  The probability of the most likely parse for the entire sentence will be `score[0][N][S]` (where S is the start symbol). The actual tree can be reconstructed by following the backpointers.

The CKY algorithm has a time complexity of $O(N^3 \cdot |G|)$, where $N$ is the length of the sentence and $|G|$ is the size of the grammar (number of rules).

---
## Conclusion: Deciphering the Blueprint of Language

Grammars and parsing algorithms are fundamental to NLP's ability to interpret and understand human language beyond the level of individual words. While early rule-based parsers laid the groundwork, the inherent ambiguity of language necessitated the move towards probabilistic approaches like PCFGs. Combined with data-driven insights from treebanks, these statistical models allow us to find the most likely syntactic structures for sentences, a critical step for many advanced NLP applications. The journey from simple CFG rules to sophisticated lexicalized and latent variable parsing models continues to enhance our machines' capacity to process language with greater accuracy and nuance.
