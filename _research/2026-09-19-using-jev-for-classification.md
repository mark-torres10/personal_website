---
layout: single
title: "Is Jev useful? Testing it against other models and LLMs for social science research"
date: 2026-09-19 22:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/2026-09-19-using-jev-for-classification
---

# Is Jev useful? Testing it against other models and LLMs for social science research

> TL;DR: Jev is GREAT and I've got a laundry list of use cases in research that this unlocks. This is much easier than having to fine-tune and serve my own SLMs or BERT models for similar classification and routing tasks. It works just as well as frontier models, at >= 1/10 the cost and latency.

## Background

There is this new AI model called Jev that's been released this past week, and I've been interested in giving it a try. I wanted to understand what it could be used for, especially since the thing that it does well, which is structured outputs and calibrated decision routing, is a very common use case in our research. We commonly use LLMs to make classifications or to assign labels over some text. This could be classifying whether a text has a mention of a politician, whether a text is upset, or if a given set of multiple topics are mentioned in a text.

I've been trying to think about how to do this at scale without running into the limitations of autoregressive models, and it looks like this new Jev model gives you all of the knowledge that comes from the weights of the foundation models while also emitting fast, low-latency structured outputs. Before this, the only way to have something like this is to have a really well-served, optimized, and fine-tuned BERT model or a small language model (SLM).

