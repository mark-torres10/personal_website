---
layout: single
title:  "What I'm learning (2024-05-20)"
date:   2024-05-20 08:00:00 +0800
classes: wide
toc: true
categories:
- personal
permalink: /personal/learnings-2024-05-20
---
# What I'm Learning (2024-05-20 edition)

## Machine learning

## Software engineering

## Research
- ["Conversations Gone Alright: Quantifying and Predicting Prosocial Outcomes in Online Conversations"](https://dl.acm.org/doi/abs/10.1145/3442381.3450122)
    - I think this paper does a different spin on the question of online content moderation in which they make a top-down claim of (1) what constitutes "prosocial online behavior" and (2) why this is an important part of content moderation in addition to just removing bad content.
    - They consistently raise a fair point when they say "is a prosocial outcome simply the lack of antisocial behavior or something more?". Most content moderation takes the perspective of equating "good" content moderation with merely removing bad language.
    - I like the list of traits that they mention. These make sense (e.g., social cohesion, subsequent comments, conversation depth), but I think they could have (1) complemented the approach with more traditional NLP (though they did say they wanted to introduce novel traits that were different from existing sentiment analysis approaches), and (2) used other trait classifiers instead of hand-crafting their own (this may be a constraint that they faced; for example, the Perspective API, which they used to detect toxicity, also detects prosocial attributes in conversations). I also disagreed with how some of the attributes were operationalized (e.g., measuring laughter or personal disclosure through more hard-coded keyword approaches). But, I did like that they then collapsed these traits using PCA to surface the combination that explains the highest degree of variance.
    - I'm not completely convinced by the results of their model that predicts which two conversations will have a more prosocial outcome. Their model barely edges human predictions in accuracy (0.578 to 0.563), but the annotators for the training set had only a moderate Krippendorff's $\alpha=0.59$ (which the authors themselves point out, and is to be expected given how subjective this task is). At best, the model does somewhere around the same level as humans, which is a contribution in and of itself because it does verify the explainable power of the factors chosen. I think this is directionally correct though and merits some follow-up, especially since, as the authors note, "[results] suggest that annotators were able to pick up on a complementary set of linguistic signals not used by the model, which future models might identify to improve performance.

## Personal
