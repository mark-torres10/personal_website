---
layout: single
title: "Probability theory, MGFs, Taylor Series, and Chi-Squared"
date: 2026-01-27 15:00:00 +0800
classes: wide
toc: true
categories:
- self_education
permalink: /self_education/2026-01-27-probability-theory-notes
---

# Probability theory, MGFs, Taylor Series, and Chi-Squared: Personal notes for math foundations

I'm brushing up foundational math principles and concepts related to the mathematics of machine learning and AI. Here are some notes that I'm taking along the way.

## Probability theory basics

Let $X = X_1 + X_2$, where $X_1 \sim N(\mu_1, \sigma_1^2)$ and $X_2 \sim N(\mu_2, \sigma_2^2)$ are independent.

### Basic linearity of expectations

We have $X = X_1 + X_2$. Then, $$E[X] = E[X_1 + X_2] = E[X_1] + E[X_2]$$

Given $X_1 \sim N(\mu_1, \sigma_1^2)$ and $X_2 \sim N(\mu_2, \sigma_2^2)$, then $E[X_1] = \mu_1$ and $E[X_2] = \mu_2$.

Therefore, we know that the resulting distribution is centered around $\mu_1 + \mu_2$, using **linearity of expectations**.

### Calculating variance

*The variance of independent variables is additive.*

We can show this in the following way:

We know that

$$Var(X) = E[(X - E[X])^2]$$

Since $X = X_1 + X_2$ and $E[X] = E[X_1 + X_2] = \mu_1 + \mu_2$ (as shown previously), we can replace these terms in the equality.

$$Var(X) = E[(X - E[X])^2] \Rightarrow E[((X_1 + X_2) - (\mu_1 + \mu_2))^2]$$

Grouping terms:

$$Var(X) = E[(X - E[X])^2] \Rightarrow E[((X_1 - \mu_1) + (X_2 - \mu_2))^2]$$

We distribute:

$$Var(X) = E[(X - E[X])^2] \Rightarrow E[(X_1 - \mu_1)^2 + 2(X_1 - \mu_1)(X_2-\mu_2) + (X_2 - \mu_2)^2] $$

We use linearity of expectations to distribute:

$$E[(X_1 - \mu_1)^2 + 2(X_1 - \mu_1)(X_2-\mu_2) + (X_2 - \mu_2)^2]$$ 
$$\Rightarrow E[(X_1 - \mu_1)^2] + 2E[(X_1-\mu_1)(X_2-\mu_2)] + E[(X_2 - \mu_2)^2]$$

We take the definitions of variance and covariance:

$$Cov(X, Y) = E[(X-E[X])(Y-E[Y])]$$
$$Var(X) = E[(X-E[X])^2] = E[(X-E[X])(X-E[X])] = Cov(X, X)$$

We also remember that $E[X_1] = \mu_1$ and $E[X_2] = \mu_2$.

Given all this, we can plug in terms.

$$E[(X_1 - \mu_1)^2] + 2E[(X_1-\mu_1)(X_2-\mu_2)] + E[(X_2 - \mu_2)^2]$$

$$\Rightarrow Var(X_1) + Cov(X_1, X_2) + Var(X_2)$$

Given that $X_1$ and $X_2$ are independent, we can show that this makes $Cov(X_1, X_2) = 0$.

- If $X_1$ and $X_2$ are independent, we can factor out the expectations: $E[X_1X_2] = E[X_1]E[X_2]$.
- Looking at our $Cov(X_1, X_2)$ term: $$Cov(X_1, X_2) = E[(X_1-\mu_1)(X_2-\mu_2)]$$ $$\Rightarrow E[(X_1-E[X_1])(X_2-E[X_2])]$$
- We distribute within the expectation: $$E[(X_1-E[X_1])(X_2-E[X_2])]$$ $$\Rightarrow E[X_1X_2 - X_1E[X_2] - X_2E[X_1] + E[X_1]E[X_2]]$$
- We know that for taking expectations, $E[cX] = cE[X]$ and that $E[X]$ and $E[Y]$ are treated as constants within the expectation expression.
- Therefore, we get the following: $$E[X_1X_2 - X_1E[X_2] - X_2E[X_1] + E[X_1]E[X_2]]$$ $$\Rightarrow E[X_1X_2] - E[X_1]E[X_2] - E[X_2]E[X_1] + E[X_1]E[X_2]$$ $$\Rightarrow E[X_1X_2] - E[X_1]E[X_2]$$
- Given our definition of independence, $E[X_1X_2] = E[X_1]E[X_2]$. Therefore, $E[X_1X_2] - E[X_1]E[X_2] = 0$.

As a result, we rigorously demonstrate that the **variance of our new distribution is defined as $\sigma_1^2 + \sigma_2^2$**.

## What is a moment-generating function?

We want to prove that the distribution for $X = X_1 + X_2$, where $X_1 \sim N(\mu_1, \sigma_1^2)$ and $X_2 \sim N(\mu_2, \sigma_2^2)$, is also a normal distribution.

### What's the problem that we're solving?

### What is a Taylor Series?

##