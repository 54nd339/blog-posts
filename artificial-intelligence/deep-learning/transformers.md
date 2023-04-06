---
title: Attention, Transformers, and Large Language Models
description: Recurrent networks process a sequence one step at a time, which is slow and forgetful over long spans. Attention lets every position look directly at every other position in one parallel operation. Stack that into a transformer, train it to predict the next token on enough text, and you get GPT.
date: 2023-04-06
draft: false
slug: /artificial-intelligence/transformers
tags:
  - Artificial Intelligence
  - NLP
  - Deep Learning
---

To translate a sentence, a [recurrent network](/citadel/artificial-intelligence/generalisation) reads it word by word, compressing everything seen so far into a fixed-size hidden state. Two problems: it can't parallelize (step `t` needs step `t−1`'s output), and information from early words has to survive being squeezed through dozens of steps to influence a decision at the end — the long-range dependency problem that even LSTMs only partly solve.

**Attention** removes both limits. Instead of passing information hand-to-hand down a chain, every position computes, in one parallel operation, a weighted look at every other position — pulling in exactly the context it needs regardless of distance. The **transformer** (2017, "Attention Is All You Need") is a network built almost entirely from stacked attention, and scaled up on enough text it's GPT, Claude, and every modern language model.

## Scaled dot-product attention

Each input token is a vector. From it, three projections produce a **query**, a **key**, and a **value**:

- To decide how much position `i` should attend to position `j`, take the dot product of `i`'s query with `j`'s key. Big dot product = relevant.
- Scale by `1/√d` (so the dot products don't grow with dimension and saturate the softmax), then softmax across all `j` to get attention weights that sum to 1.
- The output for position `i` is the weighted sum of all positions' **values**, using those weights.

$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d}}\right)V$$

For a sequence of length `n` this is an `n × n` matrix of weights — every token can look at every token — computed as two matrix multiplies, which GPUs love. The cost is that it's **O(n²)** in sequence length, which is why context windows were historically limited and why a lot of research targets that quadratic term.

## Multi-head attention

One attention operation forces a single notion of "relevance." **Multi-head** attention runs several attention operations in parallel, each with its own query/key/value projections into a smaller subspace, then concatenates their outputs. Different heads specialize — one tracks syntactic dependencies, one tracks coreference ("it" → which noun), one attends locally, one globally. Cheap, because each head works in `d/h` dimensions.

## Self- vs cross-attention, and masking

- **Self-attention** — queries, keys, and values all come from the same sequence; tokens attend to each other.
- **Cross-attention** — queries from one sequence (the decoder's output so far), keys and values from another (the encoder's representation of the source). Used in translation-style encoder-decoder models.
- **Causal masking** — for a model that generates text left to right, position `i` must not attend to positions after it (it'd be cheating — looking at the answer). Mask those weights to `−∞` before the softmax. This is what makes GPT-style models *autoregressive*.

## The transformer block

The full block, repeated `N` times (12 for the original, 96+ for large models):

1. Multi-head self-attention over the input.
2. A residual connection (add the input back) and **layer normalization**. (Modern models put the norm *before* the sublayer — "pre-norm" — for more stable training.)
3. A position-wise **feed-forward network** — two linear layers with a nonlinearity, applied to each position independently, typically expanding to 4× the width and back. This is where much of the model's parameter count and "knowledge" lives.
4. Another residual + layer norm.

Residual connections let gradients flow through very deep stacks; layer norm keeps activations well-scaled.

## Positional information

Attention is **permutation-invariant** — shuffle the tokens and the weighted sums are the same, because there's no notion of order in a dot product. So position must be injected:

- **Sinusoidal** (original) — add fixed sine/cosine patterns of varying frequency to the token embeddings.
- **Learned** — a trainable embedding per position (BERT).
- **Rotary (RoPE)** — rotate the query and key vectors by an angle proportional to position, so their dot product depends on *relative* position. The current standard; extrapolates better to longer sequences.
- **ALiBi** — a linear distance penalty added to attention scores.

## Model families

- **Encoder-only (BERT)** — bidirectional self-attention (every token sees every other), trained with **masked language modelling** (blank out 15% of tokens, predict them). Great for understanding tasks — classification, retrieval, NER — where you have the whole input at once.
- **Decoder-only (GPT, Llama, Claude)** — causal self-attention, trained to **predict the next token**. Great for generation, and it turns out that "predict the next token really well over a huge corpus" produces a model that can also summarize, translate, reason, and code, because all of those appear in the training text.
- **Encoder-decoder (T5, original transformer)** — encode the input bidirectionally, decode the output autoregressively with cross-attention to the encoding. Natural for translation and other seq-to-seq tasks.

## Tokenization

Text is split into **subword** tokens by **byte-pair encoding** (or SentencePiece / WordPiece): start from characters, repeatedly merge the most frequent adjacent pair, building a vocabulary of ~30k–100k tokens where common words are one token and rare words are a few. A fixed vocabulary that never has an out-of-vocabulary word, and it's why models "count letters" badly — they see tokens, not characters.

## Scaling and efficiency

- **Scaling laws** — loss falls predictably as a power law in model size, data, and compute; the **Chinchilla** result showed most large models were under-trained on data relative to their parameter count, shifting the field toward more tokens per parameter.
- **KV cache** — during generation, keys and values for past tokens don't change, so cache them; each new token only computes its own query against the cache. Essential for fast inference, and its memory cost is what limits batch size and context length.
- **FlashAttention** — compute attention without materializing the `n × n` matrix in slow memory, by tiling and recomputing — a large speedup and memory saving with identical results.
- **Mixture of Experts** — replace the feed-forward layer with many "expert" FFNs and a router that sends each token to a few; total parameters go up, compute per token stays flat.

## Adaptation

- **Prompting / in-context learning** — give examples in the input; no weight changes.
- **Supervised fine-tuning (SFT)** — continue training on curated instruction-response pairs.
- **RLHF / DPO** — align outputs to human preferences (see [reinforcement learning](/citadel/artificial-intelligence/reinforcement-learning)).
- **Parameter-efficient tuning (LoRA)** — freeze the base model, train small low-rank adapter matrices — a few million trainable parameters instead of billions.
- **Retrieval-augmented generation** — fetch relevant documents ([vector search](/citadel/tech/vector-search)) and put them in the context, so the model answers from provided facts rather than parametric memory.

## The one idea to keep

Attention lets every position in a sequence attend directly to every other in one parallel matrix operation — `softmax(QKᵀ/√d)V` — replacing the recurrent network's slow, forgetful hand-to-hand chain. A transformer stacks multi-head attention with feed-forward layers, residual connections, and layer norm; add causal masking and it generates text left to right. Train that to predict the next token over enough of the internet, and next-token prediction alone yields a model that summarizes, translates, and reasons — which is then aligned with SFT and RLHF.
