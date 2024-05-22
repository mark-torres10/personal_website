---
layout: single
title:  "Experiments with LLM classification for political content (Part IX)"
date:   2024-05-22 04:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/llm-experiments-pt-ix
---
# Using LLMs to classify if social media posts are political or not (Part IX).
I'm working on a project that involves gathering social media posts from [Bluesky](https://bsky.app/) and analyzing them. Part of that project requires knowing which posts are about political or social topics, and if so, what political side they support.

[Previously](https://markptorres.com/research/llm-experiments-pt-viii), I worked on migrating the data pipeline to KLC. Now I'm working on more work with using KLC for the project. Specifically, I want to move the data preprocessing and filtering steps to KLC. 

Specifically, I'll do the following:
- Consolidate the data formats across the different sync types.
- Update the data preprocessing and filtering steps.
- Create a pipeline to run the data preprocessing and filtering steps.
- Run the pipeline on a cron job on the compute cluster.

## Consolidate the data formats across the different sync types

