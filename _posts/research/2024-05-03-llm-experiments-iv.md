---
layout: single
title:  "Experiments with LLM classification for political content (Part IV)"
date:   2024-05-03 15:00:00 +0800
classes: wide
toc: true
categories:
- research
permalink: /research/llm-experiments-pt-iv
---
# Using LLMs to classify if social media posts are political or not (Part IV).

I'm working on a project that involves gathering social media posts from [Bluesky](https://bsky.app/) and analyzing them. Part of that project requires knowing which posts are about political or social topics, and if so, what political side they support. Current ML classifiers don't work that well out of the box, so I'm trying to create our own classification scheme using LLMs. I'm trying to use LLMs in order to classify [Bluesky](https://bsky.app/) posts as either having political content or not, and if so, the political ideology, and I've found that LLMs work quite well for this task. I've used Llama3-8b and Llama3-70b via [Groq](https://groq.com/) so far, but are also open to experimenting with other open-source models as well (I have the on-prem infrastructure to host our own models, which is much cheaper at scale).

Previously, I've tried using just naive text classification and then afterwards adding context to the classifications. Now that I've shown that individual prompts work, I've also worked on how to include batching and adding some (simple) scaling for the prompts. I then added a service for getting current news coverage from different news orgs across the political spectrum and created an interface for surfacing relevant articles given a query.

Currently, I'm working on (1) updating and formalizing an LLM inference pipeline and (2) generating data for annotations.

### Updating the LLM inference pipeline

### Getting data for annotations

### Putting it all together

### Next steps
Next, I'd like to go back to other improvements for the process, such as:
- How does our model perform with other LLMs (e.g., Mixtral)?
- Can we experiment with optimizing the prompt (e.g, with [dspy](https://github.com/stanfordnlp/dspy))?

I'd also like to revisit some of the points related to improving how to add context about current events:
- For determining when to get context for a post, investigate various strategies such as:
    - Keyword matching: see if a keyword (e.g., a name of an event) comes up. Need to figure out keywords that describe topics that are in the news (this is easiest if it is the name of a notable event, place, person, piece of legislature, etc.) and then we can easily pattern match that against posts that have that keyword.
    - Posts that the LLM knows is political but isn't sure what the political ideology is.
- Determine how to format the context that's given to the LLM prompt.
    - An interesting frame could be first asking the LLM to distill the sentiments and thoughts of each political party about a certain topic, based on the articles that we have for each topic, and then passing this distilled summary to the LLM itself.
- Only insert into the vector store if it doesn’t already exist there.
- At some point, add a maximum distance measure so we get only relevant articles (will take some experimentation in order to see what a good distance is).
