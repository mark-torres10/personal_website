---
layout: single
title:  "Do social media recommendation algorithms make people polarized?"
date:   2024-01-06 15:00:00 +0800
classes: wide
toc: true
categories:
- research
permalink: /research/recommender-systems-and-polarization
---

## TL;DR
It depends, maybe? Research is still inconclusive. At the very least, it's not as obvious as "recommendation algorithms push people to echo chambers which makes them more polarized".

## Definitions

### What is a recommendation algorithm?

#### Why do we need a recommendation algorithm?

#### How do recommendation algorithms work?

#### How do recommendation algorithms curate what you see on social media?

### What is polarization?
At a high level, polarization happens when the beliefs of a society are concentrated on opposing extremes. For example, in America, you're either Democrat or Republican, and in a polarized society, a Democrat must have only the beliefs that Democrats have and Republicans must have only the beliefs that Republicans have. In a polarized society, there's diminished understanding between both sides, because both sides care about fundamentally different problems.

#### Definitions

#### What polarization is

#### What polarization is not

## How do recommendation algorithms affect polarization?

### What do recommendation algorithms optimize for?

#### What is engagement?

#### How do we measure engagement?

#### Why is engagement something to optimize for?

### Do recommendation algorithms cause more polarization?

#### Yes, they do

Most research on reducing polarization operates under this "filter bubble" paradigm.

#### Actually, maybe not?


#### What else could be causing more polarization on social media? Are recommendation algorithms to blame?
[Alcott et al. (2020)](https://www.aeaweb.org/articles?id=10.1257%2Faer.20190658&utm_campaign=Johannes) found that paying people to stay off Facebook for 4 weeks caused a reduction in feelings of political polarization. However, they also reported lower news consumption overall. Is the reduction in polarization due to not seeing the Facebook algorithm's recommendations or is it due to not being around the news,

##### Political parties
Could political party be a confounding factor? Research (e.g., [Huszar et al. (2021)](https://www.pnas.org/doi/full/10.1073/pnas.2025334119) and [Flamino et al. (2023)](https://www.nature.com/articles/s41562-023-01550-8)) has found that right-leaning users experience more algorithmic amplification of their feeds than others.

Right-leaning users are also more likely to report experiencing polarization than other users.

However, does the fact that right-leaning users experience more algorithmic amplification than others necessarily mean that the algorithms cause more polarization in them?

##### Political elites
(something about political elites?)


## Looking ahead: what are some ways to improve recommendation algorithms?

### What do we mean by "improve" recommendation algorithms?

#### Changing the design of the recommendation algorithms
Some interventions change the recommendation algorithm design itself. For example, [Li et al. (2021)](https://dl.acm.org/doi/10.1145/3437963.3441769) note that traditional recommendation algorithms, which provide better recommendations to users whose tastes are more mainstream, can be adapted to reflect more personalized preferences. [Gao et al. (2022)](https://dl.acm.org/doi/10.1145/3477495.3531890) propose a way for algorithms to be created that can optimize for a given metric (e.g., engagement) while giving more diverse results along another metric (e.g., polarization). [Celis et al. (2019)](https://www.cs.yale.edu/homes/vishnoi/Polarization.pdf) provide a slightly different approach where they instead constrain the sampling distribution so that the recommendation algorithm is less likely to pick undesirable outcomes. 

A common thread behind these approaches is a desire to "upsample" desirable outcomes (e.g., posts that we want users to see) while "downsampling" undesirable outcomes (e.g., posts that promote polarization). This is similar to other work in the recommender algorithms literature (e.g., [Do et al. (2023)](https://arxiv.org/pdf/2210.09957.pdf) and [Zehlike and Castillo (2018)](https://arxiv.org/abs/1805.08716)) that look at how we can make these algorithms more fair (e.g., debiasing on race, gender, etc., see [Zehlike et al., (2020)](https://arxiv.org/pdf/2103.14000.pdf) for a survey of current research). 

#### Changing the objective function of recommendation algorithms
Other work focuses on the objective functions of these recommendation algorithms. Normally, these prioritize "engagement", however that may be defined.
(Stray - optimize human values)

#### Explaining why the algorithm made its recommendations
Much of machine learning is traditionally a "black box", where an algorithm can make a recommendation but we as users are unable to explain why it came to that recommendation. As black-box algorithms become more prevalent (e.g., [large language models](https://arxiv.org/abs/2310.00603)), [more work](https://orca.cardiff.ac.uk/id/eprint/152361/1/3561048.pdf) is being done to improve the explainability of these models.

While we want to improve transparency of algorithms in general, is increased transparency helpful for reducing polarization? 

### Recommendation algorithms and the alignment problem

### Conclusion
