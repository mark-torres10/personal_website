---
layout: single
title:  "Experiments with LLM classification for political content (Part VI)"
date:   2024-05-08 20:00:00 +0800
classes: wide
toc: true
categories:
- research
permalink: /research/llm-experiments-pt-vi
---
# Using LLMs to classify if social media posts are political or not (Part VI).

I'm working on a project that involves gathering social media posts from [Bluesky](https://bsky.app/) and analyzing them. Part of that project requires knowing which posts are about political or social topics, and if so, what political side they support. Current ML classifiers don't work that well out of the box, so I'm trying to create our own classification scheme using LLMs. I'm trying to use LLMs in order to classify [Bluesky](https://bsky.app/) posts as either having political content or not, and if so, the political ideology, and I've found that LLMs work quite well for this task. I've used Llama3-8b and Llama3-70b via [Groq](https://groq.com/) so far, but are also open to experimenting with other open-source models as well (I have the on-prem infrastructure to host our own models, which is much cheaper at scale).

[Previously](https://markptorres.com/research/llm-experiments-pt-i), I confirmed that LLMs are promising for our classification task. We now want to replicate this. We [previously](https://markptorres.com/research/llm-experiments-pt-iv) synced the data for annotation. We then [cleaned up](https://markptorres.com/research/llm-experiments-pt-v) the code to have a more robust ETL pipeline.

Now we want to get some baselines with the Perspective API in order to learn more about the conversation properties of our posts.

## Primer on the Perspective API

The [Perspective API](https://perspectiveapi.com/) is an API from Google Jigsaw that detects (and reduces) toxicity online (and, as of late, also promotes healthy dialogue).

## Where the Perspective API fits into our project

## Classifiying our posts using the Perspective API

We have a thesis that by explicitly downranking toxic posts and upranking constructive posts, we'll surface more of the opinions that are traditionally silenced by algorithmic amplification and give a more nuanced perspective on what people from a particular political party believe (as opposed to just the most extreme, rage-filled, clickbait-filled posts that are normally pushed).

### Loading the posts from MongoDB

### Classifying with the Perspective API

### Storing the results of the classifications into MongoDB

## Analyzing our results

## Creating a demo app with Streamlit for future analysis

## Next steps
Now that we have some benchmarks for ...

Some of the things that I want to work on next are:
- Updating and refactoring the LLM pipeline to label the posts efficiently at scale.
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
