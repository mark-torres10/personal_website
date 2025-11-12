---
layout: single
title:  "What I'm reading today (2025-11-12)"
date:   2025-11-12 05:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/2025-11-12-what-im-reading
---

# What I'm reading today (2025-11-12)

A brief compilation of things I'm reading and looking at today (and across the next couple of days). Recording my readings like this helps keep me accountable to understanding what I'm reading, while also keeping it light and sustainable.

## 1. LLM Can be a Dangerous Persuader: Empirical Study of Persuasion Safety in Large Language Models

**[Paper link](https://www.alphaxiv.org/overview/2504.10430v1)**

There's been plenty of work done on trying to persuade LLMs to do malicious or harmful things. But this paper interestingly looks at the reverse, evaluating how well LLMs can be themselves dangerous persuaders.

What I thought was an interesting nugget was the finding that some models that appeared "safe" by refusing harmful tasks still exhibited unethical behavior when they did engage in persuasion. A lot of the safety work that I've read discusses the propensity for models to reject harmful queries, but what happens in an agentic future where it'll be mostly LLMs talking to each other? In that case, it sounds like if you could jailbreak an LLM to be a malicious actor, it would have both the ability to and the incentive for persuading other models.

In addition, external contextual factors, such as the persuader's benefit from achieving the goal or pressure to accomplish the persuasion, led to higher usage of unethical strategies. LLMs will increasingly do things autonomously on behalf of users, and as they do, they'll increasingly learn about user motives and objectives and will possibly be more apt to engage in deception if it furthers their mission to be a "helpful assistant".
