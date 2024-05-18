---
layout: single
title:  "Experiments with LLM classification for political content (Part VII)"
date:   2024-05-08 20:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/llm-experiments-pt-vii
---
# Using LLMs to classify if social media posts are political or not (Part VII).
I'm working on a project that involves gathering social media posts from [Bluesky](https://bsky.app/) and analyzing them. Part of that project requires knowing which posts are about political or social topics, and if so, what political side they support. Current ML classifiers don't work that well out of the box, so I'm trying to create our own classification scheme using LLMs. I'm trying to use LLMs in order to classify [Bluesky](https://bsky.app/) posts as either having political content or not, and if so, the political ideology, and I've found that LLMs work quite well for this task. I've used Llama3-8b and Llama3-70b via [Groq](https://groq.com/) so far, but are also open to experimenting with other open-source models as well (I have the on-prem infrastructure to host our own models, which is much cheaper at scale).

[Previously](https://markptorres.com/research/llm-experiments-pt-i), I confirmed that LLMs are promising for our classification task. We now want to replicate this. We [previously](https://markptorres.com/research/llm-experiments-pt-iv) synced the data for annotation. We then [cleaned up](https://markptorres.com/research/llm-experiments-pt-v) the code to have a more robust ETL pipeline.

Now I want to optimize the LLM pipeline even more. We'll want to be able to support running this on the scale of 20,000-50,000 posts per day. We want to do this so that we can run our classifier on firehose data.

## Experiment with converting JSON to YAML

### Comparison of JSON format to YAML

### Adding a YAML format to the context generation

### Comparing token counts and LLM performance under both formats

## Caching and pre-computing context

## Reducing tokens in the request

## Minimize requests to Bluesky for context generation as much as possible