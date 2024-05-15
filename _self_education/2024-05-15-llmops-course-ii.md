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

# Session II: Questioning Barbie and Oppenheimer Through the Use of Agents (and deploy as a Gradio app)
I'm working on improving my RAG-building skills. As a part of that, I'm taking courses online. The course I'm currently taking is from [AI Makerspace](https://github.com/AI-Maker-Space), as part of their [LLMOps](https://github.com/AI-Maker-Space/LLM-Ops-Cohort-1) curriculum.

The related project notebook is [here](https://github.com/AI-Maker-Space/LLM-Ops-Cohort-1/blob/main/Week%201/Thursday/Agent%20Powered%20Barbenheimer%20Application%20(Assignment%20Version).ipynb), and my own version of that notebook [here](https://colab.research.google.com/drive/1q6NZYUOcplDcAJFJAXMtFh-my73wOzjg).

I'll also follow along with [this](https://github.com/google/generative-ai-docs/blob/main/examples/gemini/python/langchain/Gemini_LangChain_QA_Chroma_WebLoad.ipynb) Langchain notebook that uses [Gemini](https://ai.google.dev/models/gemini) for our embeddings and [Chroma](https://docs.trychroma.com/) for our vector database, and I'll use those instead of OpenAI and FAISS like in the [last](https://markptorres.com/self_education/llm-course-part-i) portion.

## Setup

First, we install the correct packages and pass in our API key.

```python
!pip install -q -U langchain langchain-google-genai chromadb
```

```python
import os
import getpass

os.environ['GOOGLE_API_KEY'] = getpass.getpass('Gemini API Key:')
```

## Contruct a Barbie retriever
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

#### Using embeddings for the Barbie reviews
For the Barbie reviews, we'll create our embeddings using a dense retriever.
```python

```

## Construct an Oppenheimer retriever

We'll repurpose a lot of the same logic from the Barbie retriever to create an Oppenheimer retriever.

## Combine the two and allow users to query both resources from a single input through the use of Agents

## Deploy as a Gradio app
