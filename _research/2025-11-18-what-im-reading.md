---
layout: single
title:  "What I'm reading today (2025-11-18)"
date:   2025-11-18 15:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/2025-11-18-what-im-reading
---

# What I'm reading today (2025-11-18)

A brief compilation of things I'm reading and looking at today (and across the next couple of days). Recording my readings like this helps keep me accountable to understanding what I'm reading, while also keeping it light and sustainable.

## 1. CAMEL: Communicative Agents for "Mind" Exploration of Large Language Model Society

**[Paper link](https://www.alphaxiv.org/overview/2303.17760v2)**

A really great framework from the folks at Camel AI. Looks like their introductory paper to their multi-agent system framework, and they've got plenty more as well!

Their setup works like this:

- **Role-Playing Framework**: The system begins with a human providing a general idea (e.g., "develop a trading bot") and assigning broad roles (e.g., Python Programmer, Stock Trader). A task specifier agent then refines this initial concept into a specific, actionable task description, reducing the burden on human users to formulate precise prompts.
- **Inception Prompting**: This sophisticated prompt engineering strategy guides agent behavior through three main components. The prompts also include specific formatting requirements and behavioral constraints to prevent common issues in autonomous agent interactions.
  - Task Specifier Prompt: Guides the elaboration of the initial human idea
  - Assistant System Prompt: Defines the assistant's role, communication format, and constraints
  - User System Prompt: Establishes the user's role, interaction patterns, and termination conditions
- **Conversational Loop**: The AI user initiates dialogue by providing instructions to the AI assistant, who responds with solutions. This creates a multi-turn, instruction-following conversation that continues until the AI user determines the task is complete using a special termination token (<CAMEL_TASK_DONE>).

It seems like they carefully curated both the prompts and the orchestration layers to get these to work well, and the results are compelling! They show a lot of examples of large-scale conversational datasets across multiple domains that were generated with this methodology.

## 2. Exploring Prosocial Irrationality for LLM Agents: A Social Cognition View

**[Paper link](https://www.alphaxiv.org/overview/2405.14744v5)**

I find myself pretty unconvinced by this paper, primarily because I think it tries to attribute patterns and deeper cognitive underpinnings to LLM hallucinations when other work seems to point to LLM hallucinations being, among other things, [an artifact of how LLMs are evaluated](https://arxiv.org/abs/2509.04664) (e.g., LLMs don't want to say that they don't know something) as well as LLM tendencies for people-pleasing behavior. I don't think that in this case, LLMs can be (nor should they be) used to study human cognition principles, and I think LLM tendencies to reinforce things like sycophancy, herd effect, and confirmation bias are artifacts of the "next word prediction" pretraining objective and then human evaluator-based posttraining. But LLMs are getting so good these days that I think sometimes people forget that LLMs aren't human at all...

## 3. S3: Social-network Simulation System with Large Language  Model-Empowered Agents

**[Paper link](https://www.alphaxiv.org/overview/2307.14984v3)**

Like many other papers, this is another LLM simulation engine paper, and I think their key unique insight is grounding the networks on real social media data. The real social media data, like posts, likes, etc., is used to make a composite profile of each person. Then these grounded agents are set loose in the network and allowed to engage or interact.

Each agent in the simulation carries a profile built from real social media data: their inferred gender, age, occupation, past posts, and even the types of accounts they follow. When the agent receives a new message—say, a post about nuclear wastewater being released—they don’t react in the abstract. Instead, the system feeds the LLM a rich prompt that includes not just the content of the message, but also the agent’s internal state: “You are a 28-year-old marine biologist from Okinawa who has previously posted about ocean conservation and expressed concern about pollution. You’re currently in a state of high emotional intensity and hold a strongly negative attitude toward nuclear energy. You see this post: ‘The government says this water is safe. Here’s the official report.’” The LLM then generates a response, not by defaulting to a neutral summary or an obvious opinion, but by simulating how this particular person, with this history, would likely reply.

Overall, really interesting paper that I'll be doing a deeper dive on as this "grounding the AI agents on social media data" is something that I'm currently interested in as well.

## 4. MOSAIC: Modeling Social AI for Content Dissemination and Regulation in Multi-Agent Simulations

**[Paper link](https://www.alphaxiv.org/overview/2504.07830v3)**

This paper fits into the AI agent simulation networks architecture, but it looks like they're specifically interested in studying misinformation.

The setup that they have is pretty similar to other AI agent simulation frameworks:

- **Agent Creation and Personas**: they chose to ground their personas using anonymized survey data from 204 human participants collected through Prolific, covering demographics, political views, social behaviors, and personal values. This data is transformed into natural language persona descriptions that capture individual characteristics. Second, synthetic personas are generated using a structured "Agent Bank" with 23 multiple-choice questions covering attributes like age, gender, location, education, and political stance, with distributions mirroring real-world demographics.
  - So far, what I've seen are common grounding datasets to use for AI agent creation and personas are (1) real human survey data that they've collected, (2) aggregated survey data collected by others (e.g., ANES), or (3) social media data (like in S3 above).
- **AI Agents Interacting**: AI agents drive the interaction components. Each agent can perform various actions including liking posts, sharing content, commenting, flagging misinformation, following or unfollowing other users, and ignoring content. One key design component for these systems is the sort of actions that are available to the agents, and for this use case they gave AI agents the ability to flag misinformation.
- **Memory and Reflection Systems**: Here, they add an `AgentMemory` module to store past interactions and experiences. The devil is in the details with agentic memory and context engineering, and for here they try to do things like:
  - Update importance scores based on semantic categories
  - implement temporal decay to simulate how memories fade over time.
  - generate reflections about their experiences, allowing them to adapt their future behavior based on past interactions.

Misinformation content comes from 1,353 false news articles sourced from NewsGuard's proprietary database, while factual news consists of 2,470 articles scraped from major media outlets via NewsAPI. This dual content stream allows researchers to study how true and false information compete for attention and engagement within the same network.

For their feed algorithm, they use a simple recency-based feed algorithm combined with social network relationships, creating a follower-based feedback loop that can amplify certain types of content. This is a knob that could be turned when creating the simulation.

They found in their work that, contrary to extensive empirical research showing that false news spreads faster and more extensively than true news in human networks, MOSAIC simulations revealed that LLM-driven agents do not preferentially spread misinformation. The researchers think that this behavior stems from LLMs' inherent safety alignment and conservative sharing policies, which make agents more hesitant to propagate uncertain or potentially harmful information and which I think is spot on.

It's not clear to me what about their approach is unique to misinformation, as any of the many other AI agent simulation network proposal architectures could've just as easily worked here (I mention S3 and CAMEL above). Still, interesting paper! I think it can be done with any of the other AI agent simulation architectures and they could've just added the misinformation and fact-checking tools on top, so it's good to see that after reading a few of these papers, I'm seeing commonalities in the design.

## 5. Agent-Based Simulations of Online Political Discussions: A Case Study on Elections in Germany

**[Paper link](https://www.alphaxiv.org/overview/2503.24199v1)**

Good end-to-end AI agent-based simulation project. They fine-tuned a Llama model to generate realistic tweets, based on the dataset that they manually curated. Then they found profiles of real Twitter users who had posted at least 15 times and replied to at least 25 others, and used these as the "historical context" that they initialized each agent with.

During each simulation iteration, the system cycled through four phases: post generation, comment generation, liking, and disliking.

- **Post Generation**: In each round, a subset of agents (20%) generated new posts using the post-generation model, drawing from their own history and recent platform activity.
- **Comment Generation**: A larger subset (80%) then responded to these posts using the reply-generation model, with selection for reply targets influenced by a classifier that estimated which user-post pairs were most likely to interact, based on historical alignment and engagement probability.
- **Liking/disliking**: Additionally, a smaller random subset of agents liked or disliked existing posts, mimicking passive engagement behavior.
  
The simulation ran for 30 iterations under six distinct experimental conditions, varying whether historical context was used, whether resource and motivation constraints were active, and how replies were ranked (by relevance, chronology, or randomness). Of course all of these choices are hyperparameters that can be set, and it would be interesting to vary them each one by one.

They also introduced reward and time constraint functions, to skew the AI agents to consider the cost and reward of doing certain actions. The simulation had a general resource budget that replenishes over time (representing a user’s available time and energy), a per-session budget (limiting how much one can do in a single sitting), and a reward function that combines personal value (time spent, content enjoyed) and social feedback (likes, retweets, FOMO). Agents had a probabilistic login mechanism modeled as a Poisson process, determining when they joined the platform based on expected rewards.

Some thoughts on their findings:

- I think the addition of a cost and reward mechanisms and a budget is a really interesting wrinkle. It's the most significant contribution that this paper has, in my opinion, in the broader AI agent simulation literature. By adding a cost penalty to doing certain actions, you constrain the action space, and by adding a reward function, you nudge the model to favor certain actions over others. Of course this is finicky and needs to be tuned and maybe you run into the critique that ABMs have where their findings are dependent on hyperparameters. But I think this actually works well in the AI agent use case because the AI agents are intelligent enough to plan out these tradeoffs themselves and to introduce noise into the results (hopefully based on their different personas), meaning that the range of actions per agent is really personalized.
- It's great that they were able to resurface commonly found results in the social science literature, such as when given constraints, agents defaulted more frequently to passive actions like liking or disliking, suggesting resource scarcity nudges users toward low-effort responses. Though this is great to see, like I mentioned before I wonder to what degree this is "overfitting to the test set", where since you know what humans are supposed to act like, you can tune your parameters to overfit that. But it's also unclear to me that, even if that were the case, that it's necessarily a bad thing, since don't you want the simulators to be as faithful to humans as possible (assuming that your sample of humans is a good sample of humans as a whole, then overfitting on your sample seems OK)?
