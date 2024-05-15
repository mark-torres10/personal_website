---
layout: single
title:  "LLMOps self-education, Pt. III"
date:   2024-05-15 05:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /self_education/llm-course-part-iii
---

# Session III: Creating a more robust RAQA system using LlamaIndex
I'm working on improving my RAG-building skills. As a part of that, I'm taking courses online. The course I'm currently taking is from [AI Makerspace](https://github.com/AI-Maker-Space), as part of their [LLMOps](https://github.com/AI-Maker-Space/LLM-Ops-Cohort-1) curriculum.

The related project notebook is [here](https://github.com/AI-Maker-Space/LLM-Ops-Cohort-1/blob/main/Week%202/Tuesday/LLamaIndex%20RAQA%20Tool%20(Assignment%20Version).ipynb), and my own version of that notebook [here](https://colab.research.google.com/drive/158ww0tHZBUIKt1K_B162oJpMwDUdyGjy).

## Setup
Like before, we'll be working on creating an AI chat agent to answer questions about either the Barbie or Oppenheimer movies. However, instead of Langchain, we'll be using a different framework called [LlamaIndex](https://www.llamaindex.ai/).

### Overview of LlamaIndex
[LlamaIndex](https://www.llamaindex.ai/) is, as they themselves describe it, a data framework built for LLMs. [LlamaIndex does well](https://github.com/run-llama/llama_index?tab=readme-ov-file#-overview) at integrating multiple different data sources, creating ways to structure the data so it can be easily queried or indexed, and exposing helper tools for easily querying and fetching the data when needed. By default, LlamaIndex uses OpenAI as its LLM provider, though it supports integrations with [many](https://python.langchain.com/v0.1/docs/integrations/llms/) LLMs. LlamaIndex often is used with Langchain in order to build end-to-end LLM agent interfaces.

## Example applications with LlamaIndex
First we'll go over a few simple and short example notebooks to see how LlamaIndex works. These are all directly from the [LlamaIndex Github repo](https://github.com/run-llama/llama_index/tree/main/docs/docs/examples) of example notebooks and recipes.

### Example 1: OpenAI Agent
We'll start with a simple LLM agent, using OpenAI, that uses tools in order to do some math. The original notebook is [here](https://colab.research.google.com/github/run-llama/llama_index/blob/main/docs/docs/examples/agent/openai_agent_retrieval.ipynb) and my version is [here](https://colab.research.google.com/drive/146W9ebHE2wtq4_3WqmbV5bzArLiCr8Ym).

First, we need to install dependencies:
```python
%pip install llama-index-agent-openai-legacy
```

We set up our OpenAI API key:
```python
import os
import getpass

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")
```

We do some imports.
```python
import json
from typing import Sequence

from llama_index.core.tools import BaseTool, FunctionTool
```

Then, we create some simple tools for our LLM agent to use:
```python
def multiply(a: int, b: int) -> int:
    """Multiply two integers and returns the result integer"""
    return a * b


def add(a: int, b: int) -> int:
    """Add two integers and returns the result integer"""
    return a + b


def useless(a: int, b: int) -> int:
    """Toy useless function."""
    pass

multiply_tool = FunctionTool.from_defaults(fn=multiply, name="multiply")
useless_tools = [
    FunctionTool.from_defaults(fn=useless, name=f"useless_{str(idx)}")
    for idx in range(28)
]
add_tool = FunctionTool.from_defaults(fn=add, name="add")

all_tools = [multiply_tool] + [add_tool] + useless_tools
all_tools_map = {t.metadata.name: t for t in all_tools}
```

We then create an `ObjectIndex`

### Example 2:

### Example 3:

