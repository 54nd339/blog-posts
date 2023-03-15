---
title: Learning Theory and Bayesian Learning - What Makes a Problem Learnable
description: The theory behind the algorithms - identification in the limit, query learning, and the PAC model's bounds on how much data learning takes; boosting weak learners into strong ones; and the Bayesian view with maximum-likelihood and MAP estimation, the regularisation connection, and Bayesian belief networks.
date: 2023-03-15
draft: false
slug: /artificial-intelligence/theorotical-ml
tags:
  - Artificial Intelligence
  - Machine Learning
  - Probabilistic Models
---

The [supervised algorithms](/citadel/artificial-intelligence/ml) work in practice, but they leave questions open. Which problems can be learned at all? How many examples does it take to be confident the model is close to right? Can you glue together classifiers that are each only slightly better than a coin flip and get one that's reliably accurate?

Two bodies of theory answer these: **learnability theory**, which formalises when and how efficiently learning is possible, and **Bayesian learning**, which treats learning as updating probabilities over hypotheses. This post covers both.

## Identification in the limit

The earliest formal model of learnability, from E. Mark Gold in 1967, and influential in the study of language acquisition. A learner is fed an infinite stream of examples of a target concept — say, sentences of a language. After each one it proposes a hypothesis — a grammar. It **identifies the concept in the limit** if, after some finite number of examples, its hypothesis becomes correct and never changes again.

The model's value is in what it rules out: it shows which concept classes are learnable from an infinite data stream, and how much depends on the *kind* of data — many classes are learnable from positive *and* negative examples but not from positive examples alone.

## Query learning

Instead of passively receiving examples, the learner can interrogate an **oracle**:

- **Membership query** — "is this specific instance an example of the concept?"
- **Equivalence query** — "is my current hypothesis the true concept?" If not, the oracle returns a counterexample.

Active, targeted questioning can make concepts learnable that are hard or impossible to learn from random examples, and can sharply cut the number of examples needed.

## The PAC model

**Probably Approximately Correct** learning, from Leslie Valiant in 1984, is the standard framework for the *cost* of learning. A concept class is **PAC-learnable** if there's an algorithm that, given a reasonable number of training examples, outputs a hypothesis $h$ that — with probability at least $1 - \delta$ (the "probably") — has error at most $\epsilon$ (the "approximately correct") on unseen data.

- $\epsilon$ — the accuracy parameter: the largest error you'll tolerate.
- $\delta$ — the confidence parameter: the chance you'll allow of exceeding that error.

Two costs are analysed: **sample complexity** (how many examples) and **computational complexity** (how much computation). A class is **efficiently PAC-learnable** if both are polynomial in $1/\epsilon$, $1/\delta$, the size of the target concept, and the data dimension. The related notion of a hypothesis class's *capacity* — VC dimension — is covered in [deep networks](/citadel/artificial-intelligence/deep-networks).

## Boosting

Learnability theory raises a striking question: if a concept is only *weakly* learnable — some algorithm beats chance by a hair — is it *strongly* learnable? **Boosting** answers yes, and constructively.

**AdaBoost** trains weak learners in sequence. Each round, it raises the weight of the training examples the previous learners got wrong, forcing the next learner to focus on the hard cases. The final classifier is a weighted vote of all the weak learners, with better-performing ones weighted more. Boosting reduces **bias** (the ensemble is more expressive than any member) and often **variance** too. It reappears as a practical ensemble method in [advanced machine learning](/citadel/artificial-intelligence/advanced-ml).

## Bayesian learning

Bayesian learning treats inference as updating beliefs with **Bayes' theorem**:

$$P(H \mid D) = \frac{P(D \mid H)\,P(H)}{P(D)}$$

$P(H \mid D)$ is the **posterior** (belief in hypothesis $H$ after seeing data $D$), $P(D \mid H)$ the **likelihood**, $P(H)$ the **prior** (belief before the data), $P(D)$ the **evidence**. You start with a prior and revise it as data arrives.

## Maximum likelihood and MAP

**Maximum likelihood estimation** (MLE) picks the parameters that make the observed data most probable:

$$\hat{\theta}_{MLE} = \arg\max_{\theta}\; P(D \mid \theta)$$

usually maximised as the **log-likelihood** $\ln P(D \mid \theta)$, since the log is monotonic and turns products into sums. MLE is what fits a [logistic regression](/citadel/artificial-intelligence/ml).

**Maximum a posteriori** (MAP) estimation adds a prior over the parameters:

$$\hat{\theta}_{MAP} = \arg\max_{\theta}\; P(D \mid \theta)\,P(\theta)$$

If the prior $P(\theta)$ is uniform, MAP reduces to MLE. And a non-uniform prior is exactly **regularisation**: a Gaussian prior on the weights gives an L2 penalty (ridge), a Laplace prior gives L1 (lasso). The regulariser you add to a loss function is a prior in disguise, and it does the same job — penalising extreme parameters to curb overfitting.

## Bayesian belief networks

A **Bayesian belief network** (BBN, or Bayesian network) represents a joint probability distribution compactly as a **directed acyclic graph**:

- **Nodes** are random variables.
- **Edges** are direct probabilistic influences — an arrow $X \to Y$ means $X$ directly affects $Y$.
- Each node carries a **conditional probability table** giving its distribution for every combination of its parents' values. A node with no parents just holds its prior.

The graph structure encodes **conditional independence**: a variable is independent of its non-descendants given its parents. That's what makes the network compact — instead of a full joint table exponential in the number of variables, you store one small table per node.

BBNs support several kinds of inference over the same model: predicting an outcome from evidence, diagnosing likely causes from observed effects, and **explaining away** — when one confirmed cause makes alternative causes of the same effect less likely. Both the network's structure and its tables can be learned from data.

## The one idea to keep

Learnability theory says learning has a price measured in examples and computation, and the PAC model makes that price precise — polynomial in your accuracy and confidence demands, or the class isn't efficiently learnable. Bayesian learning gives the other half: every model has a prior, whether you write one down or not, and the regularisation term you reach for to stop overfitting *is* that prior. Bayesian networks are what you get when you make the dependency structure of those priors explicit.
