---
title: Neural Network Architectures - CNN to Transformer to GPT
description: The lineage behind modern AI in one pass. Why CNNs suit images and RNNs suit sequences, what the attention mechanism changed in 2017, and how a pre-trained transformer becomes a chat model through RLHF.
date: 2024-12-02
draft: false
slug: /interview/neural-network
tags:
  - System Design
  - Interview Prep
  - Machine Learning
---

CNN, RNN, GAN, attention, transformer, GPT — the acronyms stack up, but they form a clear lineage, each architecture added to fix a specific limit of the last. This traces that line from the vision and sequence models of the 1990s and 2000s to the transformer, and then to how a raw language model is turned into a conversational one.

## The early architectures

**CNN (convolutional neural network).** Built for grid data — pixels. A convolutional layer slides a small filter across the image, reusing the same weights everywhere, which gives it **translation invariance** (a cat is a cat wherever it appears) and far fewer parameters than a fully connected layer. Dominant for image recognition, object detection, and vision generally.

**RNN (recurrent neural network).** Built for sequences — text, time series. It processes one element at a time, carrying a hidden state that acts as a memory of what came before. Its weakness is long-range: gradients vanish or explode over many steps, so early RNNs forgot distant context. Gated variants (LSTM, GRU) patched this but not fully, and the strict left-to-right processing made them slow to train.

**GAN (generative adversarial network).** Two networks in a contest: a **generator** makes fake samples, a **discriminator** tries to tell fake from real. Each pushes the other, and the generator learns to produce convincingly realistic output — used heavily for image synthesis.

## Attention and the transformer

CNNs and RNNs both struggled to relate distant parts of a long input. The fix, from the 2017 paper *Attention Is All You Need*:

**Attention** lets the model, for each position in the output, weigh every position in the input by relevance and pull from the important ones directly — no fixed-size hidden state bottleneck, and distance costs nothing.

**The transformer** is built almost entirely on attention. Because it attends to all positions at once rather than stepping through them, it processes a whole sequence **in parallel**, which collapses training time versus an RNN. The cost is that self-attention is quadratic in sequence length. Transformers became the backbone of NLP and then of vision and audio too.

## From transformer to GPT

The transformer made it practical to train very large models on very large corpora — the era of **large language models**.

**GPT (generative pre-trained transformer)** is a **decoder-only** transformer.

- **Pre-training** — on a large slice of internet text (GPT-3.5: on the order of 300 billion tokens), with one objective: predict the next token. That alone teaches grammar, facts, and style. A pre-trained model can *continue* text, but it is not yet built to answer questions.
- **Fine-tuning into a chat model** — three steps:
  1. **Supervised fine-tuning** — train on a dataset of prompts paired with good responses, so the model learns the question-answer shape.
  2. **Reward modelling** — collect prompts with several model answers each, have humans rank them, and train a separate model to predict those rankings.
  3. **RLHF** — reinforcement learning from human feedback: the model generates responses, the reward model scores them, and an RL algorithm (PPO) adjusts the model toward higher-scoring behaviour.

**BERT** took the same transformer breakthrough in a different direction — an **encoder-only** model trained to fill in masked words using context from *both* sides, aimed at understanding tasks rather than generation.

## Where it landed

- **AlphaCode** — a transformer pre-trained on public code that DeepMind reported ranking around the median among human entrants in competitive-programming contests.
- **GPT-4** — a multimodal model (image and text input), a much larger context window than its predecessors, strong scores on academic benchmarks, and more work on refusing unsafe requests.

Deployed systems add a **content-moderation** layer on both the user input and the model output to enforce safety policy.

## Each step removed one bottleneck

The through-line is narrow: CNNs added weight sharing so vision did not need astronomical parameter counts; RNNs added state so order mattered; attention removed the state bottleneck so long-range relationships were cheap; the transformer removed sequential processing so training could parallelise; and RLHF added a step that aligns a next-token predictor with what a person actually wants back. Nothing here is a clean break — each is the previous idea with one limit lifted.
