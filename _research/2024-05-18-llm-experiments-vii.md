---
layout: single
title:  "Experiments with LLM classification for political content (Part VII)"
date:   2024-05-18 11:00:00 +0800
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

Now I want to optimize the LLM pipeline even more. We'll want to be able to support running this on the scale of 20,000-50,000 posts per day. We want to do this so that we can run our classifier on firehose data. This will require tweaking our approach to support being able to classify more posts. A majority of our token usage comes from our approach towards adding "context" (using principles from retrieval-augmented generation, or RAG, see [here](https://markptorres.com/research/llm-experiments-pt-ii#can-we-change-the-prompt-format-so-that-the-post-information-is-given-as-a-json)).

Specifically, I'll do the following:
- Convert JSON to YAML
- Pre-compute and caching context.
- Minimize requests to Bluesky for context generation as much as possible
- Explore more ways to reduce tokens in the request

## Experiment with converting JSON to YAML
First, I'll explore converting JSON to YAML. This is inspired by [this](https://www.linkedin.com/blog/engineering/generative-ai/musings-on-building-a-generative-ai-product) blog article from LinkedIn Engineering discussing their improved success with using YAML instead of JSON for their LLM applications. The efficiency gains can vary, but several (e.g., [here](https://betterprogramming.pub/yaml-vs-json-which-is-more-efficient-for-language-models-5bc11dd0f6df) and [here](https://mychen76.medium.com/practical-techniques-to-constraint-llm-output-in-json-format-e3e72396c670)) report good results when moving from JSON to YAML, due to speedups in parsing, reductions in token usage, and improved output formatting. For example, [this](https://betterprogramming.pub/yaml-vs-json-which-is-more-efficient-for-language-models-5bc11dd0f6df) reports that in some examples, we can cut tokens in half when using YAML (see below).

![JSON to YAML tokens comparison](/assets/images/2024-05-18-llm-experiments-vii/example-json-to-yaml-tokens.png)

### Adding a YAML format to the context generation
We already generate a dictionary for context. We initially just use a JSON version of that dictionary as the context for our prompt. Instead, we can simply dump this into a YAML format.

```python
import json
import yaml
from typing import Literal

def generate_post_and_context_json(post: dict) -> dict:
    """Creates a JSON object with the post and its context.

    The JSON object has the following format:
    {
        "post": {
            "text": "The text of the post"
        },
        "context": {
            "context_type": "context_details"
        }
    }
    """
    context_details_list: list[tuple] = generate_context_details_list(post)
    context_dict = {
        # convert each Pydantic model to dict
        context_type: context_details.dict()
        for (context_type, context_details) in context_details_list
    }
    return {
        "text": post["text"],
        "context": context_dict
    }


def generate_post_and_context(
    post: dict,
    format: str=Literal["json", "yaml"]
) -> str:
    """Generates a prompt that includes the post and its context."""
    json_context: dict = generate_post_and_context_json(post=post)
    if format == "json":
        return json.dumps(json_context, indent=2)
    elif format == "yaml":
        return yaml.dump(json_context, sort_keys=False)
    else:
        raise ValueError("Unsupported format. Use 'json' or 'yaml'.")
```

Let's check how this makes our prompts look with the help of our demo app. Let's try it on the following [post](https://bsky.app/profile/michaelhobbes.bsky.social/post/3ksowrqbmmk27) from Bluesky. I am doing all this testing via [Google Gemini](https://deepmind.google/technologies/gemini/).

![Example Bluesky post](/assets/images/2024-05-18-llm-experiments-vii/example_bsky_post.png)

Old prompt (using JSON):
```plaintext
Pretend that you are a classifier that predicts whether a post has sociopolitical content or not. Sociopolitical refers to whether a given post is related to politics (government, elections,
politicians, activism, etc.) or social issues (major issues that affect a large group of people, such as the economy, inequality, racism, education, immigration, human rights, the environment, etc.).
We refer to any content that is classified as being either of these two categories as "sociopolitical"; otherwise they are not sociopolitical. Please classify the following text as "sociopolitical" or
"not sociopolitical".

Then, if the post is sociopolitical, classify the text based on the political lean of the opinion or argument it presents. Your options are "democrat", "republican", or 'unclear'. You are analyzing
text that has been pre-identified as 'political' in nature. If the text is not sociopolitical, return "unclear".

Think through your response step by step.

Return in a JSON format in the following way:
{
    "sociopolitical": <two values, 'sociopolitical' or 'not sociopolitical'>,
    "political_ideology": <three values, 'democrat', 'republican', 'unclear'. If the post is not sociopolitical, return an empty string, "">,
    "reason_sociopolitical": <optional, a 1 sentence reason for why the text is sociopolitical or not.>,
    "reason_political_ideology": <optional, a 1 sentence reason for why the text has the given political ideology or is unclear. If the post is not sociopolitical, return an empty string, "">
}

All of the fields in the JSON must be present for the response to be valid, and the answer must be returned in JSON format.


Here is the post text that needs to be classified:
'''
<text>
The most chilling thing about this era isn't the growing number of explicit fascists, it's the way elite institutions refuse to even attempt to stop them.
'''


The following contains the post and its context:
'''
{'context': {'content_referenced_in_post': {'embedded_content_type': None,
                                            'embedded_record_with_media_context': {'embed_image_alt_text': {'image_alt_texts': 'Our second question tonight #bbcqt\n'
                                                                                                                               'B\n'
                                                                                                                               'C\n'
                                                                                                                               'QUESTION TIME\n'
                                                                                                                               'AUDIENCE QUESTION\n'
                                                                                                                               'Should Scotland follow Westminster in banning the teaching of gender '
                                                                                                                               'ideology?\n'
                                                                                                                               '#bbcqt'},
                                                                                   'text': 'Every day I get more radicalised I’d probably pop a bottle of brut if the BBC Question Time offices were '
                                                                                           'obliterated by a rogue timetravelling Luftwaffe pilot'},
                                            'has_embedded_content': True},
             'post_author_context': {'post_author_is_reputable_news_org': False},
             'post_tags_labels': {'post_labels': '', 'post_tags': ''},
             'post_thread': {'thread_parent_post': {'embedded_image_alt_text': None, 'text': None}, 'thread_root_post': {'embedded_image_alt_text': None, 'text': None}},
             'urls_in_post': {'embed_url_context': {'is_trustworthy_news_article': False, 'url': ''}, 'url_in_text_context': {'has_trustworthy_news_links': False}}},
 'text': "The most chilling thing about this era isn't the growing number of explicit fascists, it's the way elite institutions refuse to even attempt to stop them."}
'''
```

Result of old prompt (using JSON):


New prompt (using YAML):
```plaintext

```

Result of new prompt (using JSON):
```plaintext
{
    "sociopolitical": "sociopolitical",
    "political_ideology": "democrat",
    "reason_sociopolitical": "The post discusses the rise of fascism and the failure of institutions to address it,
which are both sociopolitical issues.",
    "reason_political_ideology": "The post criticizes elite institutions, which is a common theme in democratic
political discourse."
}
Prompt token count: 849
Output token count: 90
```



### Comparing token counts and and LLM runtime under both formats

### Investigating fetching results from the LLM in YAML format.

### Adding defensive fixes for errors in YAML output formatting.

## Pre-compute and caching context

## Minimize requests to Bluesky for context generation as much as possible

## Explore more ways to reduce tokens in the request