---
layout: single
title:  "LLMOps self-education, Pt. II"
date:   2024-05-15 05:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /self_education/llm-course-part-ii
---

# Session II: Questioning Barbie and Oppenheimer Through the Use of Agents
I'm working on improving my RAG-building skills. As a part of that, I'm taking courses online. The course I'm currently taking is from [AI Makerspace](https://github.com/AI-Maker-Space), as part of their [LLMOps](https://github.com/AI-Maker-Space/LLM-Ops-Cohort-1) curriculum.

The related project notebook is [here](https://github.com/AI-Maker-Space/LLM-Ops-Cohort-1/blob/main/Week%201/Thursday/Agent%20Powered%20Barbenheimer%20Application%20(Assignment%20Version).ipynb), and my own version of that notebook [here](https://colab.research.google.com/drive/1q6NZYUOcplDcAJFJAXMtFh-my73wOzjg).

I'll also follow along with [this](https://github.com/google/generative-ai-docs/blob/main/examples/gemini/python/langchain/Gemini_LangChain_QA_Chroma_WebLoad.ipynb) Langchain notebook that uses [Gemini](https://ai.google.dev/models/gemini) for our embeddings in lieu of OpenAI's embeddings.

## Setup

First, we install the correct packages and pass in our API key.

```python
!pip install -q -U langchain langchain-google-genai
```

```python
import os
import getpass

os.environ['GOOGLE_API_KEY'] = getpass.getpass('Gemini API Key:')
```

## Contruct a Barbie retrieval agent
We'll create a Barbie retriever, which will allow us to ask questions about the Barbie film. For our case, we'll use the Wikipedia entry about the Barbie movie. We'll embed the entry into a vector database and then query it to get information about the movie.

We'll use Langchain's [WikipediaLoader](https://python.langchain.com/v0.1/docs/integrations/document_loaders/wikipedia/), which lets us pass a query into Wikipedia and get results.

We'll use Wikipedia in addition to the .csv file of movie reviews from the [last](https://markptorres.com/self_education/llm-course-part-i) portion. Our goal is to then create an [ensemble retriever](https://python.langchain.com/v0.1/docs/modules/data_connection/retrievers/ensemble/). An ensemble retriever takes documents from multiple sources and then [reranks](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf) the documents to return the documents most relevant for a query.

### Set up document loaders and text splitters
We'll set up the document loaders and text splitters for both the Wikipedia data as well as the .csv file of Barbie movie reviews.

```python
!pip install -q -U wikipedia
```

```python
load_max_docs = 1
doc_content_chars_max = 1_000_000
```

We load the documents from both the Wikipedia page as well as the Barbie movie reviews.
```python
from langchain.document_loaders import WikipediaLoader, CSVLoader

barbie_wikipedia_docs = WikipediaLoader(
    query="Barbie (film)", 
    load_max_docs=load_max_docs,
    doc_content_chars_max=doc_content_chars_max
).load()

barbie_csv_docs = CSVLoader(
    file_path="barbie.csv",
    source_column="Review"
).load()
```

```python
len(barbie_wikipedia_docs) # 1, for our 1 doc
len(barbie_csv_docs) # 75
```


Now we create document splitters for each of these sets of docs.
```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

wikipedia_text_splitter = RecursiveCharacterTextSplitter(
    chunk_size = 500,
    chunk_overlap = 0,
    length_function = len,
    is_separator_regex= False,
    separators = ["\n==", "\n", " "] # keep headings, then paragraphs, then sentences
)

csv_text_splitter = RecursiveCharacterTextSplitter(
    chunk_size = 1000,
    chunk_overlap = 50,
    length_function = len,
    is_separator_regex= False,
    separators = ["\n", " "] # keep paragraphs, then sentences
)
```

```python
chunked_barbie_wikipedia_docs = wikipedia_text_splitter.transform_documents(barbie_wikipedia_docs)
chunked_barbie_csv_docs = csv_text_splitter.transform_documents(barbie_csv_docs)
```

```python
print(len(chunked_barbie_wikipedia_docs)) # 192
print(len(chunked_barbie_csv_docs)) # 105
```

```python
chunked_barbie_wikipedia_docs[0]
```
```plaintext
Document(
    page_content='Barbie is a 2023 fantasy comedy film \
    directed by Greta Gerwig from a screenplay she wrote \
    with Noah Baumbach. Based on the eponymous fashion \
    dolls by Mattel, it is the first live-action Barbie \
    film after numerous animated films and specials. It \
    stars Margot Robbie as the title character and Ryan \
    Gosling as Ken, and follows them on a journey of \
    self-discovery through both Barbieland and the real \
    world following an existential crisis. It is also a \
    commentary regarding patriarchy and the effects of'
    metadata={
        'title': 'Barbie (film)',
        'summary': 'Barbie is a 2023 fantasy comedy film \
        directed by Greta Gerwig from a screenplay she \
        wrote with Noah Baumbach. Based on ...,
        'source': 'https://en.wikipedia.org/wiki/Barbie_(film)'
    }
)
```

### Set up embeddings and vector store
Now we set up a FAISS vector store and use our Gemini embeddings.

```python
from langchain.vectorstores import FAISS
from langchain.embeddings import CacheBackedEmbeddings
from langchain.storage import LocalFileStore
from langchain_google_genai import GoogleGenerativeAIEmbeddings
```

```python
# If there is no environment variable set for the API key, you can pass the API
# key to the parameter `google_api_key` of the `GoogleGenerativeAIEmbeddings`
# function: `google_api_key = "key"`.
core_embeddings_model = GoogleGenerativeAIEmbeddings(model="models/embedding-001")

store = LocalFileStore("./shared_cache/")

embedder = CacheBackedEmbeddings.from_bytes_store(
    core_embeddings_model, store, namespace=core_embeddings_model.model
)
```

Now we set up our FAISS vector store using this embedder and store. We'll use the Gemini embedder to create our embeddings, in lieu of the OpenAI ones that we used previously.

#### Comparison of dense vs. sparse retrieval
We can choose between dense or sparse retrieval for our documents. What's the difference? This [Reddit](https://www.reddit.com/r/MachineLearning/comments/z76uel/d_difference_between_sparse_and_dense_information/) thread was really insightful for learning about dense and sparse retrieval.

##### Dense retrieval
Dense retrieval refers to using dense vectors for representing documents. Dense vectors refer to dense representations, such as those learned from an encoding layer of some pre-trained model like BERT or GPT. For these types of vectors, we can only compare similarity by comparing the vector distances between embeddings. At scale, we can't exactly determine which vectors are closest to each other so we use [approximate nearest neighbors](https://www.cs.cmu.edu/~agray/approxnn.pdf) for our similarity matching.

##### Sparse retrieval
In sparse retrieval, we represent our documents as sparse vectors, meaning that they are represented by vectors whose values are mostly zeroes. This represented word vector representations prior to algorithms like Word2Vec, so some examples of sparse representations would be something like ["Bag of Words"](https://en.wikipedia.org/wiki/Bag-of-words_model), which represents a document as a vector where the $i^{th}$ index represents the $i^{th}$ word in the vocabulary and the value is the frequency of that word in the document. Since most words won't appear in a document, most entries will be zero.

Using sparse representations opens up faster and more efficient retrieval methods than distance metrics. For example, we can use [BM25](https://en.wikipedia.org/wiki/Okapi_BM25), which measures the overlap in "important words" between the query and the document. 

##### Using dense vs. sparse retrieval
Dense retrieval helps us fetch relevant documents by seeing which documents are most "similar" to our query. This is good for many common query use cases, and it gives us flexibility to use natural language as dense embedding representations should capture the semantic "meaning" of our query and know which documents have a similar semantic meaning.

However, sparse retrieval helps us do more "keyword-related" searches, where we have specific keywords that we want information on and we want our result to have specifically those keywords. This can complement dense retrieval by, for example, allowing us to make shorter queries or hone in on important terms.

We can combine both using an [ensemble retriever](https://python.langchain.com/v0.1/docs/modules/data_connection/retrievers/ensemble/), which lets us fetch results from both dense and sparse retrieval, reranks them using [this](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf) algorithm, and then presents us with the top $k$ results.

#### Setting up our Barbie retrievers
For the Barbie reviews, we'll create our embeddings using a dense retriever. We'll convert our FAISS store as a retriever.
```python
barbie_csv_faiss_retriever = FAISS.from_documents(
    chunked_barbie_csv_docs, embedder
).as_retriever()
```

For the Barbie Wikipedia page, we'll use sparse retrieval using BM25.

```python
from langchain.retrievers import BM25Retriever

# set up BM25 retriever
barbie_wikipedia_bm25_retriever = BM25Retriever.from_documents(
    chunked_barbie_wikipedia_docs
)
barbie_wikipedia_bm25_retriever.k = 1

# set up FAISS vector store
barbie_wikipedia_faiss_store = FAISS.from_documents(
    chunked_barbie_wikipedia_docs, embedder
)

barbie_wikipedia_faiss_retriever = (
    barbie_wikipedia_faiss_store.as_retriever(
        search_kwargs={"k": 1}
    )
)
```

We can now set up our ensemble retriever. Given that we have more docs from Wikipedia than our Barbie reviews, we can bias the reranker to prefer the sparse Wikipedia docs over the dense Barbie review vectors. 

```python
from langchain.retrievers import EnsembleRetriever

# set up ensemble retriever
barbie_ensemble_retriever = EnsembleRetriever(
    retrievers=[
        barbie_wikipedia_bm25_retriever
        barbie_wikipedia_faiss_retriever
    ],
    weights=[0.25, 0.75] # should sum to 1
)
```

### Creating our retrieval agent

Now, we have our retrievers, which, when given a query, can fetch relevant information from both the Barbie reviews and the Barbie Wikipedia page. We can now create a retrieval agent. An LLM [agent](https://huggingface.co/blog/open-source-llms-as-agents) is an LLM that can make make observations about their environment and act accordingly, often with tools. [This](https://huggingface.co/blog/open-source-llms-as-agents) Hugging Face blog post has a great introduction to LLM agents. [This](https://arxiv.org/pdf/2309.07864) paper has a good overview of the different types of existing LLM agents, although given how fast the field is moving this document is likely out of date.

LLMs are great at tasks such as summarization, comprehension, and generalization, but struggle greatly with hallucination. However, if we give tools to LLMs, we give them a way to do things such as calculate math problems (with a calculator), run code (with a code execution environment), look up the weather (using the Weather app), or, in our case, give us information about Barbie.

For our agent, we'll need to:
1. Create the retrieval tool, which is a tool that will let the agent access our ensemble retriever.
2. Set up the agent. The agent will take a query and then, if appropriate, use the tool to fetch information from our retrievers in order to answer questions about Barbie. 

#### Creating retriever tools for Barbie
We can create [retriever tools](https://api.python.langchain.com/en/latest/tools/langchain.tools.retriever.create_retriever_tool.html) for fetching information from our retrievers. The tool is an abstraction that connects our retriever to our LLM. We give the tool a name and describe what it does, and this is passed to the LLM in a format like:

```plaintext
You have access to the following tools:
- [tool name]: [tool function]
```

We can set up different tools for each of our retrievers and then combine them into a list.

```python
from langchain.agents.agent_toolkits import create_retriever_tool

barbie_wikipedia_retrieval_tool = create_retriever_tool(
    barbie_ensemble_retriever, 
    "Wikipedia",
    "Searches and returns documents regarding the plot, history, and cast of the Barbie movie"
)

barbie_csv_retrieval_tool = create_retriever_tool(
    barbie_csv_faiss_retriever,
    "PublicReviews",
    "Searches and returns documents regarding public reviews of the Barbie movie"
)

barbie_retriever_tools = [
    barbie_wikipedia_retrieval_tool,
    barbie_csv_retrieval_tool
]
```

#### Creating the retrieval agent
Now that we have the tools, we can combine then into a retrieval agent. Since we're using Gemini for our embeddings, we also need to use Gemini for our chat model as well.  [This](https://github.com/GoogleCloudPlatform/generative-ai/blob/main/gemini/reasoning-engine/intro_reasoning_engine.ipynb) is a good resource for how to do so, and we'll follow the Langchain chat resource from [this](https://python.langchain.com/v0.1/docs/modules/agents/agent_types/structured_chat/) documentation.

```python
!pip install langchainhub
```

```python
from langchain import hub
from langchain.agents import AgentExecutor, create_structured_chat_agent
from langchain_google_genai import ChatGoogleGenerativeAI
```

First, we get a prompt that will be the template for our chat interface. We'll import one so we don't have to hard-code it ourselves.

```python
prompt = hub.pull("hwchase17/structured-chat-agent")
```

Then, we define the LLM that we'll be using. Note, I found that if you use "gemini-pro" or "gemini-1.0", you get a 400 error as those models don't support multiturn chats. I found [this](https://github.com/google-gemini/generative-ai-python/issues/278) Github issue to be helpful; turns out the problem is fixed using "gemini-1.5".
```python
model_name = "gemini-1.5-pro-latest"
llm = ChatGoogleGenerativeAI(model=model_name, temperature=0)
```

Now we create our chat agent:
```python
barbie_agent = create_structured_chat_agent(
    llm, barbie_retriever_tools, prompt
)
```

We now create an agent executor, which executes queries on the agent. The [Langchain](https://python.langchain.com/v0.1/docs/modules/agents/concepts/#agentexecutor) docs discuss why we use an agent executor, which is that the agent executor helps manage the execution of the agent, catches errors, passes the intermediate inputs back to the agent, and more. According to these docs, the pseudocode for the executor is like the following:

```plaintext
next_action = agent.get_action(...)
while next_action != AgentFinish:
    observation = run(next_action)
    next_action = agent.get_action(..., next_action, observation)
return next_action
```

Also according to the docs, the executor also handles complexities such as:
1. Handling cases where the agent selects a non-existent tool
2. Handling cases where the tool errors
3. Handling cases where the agent produces output that cannot be parsed into a tool invocation
4. Logging and observability at all levels (agent decisions, tool calls) to stdout and/or to LangSmith.

##### Testing our Barbie retrieval agent

Let's try our agent and see how it looks!

```python
barbie_retriever_agent_executor(
  {"input" : """
    Did people like Barbie, or did they find it too Philosphical? \
    If they did, can you tell me why the movie is so Philosophical?"""
  }
```

```plaintext
> Entering new AgentExecutor chain...
Could not parse LLM output: ```json
{
 "response": "The Barbie movie has received a mixed reception, with some viewers praising its philosophical themes and others finding them too heavy-handed or confusing.  Here's a breakdown of why the movie is considered philosophical:\n\n* **Existentialism:** The movie explores themes of identity, purpose, and the meaning of life, particularly from Barbie's perspective as she grapples with her existence in both Barbieland and the real world. \n* **Feminism and Gender Roles:** The film critiques societal expectations of women and the patriarchy, highlighting the challenges and contradictions women face in a male-dominated world. \n* **Consumerism and Capitalism:**  The movie satirizes the commercialization of Barbie and the impact of consumer culture on society. \n* **The Nature of Reality:** The film blurs the lines between fantasy and reality, prompting viewers to question their own perceptions and the nature of truth. \n\nWhether these philosophical themes resonate with viewers depends on individual preferences and interpretations. Some find the movie's exploration of these themes thought-provoking and insightful, while others find them preachy or distracting from the overall entertainment value. Ultimately, the reception of the movie's philosophical elements is subjective."
}
```Invalid or incomplete responseThought: I need to look up public reviews of the movie to see what people thought.
Action:
```json
{
  "action": "PublicReviews",
  "action_input": {"query": "What did people think of the philosophical themes in the Barbie movie?"}
}
```: 27
Review_Date: 19 July 2023
Author: dreopdreef
Rating: 6
Review_Title: Not so unique as expected
Review: The concept of a Barbie movie is really unique and got me all exited. But unfortunately to me it delivers the same messages that we keep hearing as of lately.
Review_Url: /review/rw9201199/?ref_=tt_urv

: 50
Review_Date: 29 July 2023
Author: Vic_max
Rating: 7
Review_Title: A lot like The Lego Movie - except for the 2nd half
Review: As a guy who never played with Barbie dolls, I was surprised to be caught up in the enthusiasm to see this movie; I purchased tickets well in advance when they first went on sale.
Review_Url: /review/rw9222678/?ref_=tt_urv

: 2
Review_Date: 22 July 2023
Author: Natcat87
Rating: 6
Review_Title: Too heavy handed
Review: As a woman that grew up with Barbie, I was very excited for this movie. I was curious to see how they would evolve the "stereotypical Barbie" into something more. But the messaging in this movie was so heavy handed that it completely lost the plot. I consider myself a proponent of gender equality, and this ain't the way to get it.
Review_Url: /review/rw9206938/?ref_=tt_urv

why they act like this and stuff but i didnt really get into the movie to be honest: i only identify myself when they talked about society and business but all the musical things, the choregraphy and even the morality was kind of superficial. But i enjoyed that Barbie realised the problems of our world and compare to her own. Some moments was funny and some more serious or drama sometimes even if its pretty soft because we are in Barbie Land. I mean i spend a great time but it was kind of ok. And i still think its a great Barbie movie: the task was not easy to represent that world but i enjoyed the set design, the colors, the camera movements. Its good anyway.```json
{
  "action": "PublicReviews",
  "action_input": {"query": "Did people find the Barbie movie too philosophical?"}
}
```: 50
Review_Date: 29 July 2023
Author: Vic_max
Rating: 7
Review_Title: A lot like The Lego Movie - except for the 2nd half
Review: As a guy who never played with Barbie dolls, I was surprised to be caught up in the enthusiasm to see this movie; I purchased tickets well in advance when they first went on sale.
Review_Url: /review/rw9222678/?ref_=tt_urv

: 2
Review_Date: 22 July 2023
Author: Natcat87
Rating: 6
Review_Title: Too heavy handed
Review: As a woman that grew up with Barbie, I was very excited for this movie. I was curious to see how they would evolve the "stereotypical Barbie" into something more. But the messaging in this movie was so heavy handed that it completely lost the plot. I consider myself a proponent of gender equality, and this ain't the way to get it.
Review_Url: /review/rw9206938/?ref_=tt_urv

: 27
Review_Date: 19 July 2023
Author: dreopdreef
Rating: 6
Review_Title: Not so unique as expected
Review: The concept of a Barbie movie is really unique and got me all exited. But unfortunately to me it delivers the same messages that we keep hearing as of lately.
Review_Url: /review/rw9201199/?ref_=tt_urv

: 45
Review_Date: 10 August 2023
Author: adamjohns-42575
Rating: 8
Review_Title: A sheer pleasure to watch.
Review: Barbie (2023) -
Review_Url: /review/rw9251197/?ref_=tt_urvAction:
```json
{
  "action": "Final Answer",
  "action_input": "The Barbie movie has received mixed reviews regarding its philosophical themes. Some viewers found them thought-provoking and insightful, while others found them too heavy-handed or repetitive. The movie explores themes like existentialism, feminism, consumerism, and the nature of reality. Whether or not someone finds it \"too philosophical\" is subjective and depends on their individual preferences."
}
```

> Finished chain.
{'input': '\n    Did people like Barbie, or did they find it too Philosphical?     If they did, can you tell me why the movie is so Philosophical?',
 'output': 'The Barbie movie has received mixed reviews regarding its philosophical themes. Some viewers found them thought-provoking and insightful, while others found them too heavy-handed or repetitive. The movie explores themes like existentialism, feminism, consumerism, and the nature of reality. Whether or not someone finds it "too philosophical" is subjective and depends on their individual preferences.'}
```

We see that the agent executor runs the query, manages any errors, and keeps rerunning the agent with more and more evidence until the LLM has enough context to give a definitive answer. Once the executor finishes running, we get the following result:

```plaintext
{
    'input': 'Did people like Barbie, or did they find \
            it too Philosphical? If they did, can you \
            tell me why the movie is so Philosophical?',
    'output': 'The Barbie movie has received mixed \
            reviews regarding its philosophical \
            themes. Some viewers found them \
            thought-provoking and insightful, \
            while others found them too heavy-handed \
            or repetitive. The movie explores themes \
            like existentialism, feminism, consumerism, \
            and the nature of reality. Whether or not \
            someone finds it "too philosophical" is \
            subjective and depends on their individual \
            preferences.'
}
```
Looks like it works well! We now have an end-to-end agent executor that, given any question, can fetch information from our database of Barbie reviews as well as the Barbie movie page on Wikipedia, and then use that information to answer the question.

## Construct an Oppenheimer retrieval chain

We'll repurpose a lot of the same logic from the Barbie retrieval agent and do the same for Oppenheimer. Instead of an agent, we'll look at using chains. Unlike agents, which are given a set of tools and figure out what actions it should take, chains are a predetermined set of steps (a "chain" of logic) for how to answer a question. [This](https://www.reddit.com/r/LangChain/comments/14mbyjm/chains_vs_agents/) Reddit thread and [this](https://medium.com/@kbdhunga/exploring-langchain-chains-agents-a-quick-overview-9d0a8c4d7ba0) Medium article both have great overviews comparing the two. Often times, as per [this thread](https://community.deeplearning.ai/t/agents-vs-chains-vs-tools/516148), agents can also use chains within their work.

### Setup steps
Since the logic is the same as above, I'll skip the explanations and just add the code until we get to the new portions.

First, we load the documents.
```python
oppenheimer_wikipedia_docs = WikipediaLoader(
    query="Oppenheimer (film)",
    load_max_docs=1,
    doc_content_chars_max=1_000_000
).load()

# https://github.com/AI-Maker-Space/LLM-Ops-Cohort-1/blob/main/Week%202/Tuesday/oppenheimer_data/oppenheimer.csv
oppenheimer_csv_docs = CSVLoader(
    file_path="oppenheimer.csv",
    source_column="Review_Url"
).load()
```

Then we split the documents.
```python
chunked_opp_wikipedia_docs = wikipedia_text_splitter.transform_documents(oppenheimer_wikipedia_docs)
chunked_opp_csv_docs = csv_text_splitter.transform_documents(oppenheimer_csv_docs)
```

Then we set up our retrievers.
```python
opp_csv_faiss_retriever = FAISS.from_documents(
    chunked_opp_csv_docs, embedder
).as_retriever()

# set up BM25 retriever
opp_wikipedia_bm25_retriever = BM25Retriever.from_documents(
    chunked_opp_wikipedia_docs
)

opp_wikipedia_bm25_retriever.k = 1

# set up FAISS vector store
opp_wikipedia_faiss_store = FAISS.from_documents(
    chunked_opp_wikipedia_docs, embedder
)

opp_wikipedia_faiss_retriever = opp_wikipedia_faiss_store.as_retriever(search_kwargs={"k": 1})

# set up ensemble retriever
opp_ensemble_retriever = EnsembleRetriever(
    retrievers=[
        opp_wikipedia_bm25_retriever,
        opp_wikipedia_faiss_retriever
    ],
    weights=[0.25, 0.75] # should sum to 1
)
```

### Creating the chain
To create the chain, we'll use Langchain Expression Language (LCEL). According to the [docs](https://python.langchain.com/v0.1/docs/expression_language/), LCEL is a useful, expressive, and robust way of creating chains. It makes use of a pipe operator and the steps are composed in a very expressive and readable way.

For example, for a simple LLM app, the LCEL for the chain could look like:
```python
chain = prompt | model | output_parser
```
For a RAG app, the LCEL for the chain could look like:
```python
chain = setup_and_retrieval | prompt | model | output_parser
```

A full example, based on the [docs](https://python.langchain.com/v0.1/docs/expression_language/get_started/), could look something like this:

```python
from langchain_community.vectorstores import DocArrayInMemorySearch
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableParallel, RunnablePassthrough
from langchain_openai import OpenAIEmbeddings

vectorstore = DocArrayInMemorySearch.from_texts(
    ["harrison worked at kensho", "bears like to eat honey"],
    embedding=OpenAIEmbeddings(),
)
retriever = vectorstore.as_retriever()

template = """Answer the question based only on the following context:
{context}

Question: {question}
"""
prompt = ChatPromptTemplate.from_template(template)
output_parser = StrOutputParser()

setup_and_retrieval = RunnableParallel(
    {"context": retriever, "question": RunnablePassthrough()}
)
chain = setup_and_retrieval | prompt | model | output_parser

chain.invoke("where did harrison work?")
```

For our case, instead of us using our retrievers to determine which information is most useful, we'll pass in all the information to the context itself and ask the LLM to figure out what information is most useful. This means that we'll need to include all of the documents as part of our prompt. We'll set up a chain that will load the documents into the prompt and then prompt the LLM to use any information necessary to make its response.

```python
from langchain.prompts import ChatPromptTemplate

system_message = """Use the information from the below two sources to answer any questions.

Source 1: public user reviews about the Oppenheimer movie
<source1>
{source1}
</source1>

Source 2: the wikipedia page for the Oppenheimer movie including the plot summary, cast, and production information
<source2>
{source2}
</source2>
"""

prompt = ChatPromptTemplate.from_messages([("system", system_message), ("human", "{question}")])
```

We set up our chain like this:
```python
oppenheimer_multisource_chain = {
    "source1": (lambda x: x["question"]) | opp_ensemble_retriever,
    "source2": (lambda x: x["question"]) | opp_csv_faiss_retriever,
    "question": lambda x: x["question"],
} | prompt | llm
```

Now we can invoke the chain:

```python
oppenheimer_multisource_chain.invoke(
    {"question" : "What did people think of the Oppenheimer movie?"}
)
```

We can take a look at the response:
```python
AIMessage(
    content="Based on the provided sources, here's \
    a summary of public and critical opinion on the \
    Oppenheimer movie:\n\n**Critical Acclaim:**\n\
    \n* **Widespread praise:** Critics lauded the film \
    for its screenplay, cast performances (especially \
    Cillian Murphey and Robert Downey Jr.), and \
    stunning visuals.\n* **Nolan's best?** Many \
    ranked it among director Christopher Nolan's \
    finest works and as one of 2023's best films. \
    Some even hailed it as one of the century's \
    best.\n* **Rotten Tomatoes:**  A highly positive \
    93% approval rating from critics, with an average \
    score of 8.6/10.\n\n**Public Reception:**\n\
    \n* **Mixed reactions:** While many found it \
    a cinematic masterpiece, others felt it was \
    overhyped and tedious.\n* **Impactful but \
    difficult:** Some viewers found the film \
    emotionally draining, reflecting the heavy \
    subject matter.\n* **Glorification vs. \
    Victim:**  There's debate on whether the \
    film glorifies Oppenheimer or portrays him \
    as a victim of circumstance burdened by \
    his creation.\n* **Lack of horror?:** Some, \
    particularly those connected to Hiroshima, \
    felt the film didn't adequately depict the \
    horrors of nuclear weapons.\n\n**In \
    conclusion:**  Oppenheimer was a critically \
    acclaimed film with powerful performances \
    and visuals. However, its reception was more \
    mixed among the general public, with some \
    finding it emotionally challenging and others \
    debating its portrayal of Oppenheimer and \
    the consequences of his actions. \n",
    response_metadata={
        'prompt_feedback': {'block_reason': 0, 'safety_ratings': []}
        'finish_reason': 'STOP',
        'safety_ratings': [
            {
                'category': 'HARM_CATEGORY_SEXUALLY_EXPLICIT'
                'probability': 'NEGLIGIBLE',
                'blocked': False
            },
            {
                'category': 'HARM_CATEGORY_HATE_SPEECH',
                'probability': 'NEGLIGIBLE',
                'blocked': False
            },
            {
                'category': 'HARM_CATEGORY_HARASSMENT',
                'probability': 'NEGLIGIBLE',
                'blocked': False
            },
            {
                'category': 'HARM_CATEGORY_DANGEROUS_CONTENT'
                'probability': 'NEGLIGIBLE',
                'blocked': False
            }
        ]
    },
    id='run-8c66a05a-add4-41b1-bedf-68266ff1eee4-0'
)
```

It looks great! Unlike in the agent case, we created a chain to pass in all our docs in the prompt so that the LLM could choose what evidence would be useful for it to consider.

## Creating a single agent combining Barbie and Oppenheimer
Previously, we created an agent that could interact with the Barbie data. Now let's create a single agent that can interact with both the Barbie and the Oppenheimer data.

### Setting up the tools
Just like how we set up the previous agent, which had tools that could be used to get relevant information, we'll also give our new agent tools. We'll use our Barbie agent and our Oppenheimer chain as tools for our new agent to have at its disposal.

First, we have to create a custom function to be able to use our Oppenheimer chain. This is because the chain expects a particular format for its input:

```python
def query_oppenheimer(input):
    return oppenheimer_multisource_chain.invoke({"question" : input})
```

Then we have to create `Tool` instances that use our Barbie agent and Oppenheimer chain:

```python
from langchain.agents import Tool

def query_oppenheimer(input):
    return oppenheimer_multisource_chain.invoke({"question" : input})

tools = [
    Tool(
        name = "BarbieInfo",
        func=barbie_retriever_agent_executor.invoke,
        description="useful for when you need to answer questions about Barbie. Input should be a fully formed question."
    ),
    Tool(
        name = "OppenheimerInfo",
        func=query_oppenheimer,
        description="useful for when you need to answer questions about Oppenheimer. Input should be a fully formed question."
    ),
]
```

### Creating the agent
We'll now create our agent. For our use case, we want an agent that can implement the [ReAct](https://react-lm.github.io/) logic. [This](https://huggingface.co/blog/open-source-llms-as-agents) article gives a great introduction into ReAct, which is a way of prompting that tells an LLM to (1) reason (Re) about what it should do and (2) Act (Act) based on that. We ask the LLM to think about what it should do, perform an action, and then either re-evaluate based on the output of that action or determine that it has answered the question sufficiently. [Here](https://ai.google.dev/docs/react_gemini_prompting) is another example that uses the original ReAct prompting.

According to the blog post, the LLM essentially calls this prompt in a loop:

```plaintext
Here is a question: "{question}" 
You have access to these tools: {tools_descriptions}. 
You should first reflect with ‘Thought: {your_thoughts}’, then you either:
- call a tool with the proper JSON formatting,
- or your print your final answer starting with the prefix ‘Final Answer:
```

Then the agent executor parses the answer and
- if it contains the string ‘Final Answer:’, the loop ends and you print the answer,
- else, the LLM should have output a tool call: you can parse this output to get the tool name and arguments, then call said tool with said arguments. Then the output of this tool call is appended to the prompt, and you call the LLM again with this extended information, until it has enough information to finally provide a final answer to the question.

For our case, we essentially follow the same steps as before, but we import a different LLM agent. Instead of the chat agent as before, we import an LLM agent for performing ReAct.

```python
from langchain.agents import AgentExecutor, create_react_agent
```
Like before, we also pull in a prompt optimized for ReAct instead of creating our own

```python
prompt = hub.pull("hwchase17/react")
```

We'll also re-use the same Gemini model as before:
```python
model_name = "gemini-1.5-pro-latest"
llm = ChatGoogleGenerativeAI(model=model_name, temperature=0)
```

Now, like before, we put it all together.
```python
barbenheimer_agent = create_react_agent(llm, tools, prompt)
```

Now we can create our AgentExecutor as before.

```python

```

### Checking out the results
Great! Let's see how it looks:

```python
barbenheimer_agent_chain.invoke(
    {"input" : "What did people like about the Barbie movie?"}
)
```

```plaintext
> Entering new AgentExecutor chain...
Thought: I need to find information about the Barbie movie.
Action: BarbieInfo
Action Input: What did people like about the Barbie movie?

> Entering new AgentExecutor chain...
Based on the available reviews, people liked several aspects of the Barbie movie:

1. Visual Spectacle: The movie is praised for its wonderful visual spectacle, indicating that the visuals and cinematography were impressive.

2. Creative Physical Comedy: The movie is described as having a great deal of creative physical comedy, suggesting that the humor and comedic elements were well-received by viewers.

3. Empowerment Theme: Some viewers appreciated the theme of empowerment in the movie, which resonated with them and made the film a big hit.

4. Darkly Satirical Look at Gender Structure: The movie is commended for taking a darkly satirical look at gender structure in society, providing a unique and thought-provoking perspective on the subject.

5. Emotional Engagement: The main storyline of Barbie's existential crisis is mentioned as being emotionally engaging, indicating that it connected with the audience on an emotional level.

6. Funny Comedy: The comedy in the movie is described as funny, with some viewers mentioning that it elicited laughs from everyone in the cinema.

7. Music and Choreographed Dances: The music and choreographed dances in the movie are mentioned as being enjoyable, adding to the overall entertainment value.

8. Nostalgic Representation of Barbie: The movie is praised for capturing the doll-like aesthetic and nostalgic representation of the Barbie brand, with elements such as plastic and fake food and drink, gliding cars, and actors who move and interact in a doll-like manner.

It's important to note that these are just a few highlights from the available reviews, and individual opinions may vary.

> Finished chain.

Observation: {'input': 'What did people like about the Barbie movie?', 'chat_history': [AIMessage(content="People liked several aspects of the Barbie movie. Here are some of the positive reviews:\n\n- The movie is described as a wonderful visual spectacle with creative physical comedy.\n- It is praised for its theme of empowerment, which resonated with many viewers.\n- The film takes a darkly satirical look at gender structure in society, providing a unique and thought-provoking perspective.\n- The main storyline of Barbie's existential crisis is emotionally engaging.\n- The comedy in the movie is funny and elicited laughs from the audience.\n- The music and choreographed dances are enjoyable.\n- The movie captures the doll-like aesthetic, with plastic and fake food and drink, gliding cars, and actors who move and interact in a doll-like manner.\n\nOverall, the Barbie movie received positive feedback for its visual appeal, themes, humor, and nostalgic representation of the Barbie brand.", additional_kwargs={}, example=False), HumanMessage(content='What did people like about the Barbie movie?', additional_kwargs={}, example=False), AIMessage(content="Based on the available reviews, people liked several aspects of the Barbie movie:\n\n1. Visual Spectacle: The movie is praised for its wonderful visual spectacle, indicating that the visuals and cinematography were impressive.\n\n2. Creative Physical Comedy: The movie is described as having a great deal of creative physical comedy, suggesting that the humor and comedic elements were well-received by viewers.\n\n3. Empowerment Theme: Some viewers appreciated the theme of empowerment in the movie, which resonated with them and made the film a big hit.\n\n4. Darkly Satirical Look at Gender Structure: The movie is commended for taking a darkly satirical look at gender structure in society, providing a unique and thought-provoking perspective on the subject.\n\n5. Emotional Engagement: The main storyline of Barbie's existential crisis is mentioned as being emotionally engaging, indicating that it connected with the audience on an emotional level.\n\n6. Funny Comedy: The comedy in the movie is described as funny, with some viewers mentioning that it elicited laughs from everyone in the cinema.\n\n7. Doll-Like Aesthetic: The movie is praised for capturing a doll-like aesthetic, with plastic and fake food and drink, gliding cars, and actors who move and interact in a doll-like manner. This aspect adds to the charm and nostalgia of the Barbie brand.\n\nIt's important to note that these are just a few highlights from the available reviews, and individual opinions may vary.", additional_kwargs={}, example=False), HumanMessage(content='What did people like about the Barbie movie?', additional_kwargs={}, example=False), AIMessage(content="Based on the available reviews, people liked several aspects of the Barbie movie:\n\n1. Visual Spectacle: The movie is praised for its wonderful visual spectacle, indicating that the visuals and cinematography were impressive.\n\n2. Creative Physical Comedy: The movie is described as having a great deal of creative physical comedy, suggesting that the humor and comedic elements were well-received by viewers.\n\n3. Empowerment Theme: Some viewers appreciated the theme of empowerment in the movie, which resonated with them and made the film a big hit.\n\n4. Darkly Satirical Look at Gender Structure: The movie is commended for taking a darkly satirical look at gender structure in society, providing a unique and thought-provoking perspective on the subject.\n\n5. Emotional Engagement: The main storyline of Barbie's existential crisis is mentioned as being emotionally engaging, indicating that it connected with the audience on an emotional level.\n\n6. Funny Comedy: The comedy in the movie is described as funny, with some viewers mentioning that it elicited laughs from everyone in the cinema.\n\n7. Music and Choreographed Dances: The music and choreographed dances in the movie are mentioned as being enjoyable, adding to the overall entertainment value.\n\n8. Nostalgic Representation of Barbie: The movie is praised for capturing the doll-like aesthetic and nostalgic representation of the Barbie brand, with elements such as plastic and fake food and drink, gliding cars, and actors who move and interact in a doll-like manner.\n\nIt's important to note that these are just a few highlights from the available reviews, and individual opinions may vary.", additional_kwargs={}, example=False), HumanMessage(content='What did people like about the Barbie movie?', additional_kwargs={}, example=False), AIMessage(content="Based on the available reviews, people liked several aspects of the Barbie movie:\n\n1. Visual Spectacle: The movie is praised for its wonderful visual spectacle, indicating that the visuals and cinematography were impressive.\n\n2. Creative Physical Comedy: The movie is described as having a great deal of creative physical comedy, suggesting that the humor and comedic elements were well-received by viewers.\n\n3. Empowerment Theme: Some viewers appreciated the theme of empowerment in the movie, which resonated with them and made the film a big hit.\n\n4. Darkly Satirical Look at Gender Structure: The movie is commended for taking a darkly satirical look at gender structure in society, providing a unique and thought-provoking perspective on the subject.\n\n5. Emotional Engagement: The main storyline of Barbie's existential crisis is mentioned as being emotionally engaging, indicating that it connected with the audience on an emotional level.\n\n6. Funny Comedy: The comedy in the movie is described as funny, with some viewers mentioning that it elicited laughs from everyone in the cinema.\n\n7. Music and Choreographed Dances: The music and choreographed dances in the movie are mentioned as being enjoyable, adding to the overall entertainment value.\n\n8. Nostalgic Representation of Barbie: The movie is praised for capturing the doll-like aesthetic and nostalgic representation of the Barbie brand, with elements such as plastic and fake food and drink, gliding cars, and actors who move and interact in a doll-like manner.\n\nOverall, the Barbie movie received positive feedback for its visual appeal, themes, humor, and nostalgic representation of the Barbie brand.", additional_kwargs={}, example=False), HumanMessage(content='What did people like about the Barbie movie?', additional_kwargs={}, example=False), AIMessage(content="Based on the available reviews, people liked several aspects of the Barbie movie:\n\n1. Visual Spectacle: The movie is praised for its wonderful visual spectacle, indicating that the visuals and cinematography were impressive.\n\n2. Creative Physical Comedy: The movie is described as having a great deal of creative physical comedy, suggesting that the humor and comedic elements were well-received by viewers.\n\n3. Empowerment Theme: Some viewers appreciated the theme of empowerment in the movie, which resonated with them and made the film a big hit.\n\n4. Darkly Satirical Look at Gender Structure: The movie is commended for taking a darkly satirical look at gender structure in society, providing a unique and thought-provoking perspective on the subject.\n\n5. Emotional Engagement: The main storyline of Barbie's existential crisis is mentioned as being emotionally engaging, indicating that it connected with the audience on an emotional level.\n\n6. Funny Comedy: The comedy in the movie is described as funny, with some viewers mentioning that it elicited laughs from everyone in the cinema.\n\n7. Music and Choreographed Dances: The music and choreographed dances in the movie are mentioned as being enjoyable, adding to the overall entertainment value.\n\n8. Nostalgic Representation of Barbie: The movie is praised for capturing the doll-like aesthetic and nostalgic representation of the Barbie brand, with elements such as plastic and fake food and drink, gliding cars, and actors who move and interact in a doll-like manner.\n\nIt's important to note that these are just a few highlights from the available reviews, and individual opinions may vary.", additional_kwargs={}, example=False)], 'output': "Based on the available reviews, people liked several aspects of the Barbie movie:\n\n1. Visual Spectacle: The movie is praised for its wonderful visual spectacle, indicating that the visuals and cinematography were impressive.\n\n2. Creative Physical Comedy: The movie is described as having a great deal of creative physical comedy, suggesting that the humor and comedic elements were well-received by viewers.\n\n3. Empowerment Theme: Some viewers appreciated the theme of empowerment in the movie, which resonated with them and made the film a big hit.\n\n4. Darkly Satirical Look at Gender Structure: The movie is commended for taking a darkly satirical look at gender structure in society, providing a unique and thought-provoking perspective on the subject.\n\n5. Emotional Engagement: The main storyline of Barbie's existential crisis is mentioned as being emotionally engaging, indicating that it connected with the audience on an emotional level.\n\n6. Funny Comedy: The comedy in the movie is described as funny, with some viewers mentioning that it elicited laughs from everyone in the cinema.\n\n7. Music and Choreographed Dances: The music and choreographed dances in the movie are mentioned as being enjoyable, adding to the overall entertainment value.\n\n8. Nostalgic Representation of Barbie: The movie is praised for capturing the doll-like aesthetic and nostalgic representation of the Barbie brand, with elements such as plastic and fake food and drink, gliding cars, and actors who move and interact in a doll-like manner.\n\nIt's important to note that these are just a few highlights from the available reviews, and individual opinions may vary.", 'intermediate_steps': []}
Thought:I now know the final answer.
Final Answer: People liked several aspects of the Barbie movie, including its visual spectacle, creative physical comedy, empowerment theme, darkly satirical look at gender structure, emotional engagement, funny comedy, music and choreographed dances, and nostalgic representation of the Barbie brand.
```

Let's do another example:

```python
barbenheimer_agent_chain.invoke({"input" : "What did people like about the Oppenheimer movie?"})
```

```plaintext
> Entering new AgentExecutor chain...
Thought: I need to use the OppenheimerInfo tool to find out what people liked about the movie.
Action: OppenheimerInfo
Action Input: What did people like about the Oppenheimer movie?content="Based on the provided information, here are some aspects people appreciated about the Oppenheimer movie:\n\n* **Screenplay:** Critics lauded the writing, suggesting it was engaging and thought-provoking.\n* **Cast Performances:**  The performances, particularly Cillian Murphey as Oppenheimer and Robert Downey Jr. as Lewis Strauss, were highly praised.\n* **Visuals:** The film's visuals were captivating, likely due to Nolan's use of IMAX technology and practical effects. \n\nAdditionally, some viewers found the film to be:\n\n* **One of Nolan's best:** Many considered it among the director's strongest works.\n* **Emotionally impactful:** Some viewers reported feeling deeply affected by the film's themes and Oppenheimer's personal struggles.\n\nWhile the sources don't delve into specific plot points that resonated with audiences, the overall consensus points to a well-crafted and impactful cinematic experience. \n" response_metadata={'prompt_feedback': {'block_reason': 0, 'safety_ratings': []}, 'finish_reason': 'STOP', 'safety_ratings': [{'category': 'HARM_CATEGORY_SEXUALLY_EXPLICIT', 'probability': 'NEGLIGIBLE', 'blocked': False}, {'category': 'HARM_CATEGORY_HATE_SPEECH', 'probability': 'NEGLIGIBLE', 'blocked': False}, {'category': 'HARM_CATEGORY_HARASSMENT', 'probability': 'NEGLIGIBLE', 'blocked': False}, {'category': 'HARM_CATEGORY_DANGEROUS_CONTENT', 'probability': 'NEGLIGIBLE', 'blocked': False}]} id='run-d4cd505a-fc26-436c-b504-57d9b0771ab4-0'Thought: I now know the final answer.
Final Answer: People enjoyed the Oppenheimer movie for its screenplay, cast performances, and visuals. Some viewers considered it one of director Christopher Nolan's best works and found it emotionally impactful. 


> Finished chain.
{
    'input': 'What did people like about the Oppenheimer movie?',
    'output': "People enjoyed the Oppenheimer movie for \
    its screenplay, cast performances, and visuals. Some \
    viewers considered it one of director Christopher \
    Nolan's best works and found it emotionally impactful."
}
```

The agent works great! It can answer questions about both the Barbie and Oppenheimer movies. It knows which tool to call when, and the Barbie/Oppenheimer tools that it uses are in themselves compositions of tools in the form of individual retrievers. This shows how we can connect different layers of abstraction in order to provide our LLM with a means of getting the data that it needs to answer a particular query.

## Summary
We created an LLM agent that uses information about the Barbie and Oppenheimer movies and creates a chat interface where we can ask the LLM questions and have it ground its answers in the data that we've prepared. This goes a long way towards making sure that our LLM, which naturally already is good at conversation and summarization, now has the data and the tools that it needs to ground its answers in truth and to make it that much more powerful. The related notebook is [here](https://colab.research.google.com/drive/1q6NZYUOcplDcAJFJAXMtFh-my73wOzjg) and the master solution from the original notebook can be found [here](https://github.com/AI-Maker-Space/LLM-Ops-Cohort-1/blob/main/Week%201/Thursday/Agent%20Powered%20Barbenheimer%20Application-Solution.ipynb).
