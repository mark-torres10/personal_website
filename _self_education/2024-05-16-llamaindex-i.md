# Experimenting with LlamaIndex Pt. I
As part of me learning more about LLMs, I'm working through different use cases, tools, and examples that I find online. Right now, I'm working through the [example notebooks](https://github.com/run-llama/llama_index/blob/main/docs/docs/examples/) from the LlamaIndex Github repo.

## Overview of LlamaIndex
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

We then create an `ObjectIndex`, which will allow us to use an index structure over arbitrary objects. This index handles serializing to/from the object as well as connecting it to a proper index for storage. That way, we can query for tools in the same way that we would otherwise query for documents. We don't have to tell an LLM "use this tool for this use case" or "here is all the information from all the tools". Rather, the index lets us choose relevant tools just like how we can for relevant documents. We can have a large collection of possible tools, and instead of us having to specify how each tool is used, we can store these tools so that the LLM can query it just like it would a large collection of possible documents.

During query time, the agent can retrieve the correct tool (just like how agents can query the correct docs for normal RAG) for the task.

```python
# define an "object" index over these tools
from llama_index.core import VectorStoreIndex
from llama_index.core.objects import ObjectIndex

obj_index = ObjectIndex.from_objects(
    all_tools,
    index_cls=VectorStoreIndex,
)
```

Now we create an OpenAI agent that can use an `ObjectRetriever` (built on top of the `ObjectIndex`) to retrieve the set of relevant tools for a query. These tools, or specifically their docstrings, are then passed to the LLM as context.

```python
from llama_index.agent.openai import OpenAIAgent
retriever = obj_index.as_retriever(similarity_top_k=2)
agent = OpenAIAgent.from_tools(
    tool_retriever=retriever, verbose=True
)
```

Now we can see how it works:
```python
agent.chat("What's 212 multiplied by 122? Make sure to use Tools")
```

```plaintext
Added user message to memory: What's 212 multiplied by 122? Make sure to use Tools
=== Calling Function ===
Calling function: multiply with args: {"a": 212, "b": 122}
Got output: 25864
========================

=== Calling Function ===
Calling function: useless_12 with args: {"a": 212, "b": 122}
Got output: None
========================

AgentChatResponse(
    response='212 multiplied by 122 is 25864.',
    sources=[
        ToolOutput(
            content='25864',
            tool_name='multiply',
            raw_input={'args': (), 'kwargs': {'a': 212, 'b': 122}},
            raw_output=25864,
            is_error=False
        ),
        ToolOutput(
            content='None',
            tool_name='useless_12',
            raw_input={'args': (), 'kwargs': {'a': 212, 'b': 122}},
            raw_output=None, is_error=False
        )
    ],
    source_nodes=[],
    is_dummy_stream=False
)
```

We see that the agent can understand the query, figure out which tools to use, and then use those tools to aid it in finding the right answer.

## Example 2: OpenAI ReAct Agent with Tools
We can build off the previous example and use a ReAct agent that uses tools. The original notebook is [here](https://colab.research.google.com/github/run-llama/llama_index/blob/main/docs/docs/examples/agent/react_agent.ipynb) and my version is [here](https://colab.research.google.com/drive/1loW2KD88riE_oVGNpTpI6GiNfI6DkCQA).

First, we do some setup:

```python
%pip install llama-index-llms-openai
!pip install llama-index
```

```python
import os
import getpass

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")
```

```python
from llama_index.core.agent import ReActAgent
from llama_index.llms.openai import OpenAI
from llama_index.core.llms import ChatMessage
from llama_index.core.tools import BaseTool, FunctionTool
```

Now we define our tools. We'll use `FunctionTool`, which will process the docstring and parameter signature and use that for retrieval plus make it available for the agent as context.

```python
def multiply(a: int, b: int) -> int:
    """Multiply two integers and returns the result integer"""
    return a * b


multiply_tool = FunctionTool.from_defaults(fn=multiply)

def add(a: int, b: int) -> int:
    """Add two integers and returns the result integer"""
    return a + b


add_tool = FunctionTool.from_defaults(fn=add)
```

Now we'll run some queries. We'll use GPT3.5 and then set up our ReAct agent with the LLM and tools. The agent will perform ReAct using these and think step by step about what it should do to answer the question and how to use the tools that it has in order to answer the question. Having the agent over the tools is a nice abstraction so that we ourselves don't have to deal with the logic of:
- processing the output of the LLM
- calling the correct tool if the LLM says to use the tool
- passing in the output of that tool to the LLM, and
- re-prompting the LLM with the updated context

All done over and over until the LLM makes a final answer.

```python
llm = OpenAI(model="gpt-3.5-turbo-instruct")
agent = ReActAgent.from_tools([multiply_tool, add_tool], llm=llm, verbose=True)
```

```python
response = agent.chat("What is 20+(2*4)? Calculate step by step ")
```

```plaintext
Thought: The current language of the user is: English. I need to use a tool to help me answer the question.
Action: add
Action Input: {'a': 20, 'b': 8}
Observation: 28
Thought: I can answer without using any more tools. I'll use the user's language to answer
Answer: 28
```

We can see the agent using the LLM to reason over the prompt, then deciding what tool to use based off the output of the LLM, using said tool with the correct inputs and function call, and then passing that output back to the LLM to give an updated answer. If we look at the underlying prompt, we can better see how this ReAct logic works and how our agent is prompting the LLM in order to use the tools and the history of the conversation in order to come to a solution through a step-by-step, ReAct-style logic:

```python
prompt_dict = agent.get_prompts()
for k, v in prompt_dict.items():
    print(f"Prompt: {k}\n\nValue: {v.template}")
```

```plaintext
Prompt: agent_worker:system_prompt

Value: You are designed to help with a variety of tasks, from answering questions to providing summaries to other types of analyses.

## Tools

You have access to a wide variety of tools. You are responsible for using the tools in any sequence you deem appropriate to complete the task at hand.
This may require breaking the task into subtasks and using different tools to complete each subtask.

You have access to the following tools:
{tool_desc}


## Output Format

Please answer in the same language as the question and use the following format:

```
Thought: The current language of the user is: (user's language). I need to use a tool to help me answer the question.
Action: tool name (one of {tool_names}) if using a tool.
Action Input: the input to the tool, in a JSON format representing the kwargs (e.g. {{"input": "hello world", "num_beams": 5}})
```

Please ALWAYS start with a Thought.

Please use a valid JSON format for the Action Input. Do NOT do this {{'input': 'hello world', 'num_beams': 5}}.

If this format is used, the user will respond in the following format:

```
Observation: tool response
```

You should keep repeating the above format till you have enough information to answer the question without using any more tools. At that point, you MUST respond in the one of the following two formats:

```
Thought: I can answer without using any more tools. I'll use the user's language to answer
Answer: [your answer here (In the same language as the user's question)]
```

```
Thought: I cannot answer the question with the provided tools.
Answer: [your answer here (In the same language as the user's question)]
```

## Current Conversation

Below is the current conversation consisting of interleaving human and assistant messages.
```

We can update the prompt if we would like:
```python
from llama_index.core import PromptTemplate

react_system_header_str = """\

You are designed to help with a variety of tasks, from answering questions \
    to providing summaries to other types of analyses.

## Tools
You have access to a wide variety of tools. You are responsible for using
the tools in any sequence you deem appropriate to complete the task at hand.
This may require breaking the task into subtasks and using different tools
to complete each subtask.

You have access to the following tools:
{tool_desc}

## Output Format
To answer the question, please use the following format.

```
Thought: I need to use a tool to help me answer the question.
Action: tool name (one of {tool_names}) if using a tool.
Action Input: the input to the tool, in a JSON format representing the kwargs (e.g. {{"input": "hello world", "num_beams": 5}})
```

Please ALWAYS start with a Thought.

Please use a valid JSON format for the Action Input. Do NOT do this {{'input': 'hello world', 'num_beams': 5}}.

If this format is used, the user will respond in the following format:

```
Observation: tool response
```

You should keep repeating the above format until you have enough information
to answer the question without using any more tools. At that point, you MUST respond
in the one of the following two formats:

```
Thought: I can answer without using any more tools.
Answer: [your answer here]
```

```
Thought: I cannot answer the question with the provided tools.
Answer: Sorry, I cannot answer your query.
```

## Additional Rules
- The answer MUST contain a sequence of bullet points that explain how you arrived at the answer. This can include aspects of the previous conversation history.
- You MUST obey the function signature of each tool. Do NOT pass in no arguments if the function expects arguments.

## Current Conversation
Below is the current conversation consisting of interleaving human and assistant messages.

"""
react_system_prompt = PromptTemplate(react_system_header_str)
agent.update_prompts({"agent_worker:system_prompt": react_system_prompt})
```

Now that we've updated the prompt, we can see what it would do:
```python
agent.reset()
response = agent.chat("What is 5+3+2")
print(response)
```

```plaintext
Thought: I need to use a tool to help me answer the question.
Action: add
Action Input: {"a": 5, "b": 3}
Observation: 8
Thought: I need to use a tool to help me answer the question.
Action: add
Action Input: {"a": 8, "b": 2}
Observation: 10
Thought: I can answer without using any more tools.
Answer: 10
10
```
## Example 3: ReAct agent with query engine for RAG

