---
layout: single
title:  "Experimenting with LlamaIndex, Pt. I"
date:   2024-05-16 07:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /self_education/llamaindex-i
---

# Experimenting with LlamaIndex, Pt. I
As part of me learning more about LLMs, I'm working through different use cases, tools, and examples that I find online. Right now, I'm working through the [example notebooks](https://github.com/run-llama/llama_index/blob/main/docs/docs/examples/) from the LlamaIndex Github repo. Each of the following examples is a fully-worked through example, based on the notebooks that are in the LlamaIndex examples repo.

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

## Example 3: OpenAI assistant agent
We can use LlamaIndex along with [OpenAI](https://platform.openai.com/docs/assistants/overview) in order to create an AI assistant agent.
The original notebook is [here](https://github.com/run-llama/llama_index/blob/main/docs/docs/examples/agent/openai_assistant_agent.ipynb) and my version is [here](https://colab.research.google.com/drive/1IQP42Px32HWfVaYnOjbJ7V20pZzDji0r).

First, we do some setup:
```python
%pip install llama-index-agent-openai
%pip install llama-index-vector-stores-supabase
!pip install llama-index
```

```python
import os
import getpass

os.environ["OPENAI_API_KEY"] = getpass.getpass("OpenAI API Key: ")
```

### Simple assistant agent
First, let's just us the out-of-the-box agent without any extra tooling. We can use the code interpreter that OpenAI offers instead of providing our own tool.

```python
from llama_index.agent.openai import OpenAIAssistantAgent

agent = OpenAIAssistantAgent.from_new(
    name="Math Tutor",
    instructions="You are a personal math tutor. Write and run code to answer math questions.",
    openai_tools=[{"type": "code_interpreter"}],
    instructions_prefix="Please address the user as Jane Doe. The user has a premium account.",
)

agent.thread_id # thread_Eq4zpjr4mlNa6dA9kY9HIlLS
```

```python
response = agent.chat(
    "I need to solve the equation `3x + 11 = 14`. Can you help me?"
)
print(str(response))
```

```plaintext
Certainly! The equation you've provided is a simple linear equation in one variable which can be solved with algebra. Here are the steps to solve the equation \(3x + 11 = 14\):

1. Subtract 11 from both sides of the equation to isolate the term with the variable \(x\) on one side:
\[3x + 11 - 11 = 14 - 11\]
2. Simplify both sides of the equation:
\[3x = 3\]
3. Divide both sides of the equation by 3 to solve for \(x\):
\[\frac{3x}{3} = \frac{3}{3}\]
4. Simplify the division to find the value of \(x\):
\[x = 1\]

So the solution to the equation is \(x = 1\).
```

### Assistant with the native LlamaIndex vector store and query engine
Now we can try a similar task, but using the vector story and query engine available in LlamaIndex.

First, we load the necessary imports:
```python
from llama_index.agent.openai import OpenAIAssistantAgent
from llama_index.core import (
    SimpleDirectoryReader,
    VectorStoreIndex,
    StorageContext,
    load_index_from_storage,
)

from llama_index.core.tools import QueryEngineTool, ToolMetadata
```

We then create persistent data stores for our sample data and then download the data. We'll be using financial reports from Uber and Lyft.

```python
try:
    storage_context = StorageContext.from_defaults(
        persist_dir="./storage/lyft"
    )
    lyft_index = load_index_from_storage(storage_context)

    storage_context = StorageContext.from_defaults(
        persist_dir="./storage/uber"
    )
    uber_index = load_index_from_storage(storage_context)

    index_loaded = True
except:
    index_loaded = False
```

```python
!mkdir -p 'data/10k/'
!wget 'https://raw.githubusercontent.com/run-llama/llama_index/main/docs/docs/examples/data/10k/uber_2021.pdf' -O 'data/10k/uber_2021.pdf'
!wget 'https://raw.githubusercontent.com/run-llama/llama_index/main/docs/docs/examples/data/10k/lyft_2021.pdf' -O 'data/10k/lyft_2021.pdf'
```

At this point, we should be able to see the data stored locally.
![Screenshot of current notebook status](/assets/images/2024-05-16-llamaindex-i/uber-lyft-data-notebook-screenshot.png)

We now load the data and create an index for each:

```python
from llama_index.core.schema import Document as LlamaIndexDocument
from llama_index.core.indices.vector_store.base import VectorStoreIndex

if not index_loaded:
    # load data
    lyft_docs: list[LlamaIndexDocument] = SimpleDirectoryReader(
        input_files=["./data/10k/lyft_2021.pdf"]
    ).load_data()
    uber_docs: list[LlamaIndexDocument] = SimpleDirectoryReader(
        input_files=["./data/10k/uber_2021.pdf"]
    ).load_data()

    # build index
    lyft_index: VectorStoreIndex = VectorStoreIndex.from_documents(lyft_docs)
    uber_index: VectorStoreIndex = VectorStoreIndex.from_documents(uber_docs)

    # persist index
    lyft_index.storage_context.persist(persist_dir="./storage/lyft")
    uber_index.storage_context.persist(persist_dir="./storage/uber")
```

The `SimpleDirectoryReader` automatically chooses the best file reader given the file. For our use case, 

After creating an index for each, our data should look like this:
![Screenshot of current notebook status after creating each index](/assets/images/2024-05-16-llamaindex-i/uber-lyft-data-indices.png)

Let's take a look at what one of these docs looks like.
```python
print(lyft_docs[0])
```

```plaintext
Doc ID: 223c3b51-1c8a-4cbc-bdda-26db81f32032
Text: UNITED STATESSECURITIES AND EXCHANGE COMMISSION Washington, D.C.
20549 FORM 10-K   (Mark One) ☒ ANNUAL REPORT PURS UANT TO SECTION 13
OR 15(d) OF THE SECURITIES EXCHANGE ACT OF 1934For the fiscal year
ended December 31, 2021 OR ☐ TRANSITION REPORT PURS UANT TO SECTION 13
OR 15(d) OF THE SECURITIES EXCHANGE ACT OF 1934 FOR THE TRANSITION
PERIODFR...
```

We can compare this to a picture of the PDF:
![Picture of Lyft PDF](/assets/images/2024-05-16-llamaindex-i/lyft-2021-pdf-picture.png)

It looks like the first document consists of the text extracted from the PDF. The PDF has 238 pages, and if we look at the length of `lyft_docs`, we can see that it equals 238, meaning that each document contains the text extracted from one page of the PDF.

We create an index for each of the documents. This will allow us to query each index given a query and then obtain the documents most relevant to that query. We can now build a query engine on top of each index. Let's create a query engine that, given a query, fetches the top `k=3` documents relaetd to the query.

```python
from llama_index.core.query_engine.retriever_query_engine import RetrieverQueryEngine

lyft_engine: RetrieverQueryEngine = lyft_index.as_query_engine(similarity_top_k=3)
uber_engine: RetrieverQueryEngine = uber_index.as_query_engine(similarity_top_k=3)
```

Now we can create a `Tool` object out of each query engine. This lets us use each query engine as a tool for our LLM agent.

```python
query_engine_tools = [
    QueryEngineTool(
        query_engine=lyft_engine,
        metadata=ToolMetadata(
            name="lyft_10k",
            description=(
                "Provides information about Lyft financials for year 2021. "
                "Use a detailed plain text question as input to the tool."
            ),
        ),
    ),
    QueryEngineTool(
        query_engine=uber_engine,
        metadata=ToolMetadata(
            name="uber_10k",
            description=(
                "Provides information about Uber financials for year 2021. "
                "Use a detailed plain text question as input to the tool."
            ),
        ),
    ),
]
```

We see a common framework here for working with data on LlamaIndex, which is:
1. Load and process data as documents
2. Create a way to access/interact with the data:
    1. Create an index
    2. Create a query engine on top of the index.
3. Create a tool on top that lets the agent interact with the data.
4. Pass the tool to the agent.

We can now create an engine and try it out for ourselves:

```python
agent = OpenAIAssistantAgent.from_new(
    name="SEC Analyst",
    instructions="You are a QA assistant designed to analyze sec filings.",
    tools=query_engine_tools,
    instructions_prefix="Please address the user as Jerry.",
    verbose=True,
    run_retrieve_sleep_time=1.0,
)
```

```python
response = agent.chat("What was Lyft's revenue growth in 2021?")
```

```plaintext
=== Calling Function ===
Calling function: lyft_10k with args: {"input":"What was Lyft's revenue growth in 2021?"}
Got output: Lyft's revenue increased by 36% in 2021 compared to the prior year.
========================
```

We see that our agent called the `lyft_10k` tool, based off the docstring description of the tool, and passed along the correct args required to access our index and fetch the docs that are most related.

### Assistant with an external vector store (Supabase)
Let's try to recreate our exercise from above, but now use an external database. Like in the base notebook, we'll also use Supabase.

#### Creating a Supabase account.
Let's first start by creating a Supabase account. Go to [this](https://supabase.com/dashboard/sign-in?returnTo=%2Fprojects) link to create an account. Since this is a demo, I'll just create an account and use a free trial.

After creating a new account, the website should look like this:
![Supabase dashboard](/assets/images/2024-05-16-llamaindex-i/supabase-intro-dashboard.png)

Then, create a new project:
![Supabase create project](/assets/images/2024-05-16-llamaindex-i/supabase-create-new-project.png)

This will return a set of credentials that we can then use to access our database.

#### Set up access to Supabase
Now we can set up our code in LlamaIndex:

```python
from llama_index.agent.openai import OpenAIAssistantAgent
from llama_index.core import (
    SimpleDirectoryReader,
    VectorStoreIndex,
    StorageContext,
)
from llama_index.vector_stores.supabase import SupabaseVectorStore

from llama_index.core.tools import QueryEngineTool, ToolMetadata
```

```python
# load data
reader = SimpleDirectoryReader(input_files=["./data/10k/lyft_2021.pdf"])
docs = reader.load_data()
for doc in docs:
    doc.id_ = "lyft_docs"
```

```python
os.environ["SUPABASE_PASSWORD"] = getpass.getpass("Supabase password: ")
supabase_password = os.getenv("SUPABASE_PASSWORD")
```

#### Store document vectors in Supabase
We now use the `SupabaseVectorStore` integration to connect LlamaIndex to our Supabase database. Note, I had to change the connection string, which Supabase provides, to use "postgresql" instead of "postgres", as per this [StackOverflow thread](https://stackoverflow.com/questions/71000246/sqlalchemy-exc-nosuchmoduleerror-cant-load-plugin-sqlalchemy-dialectspostgre) (yes, I used StackOverflow instead of ChatGPT. Weird, right?).

```python
vector_store = SupabaseVectorStore(
    postgres_connection_string=(
        f"postgresql://postgres.jhiypislszswpmsvctqa:{supabase_password}@aws-0-ap-southeast-1.pooler.supabase.com:5432/postgres"
    ),
    collection_name="base_demo",
)
storage_context = StorageContext.from_defaults(vector_store=vector_store)
index = VectorStoreIndex.from_documents(docs, storage_context=storage_context)
```

Let's now check that the docs are in the vector store. First we can check this from our notebook:
```python
# sanity check that the docs are in the vector store
num_docs = vector_store.get_by_id("lyft_docs", limit=1000)
print(len(num_docs))
```
We can then check what this looks like in Supabase. LlamaIndex stores our documents as vectors and if we look at Supabase we can actually see how the storage works. Each document is represented by a vector, an ID, and some metadata.
![Supabase vector store](/assets/images/2024-05-16-llamaindex-i/supabase-vector-store.png)

#### Create tools and agent
Now we follow the same procedure as before. The only difference between this version of our agent and the previous one is that our document vectors are stored in an external vector store (via Supabase) instead of a local vector store (via LlamaIndex's default storage).

Let's create a tool to query the index, an agent to use that tool, and then test that agent.

```python
lyft_tool = QueryEngineTool(
    query_engine=index.as_query_engine(similarity_top_k=3),
    metadata=ToolMetadata(
        name="lyft_10k",
        description=(
            "Provides information about Lyft financials for year 2021. "
            "Use a detailed plain text question as input to the tool."
        ),
    ),
)
agent = OpenAIAssistantAgent.from_new(
    name="SEC Analyst",
    instructions="You are a QA assistant designed to analyze SEC filings.",
    tools=[lyft_tool],
    verbose=True,
    run_retrieve_sleep_time=1.0,
)
response = agent.chat(
    "Tell me about Lyft's risk factors, as well as response to COVID-19"
)
print(str(response))
```

```plaintext
Lyft's 2021 10-K filing outlines a multifaceted risk landscape for the company, encapsulating both operational and environmental challenges that could impact its business model:

- **Economic Factors**: Risks include the ramifications of the COVID-19 pandemic, susceptibility to natural disasters, the volatility of economic downturns, and geopolitical tensions.

- **Operational Dynamics**: The company is cognizant of its limited operating history, the uncertainties surrounding its financial performance, the intense competition in the ridesharing sector, the unpredictability in financial results, and the ambiguity tied to the expansion potential of the rideshare market.

- **Human Capital**: A critical concern is the ability of Lyft to attract and maintain a robust network of both drivers and riders, which is essential for the platform's vitality.

- **Insurance and Safety**: Ensuring adequate insurance coverage for stakeholders and addressing autonomous vehicle technology risks are pivotal.

- **Reputation and Brand**: Lyft is attentive to the influence that illegal or unseemly activities on its platform can have on its public image.

- **Pricing Structure**: Changing pricing models pose a risk to Lyft's revenue streams, considering how essential pricing dynamics are to maintaining competitive service offerings.

- **Systemic Integrity**: Lyft also acknowledges risks emanating from potential system failures which could disrupt service continuity.

Furthermore, Lyft is vigilant about regulatory and legal risks that could lead to litigation and is conscious of the broader implications of climate change on its operations.

In terms of its response to COVID-19, Lyft has adopted strategic measures to secure the welfare of both its workforce and customer base:

1. **Health and Safety Protocols**: Lyft has instituted health and safety mandates tailored specifically to the ridesharing experience in view of the pandemic.

2. **Workplace Adjustments**: The company revised its workplace policies to accommodate the shifts in the work environment precipitated by the pandemic.

3. **Financial Adaptations**: To synchronize with the revenue contraction experienced during the pandemic, Lyft executed monetary realignments, which necessitated workforce reductions in 2020.

These initiatives reflect Lyft's calculated approach to navigating the operational and financial hurdles enacted by the COVID-19 pandemic. By prioritizing health and safety, nimbly altering corporate practices, and recalibrating fiscal management, Lyft aimed to buttress its business against the storm of the pandemic while setting a foundation for post-pandemic recovery.
```

We can see now how to use LlamaIndex, OpenAI, and Supabase to create an assistant agent that loads documents from an external vector store, creates a tool to query said documents, and creates an agent that can use those tools to answer questions passed to the LLM.

## Example 4: OpenAI agent query planning

## Example 5: 
