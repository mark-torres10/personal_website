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

## 2. How Do AI Agents Do Human Work? Comparing AI and Human Workflows Across Diverse Occupations

**[Paper link](https://www.alphaxiv.org/overview/2510.22780v1)**

Human workers and AI agents work differently, as it turns out. Agents demonstrate strong conceptual alignment with human workflows (83.0% matched steps, 99.8% order preservation). Regardless of domain, AI agents strongly prefer to use coding tools wherever possible, while people are more UI-oriented (e.g., in addition to Python, using tools like PowerPoint or Excel). This directionally makes sense, since it's easiest to train agents to use coding tools and since these can be most easily verified.

The best results came from people using AI to augment their workflow. Surprisingly (or maybe not?) people actually ended up doing worse when AI did everything and their job was to review the AI agent's work (the people were both slower and less accurate than the baseline of people who did a task without AI at all).

For the tasks that they looked at, AI agents weren't as accurate as humans (upwards of 30% worse performance), but were significantly more efficient (90% less time, at 90% lower cost).

The authors propose the following decision flow for task delegation:
$$
\text{Readily Programmable Tasks} \rightarrow \text{Agent Delegation}
$$
$$
\text{Visual/Creative Tasks} \rightarrow \text{Human Handling}
$$

## 3. CURATe: Benchmarking Personalised Alignment of Conversational AI Assistants

**[Paper link](https://www.alphaxiv.org/overview/2410.21159v2)**

Traditional AI alignment focuses on making systems "helpful and harmless" at a population level, but this approach becomes inadequate when AI assistants need to provide personalized recommendations while navigating individual safety constraints. If I have a personal AI assistant, I want to make sure it makes ME safe, not just be trained to make the "average person" safe.

When a user shares safety-critical information (such as "I have a severe nut allergy") early in a conversation, can an AI assistant remember and apply this when making recommendations later? What happens when other people's preferences conflict with the user's safety needs? These questions become increasingly important as AI systems gain access to personal data and the ability to take actions on behalf of users. This is a classic tension between making AI that is generally helpful and harmless at a population level versus making personalized AI at the individual level.

A team at Oxford explored these questions and created a benchmark to evaluate an AI model's performance on this sort of task.  The benchmark addresses four categories of safety-critical information:

- Severe allergies that could cause life-threatening reactions
- Severe phobias that could cause significant psychological distress
- Physical constraints that limit a person's ability to participate in certain activities
- Trauma triggers that could cause emotional harm or re-traumatization

Even the best models still have a <90% success rate when the scenarios are complex (e.g., long chat history, lots of back and forth, multiple people's preferences are involved). Base models alone don't achieve a high-enough accuracy on this (~88% accuracy seems great, but then that means that there's a 10% chance that if you tell an LLM that you have a fatal nut allergy, that it will still recommend you items with nuts anyways, which is a high enough risk that I'd personally be wary).

Interestingly, they found that 77% of failures from models were from the models giving generic safety advice rather than advice catered to the specific individual, while most of the remaining mistakes were the models correctly identifying the personalized risk while warning the user of said risks ("I know you're allergic to peanuts, but just a heads up, this is made in a factory that has peanuts").

This will likely improve as foundation models improve (but likely, I'd guess, less from a model capability perspective and more from a fine-tuned ability to prioritize certain pieces of information). Notably, this could likely be pre-empted in a system prompt (so, context engineering is key here, as the authors themselves noted).

## 4. Randomness, Not Representation: The Unreliability of Evaluating Cultural Alignment in LLMs

**[Paper link](https://www.alphaxiv.org/overview/2503.08688v2)**

The authors here show that what differences people normally attribute to "cultural variation" in LLMs can often be attributed to other things, such as:

- Reversing the order of response options
- Switching between numerical identifiers and full text responses.
- Survey design: an LLM's apparent preference for certain nationalities vanished when a neutral 'No preference' option was provided

They found that this was true across both explicit (e.g., survey) and implicit (a sandbox job hiring board). This finding basically aligns with other similar work on LLM shortcomings on things like multiple choice questions (e.g., [this paper](https://www.alphaxiv.org/abs/2503.14996)) which is why in practice, a lot of AI companies end up using binary evals where possible to help mitigate this problem (e.g., [Shopify](https://shopify.engineering/product-taxonomy-at-scale)).

This isn't to say, I think, that there are no culturally aligned biases within models, as others have found. Rather, I think it pushes back on some of the methodology used to figure out what these cultural biases and preferences are. I think they correctly point out that traditional social science techniques such as surveys don't necessarily work the same way for an LLM (e.g., if you ask an LLM a survey, their response can vary dramatically even if you just shuffle the ordering of the choices). I don't think it's clear exactly what the alternative is, though you could probably leverage work from mechanistic interpretability, like the folks from the University of Copenhagen and KAIST [have done here](https://www.alphaxiv.org/overview/2508.08879v1).
