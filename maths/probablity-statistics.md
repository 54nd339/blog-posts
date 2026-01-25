---
title: Statistics and Probability - Navigating Uncertainty and Making Sense of Data
description: Explore the essential tools of statistics and probability, from fundamental definitions of chance and events to understanding data distributions and variability. This post delves into key probability theorems, common distributions like Binomial, Poisson, and Normal, and methods for describing and analyzing data sets.
date: 2023-01-03
draft: false
slug: /pensieve/maths/statistics-probability
tags:
  - Mathematics
  - Statistics
---

Our world is brimming with variability and uncertainty. From predicting the weather to understanding market trends, making financial investments, or assessing medical treatments, the ability to quantify chance and interpret data is indispensable. Statistics and probability are the twin disciplines that provide the rigorous framework for this understanding. Probability theory allows us to model and analyze random phenomena, while statistics offers methods to collect, analyze, interpret, and draw conclusions from data. This exploration covers the fundamental concepts that underpin these vital fields.

---
## The Language of Chance: Basic Definitions – Setting the Stage

To discuss probability formally, we need a precise vocabulary.

* **Experiment:** A process or action that leads to an observable outcome.
* **Random Experiment:** An experiment whose outcome cannot be predicted with certainty before it is performed, although the set of all possible outcomes may be known. Examples include tossing a coin, rolling a die, or measuring the lifetime of a component.
* **Sample Space ($S$):** The set of all possible distinct outcomes of a random experiment. Each outcome is a **sample point**.
    * Example: For a single coin toss, $S = \{\text{Head, Tail}\}$. For a die roll, $S = \{1, 2, 3, 4, 5, 6\}$.
* **Event ($E$):** An event is any subset of the sample space $S$. It represents one or more possible outcomes of an experiment.
    * **Simple Event (or Elementary Event):** An event consisting of a single outcome (a single sample point).
    * **Compound Event:** An event consisting of more than one outcome.
        Example: When rolling a die, $E = \{\text{getting an even number}\} = \{2, 4, 6\}$ is a compound event.
* **Trial:** A single performance of a random experiment.

---
## Playing with Events: Algebra of Events – Combining Possibilities

Just like numbers, events can be combined using set operations. Understanding this "algebra of events" is crucial. Let $A$ and $B$ be events in a sample space $S$.