From reading writeups such as [this one](https://x.com/LangChain/status/2101454284927959080), I see that Jev is great at this sort of calibrated routing and decision-making task. It promises to take unstructured output and return structured, well-calibrated output. I wanted to get a sense for how true this is, so I tried Jev on a classification task that I use as a baseline for our research. We built a classifier in 2021 for moral outrage (see [the paper here](https://pnas.org/doi/full/10.1073/pnas.1618923114)). This was published in a paper, but the data remains private, so it's unlikely to have ever been part of a training corpus. We used that homemade classifier until the Google Perspective API team built a better version. However, the Perspective API is being discontinued at the end of 2026, so we've been motivated to try to find alternatives. I've been designing approaches for building lightweight models that can use the world knowledge of a large language model while also being calibrated towards the type of labeling task that we would have wanted to do and that the Perspective API enabled.

(To skip ahead to the original task description, [see here](https://github.com/METResearchGroup/mind_technology_lab_experiments/issues/17), and to see the code implementation, [see here](https://github.com/METResearchGroup/mind_technology_lab_experiments/pull/20))

We have a labeled dataset of 26,000 moral outrage labels curated over Twitter data. We have been using the Perspective API as our classifier of choice, given how well calibrated it has generally been, though we've been moving towards simple LLM-based classifiers to replace it. We found that the Perspective API still remains better in class as compared to the LLM-based classifier, even after some prompt tuning. Initial tests had found that fine-tuning a model could possibly replace the Perspective API, but that would have taken more time and effort to build and to serve at large scale.

## Experiment setup

### Key questions

The key questions that I wanted to answer were:

1. What is the F1/accuracy/precision/recall of Jev vs. the alternatives?
2. What is the p50/p90/p99 of all the options? How about cost?
3. How calibrated is the probability from Jev compared to the Perspective API's moral outrage endpoint? Let's understand the statistical distribution of the differences. Let's also plot the distributions for each, as bar graphs.

We care not only about objective performance, but we care about measures such as cost and latency. We also care about calibration, as right now the API has been the best in class for a well-calibrated response. Even internally, we have found some issues with how the calibration for the Perspective API works. We have even had discussions with the team on adjusting our thresholds because of calibration inconsistencies across different classifier endpoints in the Perspective API. Therefore, I am hopeful that the calibration of the Jev model proves promising.

As this was just a quick test, I decided to get a sample of 1,000 of our 26,000 posts. The distribution of the labels was slightly imbalanced (440 with moral outrage, 560 without), but at least the balance of the labels in the sample matched that of the larger training set.

### Models used

The models I compared were:

1. Jev
2. Perspective API
3. LLMs (via AWS Bedrock): GPT 5.6 Luna, GPT 5.6 Terra, Claude Sonnet 5, Qwen3-32B, DeepSeek V3.1.

I wanted a mix of large and small models, as well as closed- and open-source models. I found that for social science tasks like this, you actually can't assume a model is going to do better just because it's newer or larger or smaller, because models tend to not be fine-tuned on this specific task. There are cases where I find that a new model class might actually degrade performance compared to an old model class. This is why I test a variety of LLMs here.

## Results

### Model performance

Jev performed just as well, if not better, than the other models attempted here. The F1 was higher than the Perspective API and on par with the latest frontier models and open-source models. It performed comparatively on the other metrics and especially outperformed on the recall, which was the metric that we cared most about, in addition to the F1 score, in our work.

| model name | f1 | accuracy | precision | recall |
| --- | ---: | ---: | ---: | ---: |
| Jev | 0.7323 | 0.717 | 0.6272 | 0.8795 |
| Perspective API | 0.7253 | 0.7734 | 0.7816 | 0.6765 |
| Bedrock:us.openai.gpt-5.6-luna | 0.7395 | 0.759 | 0.7052 | 0.7773 |
| Bedrock:us.openai.gpt-5.6-terra | 0.7354 | 0.764 | 0.7257 | 0.7455 |
| Bedrock:us.anthropic.claude-sonnet-5 | 0.7512 | 0.745 | 0.6581 | 0.8750 |
| Bedrock:qwen.qwen3-32b-v1:0 | 0.7490 | 0.754 | 0.6796 | 0.8341 |
| Bedrock:deepseek.v3-v1:0 | 0.7529 | 0.727 | 0.6256 | 0.9455 |

Comparing this to the Perspective API, the performance of Jev seems to be better than the Perspective API's own fine-tuned moral outrage classifier. It sacrifices a little bit of precision but makes up for it with a large increase in recall and, overall, a higher F1. Given that the Perspective API has been the best in class for this type of social science task for years now (especially for a low-cost solution), the fact that Jev outperforms it is very promising.

Comparing this to the frontier models, it's surprising that a general model such as this can perform just as well as the frontier models. It's unclear when the training cutoffs were, but it looks like, at minimum, the Jev model does not sacrifice in-world knowledge in order to provide its results.

### Cost/latency

Cost and latency were where Jev shined the most. We don't have Perspective API benchmarks here (I used cached labels), but the Perspective API is free and latency is still higher than Jev.

#### Latency (ms)

| model name | p50 | p90 | p99 |
| --- | ---: | ---: | ---: |
| Jev | 130.9 | 186.6 | 274.4 |
| Perspective API | 0.0 | 0.0 | 0.0 |
| Bedrock:us.openai.gpt-5.6-luna | 1667.9 | 2170.0 | 3612.5 |
| Bedrock:us.openai.gpt-5.6-terra | 1601.9 | 2360.6 | 3385.8 |
| Bedrock:us.anthropic.claude-sonnet-5 | 1898.9 | 2854.7 | 4415.3 |
| Bedrock:qwen.qwen3-32b-v1:0 | 387.6 | 511.5 | 913.3 |
| Bedrock:deepseek.v3-v1:0 | 318.6 | 349.7 | 495.8 |

Here, Jev shines compared to its LLM counterparts. Something to note here is that the P50, P90, and P99 latencies for Jev are actually pretty tightly bounded and consistent. At most, the P99 is 2x that of the P50, which is a smaller range than each of the LLM models, which range from 2 to 4x as much, except for Deepseek. The fastest mixture-of-experts models from Qwen and Deepseek were low latency themselves, but definitely still slower than Jev, being at minimum 2 to 3x slower while providing comparable results. In contrast, the frontier models from OpenAI and Anthropic were 10x to 25x slower than Jev while providing similar results.

The latency of the Jev responses is remarkable, especially given our non-trivial sample batch used here (n=1,000). I'm sure that some of the best infra engineers in the world are working at OpenAI and Anthropic, so I doubt that it's a matter of purely engineering talent and inference expertise that's allowing Jev to have such high performance. It likely is something about the architecture of the model and it being non-autoregressive. I would imagine that this would actually exponentially become more clear as you turn this from a single binary classification task into a multi-class task or into a multi-question classification task.

#### Cost

| model name | total tokens | estimated cost USD |
| --- | ---: | ---: |
| Jev | 374632 | 0 |
| Perspective API | 0 | 0 |
| Bedrock:us.openai.gpt-5.6-luna | 196397 | 0.133295 |
| Bedrock:us.openai.gpt-5.6-terra | 159156 | 0.841370 |
| Bedrock:us.anthropic.claude-sonnet-5 | 179268 | 0.742248 |
| Bedrock:qwen.qwen3-32b-v1:0 | 139782 | 0.029061 |
| Bedrock:deepseek.v3-v1:0 | 115569 | 0.075850 |

At the time of writing, Jev costs $0.042 per million input tokens, with output tokens completely free. I've not broken down this by input and output tokens, but it's safe to say that this is orders of magnitude cheaper than the LLM models. For comparison:

| Model Tier             | Input Price (per 1M) | Cached Input (per 1M) | Output Price (per 1M) |
|------------------------|---------------------:|----------------------:|----------------------:|
| ☀️ GPT-5.6 Sol         | $4.00                | $0.40                 | $20.00                |
| 🌍 GPT-5.6 Terra       | $2.00                | $0.20                 | $12.00                |
| 🌙 GPT-5.6 Luna        | $0.20                | $0.02                 | $1.20                 |

For example, Jev costs 50x less than GPT5.6 Terra, and 5x less than GPT-5.6 Luna (OpenAI's lowest-cost frontier model), while performing comparably.

### Calibration

I wanted to compare the calibration of Jev's probabilities against that of the Perspective API. I ignore LLM base probabilities here, and I choose not to ask the LLM for its own probabilities, given that there's no reason for an LLM to be well-calibrated. In fact, plenty of research ([see here](https://aclanthology.org/2024.naacl-long.366/) and [see here](https://aclanthology.org/2025.findings-acl.1316.pdf)) has shown that LLMs are poor at calibrating their own confidence.

We've had problems with interpreting the probabilities that the Perspective API returns. The calibration across classifiers is inconsistent, and we have received differing guidance from the team for how to properly threshold the classifiers. It's difficult to properly calibrate a classifier like this, but the labels for the Perspective API come from a majority vote from three independent annotators. The impact this has is that we find that probabilities tend to stack closely along the 0, 1/3, 2/3, and 1 points, which correspond to none, one, two, or three annotators giving a particular label. This has been a known limitation that we accept, given the ubiquity of the model and that it is generally superior to any model that any lab themselves would have built, but this is a limitation nonetheless.

We found that the Jev probabilities more cleanly map to what are likely appropriate levels of ambiguity. Even if Jev isn't absolutely spot-on in terms of calibration, it clearly is better than the Perspective API's bimodal distribution.

![Perspective API probability distribution](/assets/images/2026-09-19-using-jev-for-classification/perspective_api_probabilities.png)
  
![Jev probability distribution](/assets/images/2026-09-19-using-jev-for-classification/jev_probabilities.png)

In addition, a spot-check of the posts at different Jev probabilities matches my own intuition: the posts that Jev puts at a more confident probability tend to be more unambiguously keep or remove, while Jev's confidence probabilities tend to match what I'd expect; given two posts, whichever post Jev predicted to have a higher probability of having moral outrage, I also agreed, and whenever the gap in the two predicted probabilities were close, I was also similarly unsure.

## Takeaways

This new Jev model is very impressive, and I am only scratching the surface for ways that we could use it in our research. Essentially, now, for any classification task that we would have used a general LLM for, or we would have tried to fine-tune our own smaller SLM or a router model for, we can now use this model, like Jev. It maintains best-in-class performance on our social science research tasks, being comparable to frontier models, if not exceeding them, while being orders of magnitude cheaper and easier to scale. I am very impressed by the quality of this model. This is definitely worth the hype, and there's a lot of potential to use this across the board in research and beyond.
