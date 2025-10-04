---
layout: single
title: "Creating grounded AI experts"
date: 2025-09-30 10:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-09-30-creating-grounded-ai-experts
---

# Creating grounded AI experts

I've been meticulously building out my [AI tooling and workflow](https://github.com/mark-torres10/ai_tools) over the past few months and a core part of it is my development of "AI experts", personas with particular skillsets or knowledge bases whose expertise I can rely on.

## What I've been experimenting with recently

I've done some consulting work with frontier AI labs which has given me some insight into how they think about refining and prompting language models. Among other things, I've seen some very well-designed and rigorous data preprocessing and postprocessing pipelines as well as prompting. Rubrics are everywhere, and having rubrics helps steer models into having more consistent results (as well as helps humans and LLM-as-a-judge models evaluate results). Datasets like SWE-bench (and its derivates, such as SWE-bench verified, MLE-bench, etc) showcase the importance of well-curated, distilled gold-label data in steering models to give better responses ("correct" isn't enough, as there are a million technically correct things that an LLM can say, so steering it towards "this is what a human expert would say" leads to much better results).

From this, I've recently been looking at two improvements to my AI expert generation workflow

- using rubrics
- grounding in gold-label data

I've found great results by taking my typical persona generation prompts, such as [this prompt](https://github.com/mark-torres10/ai_tools/blob/main/agents/task_instructions/create_personas/CREATE_ENGINEERING_PERSONA.md), and complementing them with [another prompt](https://github.com/mark-torres10/ai_tools/blob/main/agents/task_instructions/create_personas/RESEARCH_GROUNDED_PERSONA_CREATION.md) meant to push the AI agent to find gold-label examples online (e.g., guides, interviews, etc) and use that to create rubrics, checklists, and best practices that an AI expert can easily iterate through and review. Now, instead of saying "I want an AI expert in backend engineering", I have an AI backend engineering expert that says "here's a checklist of 3 common pitfalls that people make in this sort of circumstance, let's see if you did that". I've found this to drastically improve the consistency of the AI expert performance, allowed me to better trust the advice and recommendations it gives, and results in me prompting less as the AI experts are more likely to do the evaluations and reviews that I actually wanted in the first place.

I tried something similar with Claude 4 but it wasn't always consistently staying on task. I started getting better success with GPT-5 and then exponentially better success with Claude 4.5. Now it's something that I do whenever I need to create AI experts, as doing it in this way allows me to ground the experts in actual real-world expertise while also giving them a consistent set of criteria and rubrics to evaluate anything I ask them.

Overall, I've had great success with using AI experts in my workflow already, and I've recently found more success by, during the expert creation process, prompting the AI agent to do a deep search online for gold label samples and structuring the AI expert as one that not only has a particular domain of expertise, but also has a set of checklists and rubrics, grounded by best practices and gold-label examples, that it uses in its analyses.