* **Complement of an Event ($A'$ or $A^c$ or $\bar{A}$):** The set of all outcomes in $S$ that are not in $A$. It represents the event that "$A$ does not occur."
    $P(A') = 1 - P(A)$.
* **Union of Events ($A \cup B$):** The set of all outcomes that are in $A$ or in $B$ or in both. It represents the event "$A$ OR $B$ (or both) occur."
* **Intersection of Events ($A \cap B$):** The set of all outcomes that are in both $A$ and $B$. It represents the event "$A$ AND $B$ both occur."
* **Equally Likely Events:** Events that have the same theoretical probability of occurring. For instance, when rolling a fair die, each of the six outcomes is equally likely.
* **Mutually Exclusive Events (or Disjoint Events):** Two events $A$ and $B$ are mutually exclusive if they cannot occur simultaneously. This means their intersection is the empty set ($A \cap B = \emptyset$). If $A$ occurs, $B$ cannot, and vice-versa.
    For mutually exclusive events, $P(A \cup B) = P(A) + P(B)$.
* **Exhaustive Events:** A set of events $E_1, E_2, \ldots, E_n$ is exhaustive if their union forms the entire sample space ($E_1 \cup E_2 \cup \ldots \cup E_n = S$). This means that at least one of these events must occur when the experiment is performed.

---
## The Foundation: Axiomatic Approach to Probability – The Rules of the Game

The modern theory of probability is built upon a set of axioms, usually attributed to Andrey Kolmogorov. These axioms provide a rigorous mathematical foundation.

Let $S$ be a sample space, and let $P(A)$ denote the probability of an event $A \subseteq S$.
1.  **Non-negativity:** For any event $A$, $P(A) \ge 0$. (Probability cannot be negative).
2.  **Normalization:** The probability of the entire sample space $S$ is 1. $P(S) = 1$. (Something must happen).
3.  **Additivity for Mutually Exclusive Events:** If $A_1, A_2, \ldots, A_n, \ldots$ is a sequence of mutually exclusive events (i.e., $A_i \cap A_j = \emptyset$ for $i \neq j$), then the probability of their union is the sum of their individual probabilities:
    $$ P(A_1 \cup A_2 \cup A_3 \cup \dots) = P(A_1) + P(A_2) + P(A_3) + \dots $$
    For a finite number of mutually exclusive events, $P(\bigcup_{i=1}^n A_i) = \sum_{i=1}^n P(A_i)$.

**Some consequences derived from these axioms:**
* Probability of the impossible event (empty set $\emptyset$): $P(\emptyset) = 0$.
* Range of probability: For any event $A$, $0 \le P(A) \le 1$.
* Probability of the complement: $P(A') = 1 - P(A)$.

---
## Quantifying Chance: Probability – The Mathematics of Likelihood

* **Classical Definition of Probability:** If a random experiment can result in $N$ mutually exclusive, exhaustive, and equally likely outcomes, and $N_A$ of these outcomes are favorable to an event $A$, then the probability of event $A$ is:
    $$ P(A) = \dfrac{\text{Number of outcomes favorable to A}}{\text{Total number of possible outcomes}} = \dfrac{N_A}{N} $$

* **Addition Theorem of Probability:**
    * For any two events $A$ and $B$:
        $$ P(A \cup B) = P(A) + P(B) - P(A \cap B) $$
    * If $A$ and $B$ are mutually exclusive ($A \cap B = \emptyset$, so $P(A \cap B) = 0$):
        $$ P(A \cup B) = P(A) + P(B) $$
    * For three events $A, B, C$:
        $P(A \cup B \cup C) = P(A) + P(B) + P(C) - P(A \cap B) - P(A \cap C) - P(B \cap C) + P(A \cap B \cap C)$.

* **Conditional Probability ($P(A|B)$):**
    The probability of event $A$ occurring, given that event $B$ has already occurred, is called the conditional probability of $A$ given $B$.
    $$ P(A|B) = \dfrac{P(A \cap B)}{P(B)}, \quad \text{provided } P(B) \neq 0 $$

* **Multiplication Theorem of Probability (for Compound Events):**
    From the definition of conditional probability, we can find the probability of the simultaneous occurrence of two events:
    $$ P(A \cap B) = P(A)P(B|A) = P(B)P(A|B) $$

* **Independent Events:**
    Two events $A$ and $B$ are **independent** if the occurrence (or non-occurrence) of one event does not affect the probability of the occurrence of the other event.
    * Mathematical condition for independence:
        $$ P(A \cap B) = P(A)P(B) $$
    * If $A$ and $B$ are independent, then:
        $P(A|B) = P(A)$ (if $P(B) \neq 0$)
        $P(B|A) = P(B)$ (if $P(A) \neq 0$)

---
## Reasoning Backwards: Baye's Theorem – Updating Beliefs

Baye's theorem is fundamental for updating probabilities based on new evidence.

* **Partition of a Sample Space:** A set of events $E_1, E_2, \ldots, E_n$ forms a partition of the sample space $S$ if:
    1.  They are mutually exclusive: $E_i \cap E_j = \emptyset$ for all $i \neq j$.
    2.  They are exhaustive: $E_1 \cup E_2 \cup \ldots \cup E_n = S$.

* **Law of Total Probability:** If $E_1, E_2, \ldots, E_n$ form a partition of $S$, then for any event $A$:
    $$ P(A) = \sum_{i=1}^n P(A \cap E_i) = \sum_{i=1}^n P(A|E_i)P(E_i) $$

* **Baye's Theorem:** Allows us to find the probability of a particular event $E_i$ from a partition, given that event $A$ has occurred.
    $$ P(E_i|A) = \dfrac{P(A|E_i)P(E_i)}{P(A)} $$
    Using the Law of Total Probability for $P(A)$ in the denominator:
    $$ P(E_i|A) = \dfrac{P(A|E_i)P(E_i)}{\sum_{j=1}^n P(A|E_j)P(E_j)} $$
    $P(E_i)$ are called prior probabilities, $P(A|E_i)$ are likelihoods, and $P(E_i|A)$ are posterior probabilities.

---
## Common Patterns of Chance: Probability Distributions – Modeling Randomness

A **random variable** is a variable whose value is a numerical outcome of a random phenomenon. Random variables can be discrete (taking on a finite or countably infinite number of values) or continuous (taking on any value in an interval).
The **probability distribution** of a random variable describes how probabilities are distributed over the values of the random variable.

* For a **discrete random variable** $X$, its probability distribution is often given by a probability mass function (PMF), $P(X=x_i) = p_i$, such that $p_i \ge 0$ and $\sum p_i = 1$.
* For a **continuous random variable** $X$, its probability distribution is described by a probability density function (PDF), $f(x)$, such that $f(x) \ge 0$, $P(a \le X \le b) = \int_a^b f(x)dx$, and $\int_{-\infty}^{\infty} f(x)dx = 1$.
* **Mean and Variance of a Distribution:**
    * The **mean** or **expected value** ($E[X]$ or $\mu$) is the long-run average value of the random variable.
        * For a discrete variable: $\mu = E[X] = \sum x_i P(X=x_i)$.
        * For a continuous variable: $\mu = E[X] = \int_{-\infty}^{\infty} x f(x)dx$.
    * The **variance** ($\text{Var}(X)$ or $\sigma^2$) measures the spread of the distribution around the mean.
        * $\sigma^2 = E[(X-\mu)^2] = E[X^2] - (E[X])^2$.
    * **Jointly Distributed Random Variables:** We often consider two or more random variables together, described by a **joint probability distribution**, $p(x,y)$ or $f(x,y)$.
    * **Covariance:** A measure of the joint variability of two random variables, $X$ and $Y$. It describes how they change together.
        $$ \text{Cov}(X,Y) = E[(X-\mu_X)(Y-\mu_Y)] = E[XY] - E[X]E[Y] $$
        A positive covariance indicates that $X$ and $Y$ tend to move in the same direction, while a negative covariance indicates they move in opposite directions.
    * **Correlation:** A normalized version of covariance that measures the strength and direction of the *linear* relationship between two random variables. The correlation coefficient $\rho$ is always between -1 and 1.
        $$ \rho_{XY} = \frac{\text{Cov}(X,Y)}{\sigma_X \sigma_Y} $$


Here are some key probability distributions:
#### Discrete Distributions
* **Binomial Distribution**
    * Describes the number of successes in a fixed number ($n$) of independent **Bernoulli trials** (each trial has only two outcomes, e.g., success or failure).
    * Parameters: $n$ (number of trials) and $p$ (probability of success on a single trial).
    * Probability Mass Function (PMF): The probability of exactly $k$ successes in $n$ trials is:
        $$ P(X=k) = \binom{n}{k} p^k (1-p)^{n-k} \quad \text{for } k = 0, 1, 2, \ldots, n $$
    * Mean (Expected Value): $\mu = np$
    * Variance: $\sigma^2 = np(1-p)$
* **Negative Binomial Distribution**
    * Models the number of failures before a specified number of successes occurs in a sequence of independent Bernoulli trials.
    * Parameters: $r$ (number of successes) and $p$ (probability of success on each trial).
    * Probability Mass Function (PMF): The probability of having $k$ failures before the $r$-th success is:
        $$ P(X=k) = \binom{k+r-1}{r-1} p^r (1-p)^k \quad \text{for } k = 0, 1, 2, \ldots $$
    * Mean: $\mu = \dfrac{r(1-p)}{p}$
    * Variance: $\sigma^2 = \dfrac{r(1-p)}{p^2}$
* **Poisson Distribution**
    * Models the number of times an event occurs in a fixed interval of time or space, given that these events occur with a known constant mean rate and independently of the time since the last event.
    * Parameter: $\lambda$ (lambda), the average number of events in the interval.
    * Probability Mass Function (PMF): The probability of exactly $k$ events occurring in the interval is:
        $$ P(X=k) = \dfrac{e^{-\lambda} \lambda^k}{k!} \quad \text{for } k = 0, 1, 2, \ldots $$
    * Mean: $\mu = \lambda$
    * Variance: $\sigma^2 = \lambda$
    * It can serve as an approximation to the binomial distribution when $n$ is large and $p$ is small, with $\lambda = np$.
* **Hypergeometric Distribution**
    * Models the number of successes in a sequence of draws from a finite population without replacement.
    * Parameters: $N$ (population size), $K$ (number of successes in the population), and $n$ (number of draws).
    * Probability Mass Function (PMF): The probability of drawing exactly $k$ successes in $n$ draws is:
        $$ P(X=k) = \dfrac{\binom{K}{k} \binom{N-K}{n-k}}{\binom{N}{n}} \quad \text{for } k = 0, 1, \ldots, \min(K, n) $$
    * Mean: $\mu = n \cdot \dfrac{K}{N}$
    * Variance: $\sigma^2 = n \cdot \dfrac{K}{N} \cdot \left(1 - \dfrac{K}{N}\right) \cdot \dfrac{N-n}{N-1}$

### Continuous Distributions
* **Normal Distribution (Gaussian Distribution)**
    * A continuous probability distribution that is bell-shaped and symmetric. It is arguably the most important distribution in statistics due to the Central Limit Theorem.
    * Parameters: $\mu$ (mean) and $\sigma^2$ (variance).
    * Probability Density Function (PDF):
        $$ f(x) = \dfrac{1}{\sigma\sqrt{2\pi}} e^{-\dfrac{1}{2}\left(\dfrac{x-\mu}{\sigma}\right)^2} \quad \text{for } -\infty < x < \infty $$
    * The **Standard Normal Distribution** has $\mu=0$ and $\sigma^2=1$, often denoted by $Z$. Any normal variable $X \sim N(\mu, \sigma^2)$ can be standardized to $Z = (X-\mu)/\sigma$.
* **Exponential Distribution**
    * Models the time until an event occurs, such as the time until failure of a component or the time between arrivals in a queue.
    * Parameter: $\lambda$ (rate parameter, which is the inverse of the mean).
    * Probability Density Function (PDF):
        $$ f(x) = \lambda e^{-\lambda x} \quad \text{for } x \ge 0 $$
    * Mean: $\mu = \dfrac{1}{\lambda}$
    * Variance: $\sigma^2 = \dfrac{1}{\lambda^2}$
* **Uniform Distribution**
    * A distribution where all outcomes are equally likely within a certain range.
    * **Discrete Uniform Distribution:** For a finite set of outcomes $\{x_1, x_2, \ldots, x_n\}$, each outcome has a probability of $\dfrac{1}{n}$.
    * **Continuous Uniform Distribution:** For a continuous variable $X$ uniformly distributed over the interval $[a, b]$, the PDF is:
        $$ f(x) = \dfrac{1}{b-a} \quad \text{for } a \le x \le b $$
    * Mean: $\mu = \dfrac{a+b}{2}$
    * Variance: $\sigma^2 = \dfrac{(b-a)^2}{12}$

---
## Describing Data: Measures of Central Tendency – The "Average" Story

When analyzing data, we often want to find a typical or central value.
* **Mean (Arithmetic Mean):** The sum of all values divided by the number of values.
    * Ungrouped data ($x_1, x_2, \ldots, x_N$): $\bar{x} = \dfrac{\sum_{i=1}^N x_i}{N}$.
    * Grouped data (values $x_i$ with frequencies $f_i$): $\bar{x} = \dfrac{\sum_{i=1}^k f_i x_i}{\sum_{i=1}^k f_i} = \dfrac{\sum f_i x_i}{N_{total}}$. (For continuous grouped data, $x_i$ is the midpoint of the $i^{th}$ class).

* **Median:** The middle value of a dataset that has been ordered from least to greatest.
    * Ungrouped data: If $N$ is odd, the median is the $(N+1)/2$-th value. If $N$ is even, the median is the average of the $N/2$-th and $(N/2 + 1)$-th values.
    * Grouped data (continuous): Median $= L + \left(\dfrac{N/2 - C_f}{f_m}\right)h$
        where $L$ = lower limit of the median class, $N$ = total frequency, $C_f$ = cumulative frequency of the class preceding the median class, $f_m$ = frequency of the median class, $h$ = width of the median class.

* **Mode:** The value that appears most frequently in a dataset. A dataset can have one mode (unimodal), two modes (bimodal), or more (multimodal), or no mode if all values are unique.
    * Grouped data (continuous): Mode $= L + \left(\dfrac{f_1 - f_0}{2f_1 - f_0 - f_2}\right)h$
        where $L$ = lower limit of the modal class, $f_1$ = frequency of the modal class, $f_0$ = frequency of the class preceding the modal class, $f_2$ = frequency of the class succeeding the modal class, $h$ = width of the modal class.

---
## Measuring Spread: Measures of Dispersion – How Scattered is the Data?

Measures of dispersion describe how spread out or varied the values in a dataset are.

* **Variance ($\sigma^2$ for population, $s^2$ for sample):** The average of the squared differences from the Mean.
    * Ungrouped population data: $\sigma^2 = \dfrac{\sum_{i=1}^N (x_i - \mu)^2}{N}$ (where $\mu$ is the population mean).
        For a sample: $s^2 = \dfrac{\sum_{i=1}^n (x_i - \bar{x})^2}{n-1}$ (using $n-1$ for an unbiased estimator).
    * Grouped population data: $\sigma^2 = \dfrac{\sum f_i (x_i - \mu)^2}{\sum f_i} = \dfrac{\sum f_i x_i^2}{\sum f_i} - \mu^2$.

* **Standard Deviation ($\sigma$ for population, $s$ for sample):** The square root of the variance. It measures the typical distance of values from the mean, in the original units of the data.
    $$ \sigma = \sqrt{\text{Variance}} $$

* **Mean Deviation (MD):** The average of the absolute differences of the values from a central point (usually the mean or median).
    * About the mean (ungrouped): $MD_{\bar{x}} = \dfrac{\sum_{i=1}^N |x_i - \bar{x}|}{N}$.
    * About the mean (grouped): $MD_{\bar{x}} = \dfrac{\sum f_i |x_i - \bar{x}|}{\sum f_i}$.
    Similar formulas apply for mean deviation about the median.

---
## Inferential Statistics - Drawing Conclusions from Data

Inferential statistics uses sample data to make generalizations about an entire population.

### Random Sampling
The foundation of inference is **random sampling**, where each member of the population has an equal chance of being selected. This helps ensure the sample is representative of the population, allowing for valid generalizations. A **statistic** (e.g., sample mean $\bar{x}$) is a value calculated from a sample, used to estimate a population **parameter** (e.g., population mean $\mu$).

### Distribution of the Sample Mean ($\bar{X}$)
If we were to take many random samples of size $n$ from a population, the sample means themselves would form a distribution.
    * **Central Limit Theorem (CLT):** A fundamental theorem stating that for a sufficiently large sample size ($n \ge 30$ is a common rule of thumb), the sampling distribution of the sample mean $\bar{X}$ will be approximately normal, regardless of the shape of the population distribution. Its mean will be the population mean $\mu$, and its standard deviation (called the standard error) will be $\sigma/\sqrt{n}$.

### Point Estimation of Parameters
* **Point Estimate:** A single value (a statistic) used to estimate an unknown population parameter. For example, the sample mean $\bar{x}$ is a point estimate for the population mean $\mu$.
* **Methods of Point Estimation:**
    * **Method of Moments:** Equates sample moments (like the sample mean) to the corresponding population moments and solves for the unknown parameters.
    * **Maximum Likelihood Estimation (MLE):** Finds the parameter values that maximize the likelihood function, i.e., the values that make the observed sample data most probable.

### Statistical Intervals (Confidence Intervals)
A **confidence interval** provides a range of values which is likely to contain an unknown population parameter with a certain level of confidence.
* **Properties:** The width of a confidence interval depends on the confidence level (a higher confidence level like 99% gives a wider interval than 95%), the sample standard deviation, and the sample size (a larger sample size gives a narrower, more precise interval).
* **Large-Sample Confidence Interval for Population Mean ($\mu$):**
    **Derivation:** By the CLT, the standardized sample mean $Z = \frac{\bar{X}-\mu}{\sigma/\sqrt{n}}$ follows a standard normal distribution. For a $(1-\alpha)$ confidence level, we find the critical value $z_{\alpha/2}$ such that $P(-z_{\alpha/2} \le Z \le z_{\alpha/2}) = 1-\alpha$.
    $-z_{\alpha/2} \le \frac{\bar{X}-\mu}{\sigma/\sqrt{n}} \le z_{\alpha/2}$.
    Rearranging the inequality to isolate $\mu$:
    $\bar{X} - z_{\alpha/2} \frac{\sigma}{\sqrt{n}} \le \mu \le \bar{X} + z_{\alpha/2} \frac{\sigma}{\sqrt{n}}$.
    * **Formula:** The interval is $\bar{x} \pm z_{\alpha/2} \frac{\sigma}{\sqrt{n}}$. If the population standard deviation $\sigma$ is unknown, the sample standard deviation $s$ is used as an estimate for large $n$.
* **Large-Sample Confidence Interval for Population Proportion ($p$):**
    $$ \hat{p} \pm z_{\alpha/2} \sqrt{\frac{\hat{p}(1-\hat{p})}{n}} $$
    where $\hat{p}$ is the sample proportion.

### Testing Hypotheses
**Hypothesis testing** is a formal procedure for using sample data to decide between two competing claims about a population parameter.
1.  **State Hypotheses:** A **null hypothesis** ($H_0$), representing the status quo or no effect, and an **alternative hypothesis** ($H_a$), representing the claim to be tested.
2.  **Calculate a Test Statistic:** A value calculated from the sample data that measures how far the sample estimate is from the value stated in the null hypothesis.
3.  **Determine p-value:** The probability of observing a test statistic as extreme as, or more extreme than, the one calculated, assuming the null hypothesis is true.
4.  **Make a Decision:** If the p-value is smaller than a predetermined significance level ($\alpha$, e.g., 0.05), we reject the null hypothesis in favor of the alternative.

* **Inferences Based on a Single Sample:** These tests compare a single sample statistic to a known or hypothesized population value (e.g., a one-sample t-test for a population mean).
* **Inferences Based on Two Samples:** These tests compare statistics from two different samples to see if the populations they come from are different (e.g., a two-sample t-test to compare the means of two groups).

### Statistical Applications
* **Quality Control:** Uses statistical methods like control charts to monitor a process and ensure it is operating within its expected limits of variation.
* **Acceptance Sampling:** A quality control technique where a random sample is taken from a production lot to determine whether to accept or reject the entire lot based on the sample's quality.
* **Goodness of Fit and the $\chi^2$-Test:**
    A **goodness-of-fit** test determines if a sample dataset is consistent with a hypothesized distribution. The **Chi-Squared ($\chi^2$) Test** is a common method for this.
    * **Formula:** It compares observed frequencies ($O_i$) with expected frequencies ($E_i$) from the hypothesized distribution:
    $$ \chi^2 = \sum \frac{(O_i - E_i)^2}{E_i} $$
    A large $\chi^2$ value suggests the data does not fit the distribution well.
    * **Nonparametric Tests:** Statistical tests that do not rely on assumptions about the underlying distribution of the data (e.g., Wilcoxon rank-sum test).
* **Regression and Correlation:**
    * **Correlation:** A measure of the strength and direction of the linear relationship between two variables.
    * **Regression:** A method for modeling the relationship between a dependent variable and one or more independent variables. **Simple linear regression** aims to find the equation of a straight line ($y = a + bx$) that best fits the data, typically by using the **Method of Least Squares** to minimize the sum of the squared vertical errors between the data points and the line.

---
## Key Takeaways: The Power of Probability and Statistics

Probability and statistics are indispensable tools for navigating a world filled with uncertainty and data. They provide the methods to quantify chance, analyze trends, and make informed decisions.

* **Foundations of Probability:** The language of experiments, sample spaces, and events, governed by axioms, allows us to calculate the likelihood of occurrences using principles like the addition and multiplication rules, conditional probability, and Baye's theorem for updating beliefs.
* **Modeling Randomness:** Random variables and their probability distributions (such as Binomial, Poisson, Hypergeometric, and Normal) provide powerful models for understanding and predicting the behavior of random phenomena.
* **Descriptive Statistics:** Measures of central tendency (mean, median, mode) and dispersion (variance, standard deviation) provide a concise summary of a dataset's key characteristics.
* **Inferential Statistics:** This branch allows us to make inferences about a whole population based on a sample. Key tools include point estimation, confidence intervals for quantifying uncertainty, and hypothesis testing for making data-driven decisions.
* **Broad Applications:** From quality control and regression analysis to testing the fit of theoretical models, the applications of statistics are vast and crucial for scientific and economic progress.

The ability to think probabilistically and statistically is a critical skill in the modern world, empowering us to understand complexity and extract meaningful insights from the vast amounts of information that surround us.