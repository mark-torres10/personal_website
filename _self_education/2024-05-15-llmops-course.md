---
layout: single
title:  "LLMOps self-education, Day 1"
date:   2024-05-15 04:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /self_education/llm-course-day-1
---

# LLMOps self-education, Day 1
I'm working on improving my RAG-building skills. As a part of that, I'm taking courses online. The course I'm currently taking is from [AI Makerspace](https://github.com/AI-Maker-Space), as part of their [LLMOps](https://github.com/AI-Maker-Space/LLM-Ops-Cohort-1) curriculum.

## Session 1: Course Intro | Building Your First RAG App with LangChain, Chainlit, and Hugging Face
I'm starting by following through the content in the first [video](https://www.youtube.com/watch?v=d1Oj5vrTWC4), which is a broad overview of what is being built. The related project notebook is [here](https://github.com/AI-Maker-Space/LLM-Ops-Cohort-1/tree/main/Week%201/Tuesday), and this code is from that notebook (so, not my own) though I'll be creating my own version of it and adding notes as well.

I'm forking the Google Colab notebook to my own Google Drive [here](https://colab.research.google.com/drive/1lCHC-W3DoO4_Nxr2165QpvvcEkpGq8Mc), but I'll be talking through my logic and notes in this post.

### Setup
This notebook goes over creating a RAG (or, as they call it "RAQA" (retrieval-augmented QA)) system that answers questions about the Barbie movie. First we start with installing dependencies

```python
!pip install -q -U openai langchain
```

And then setting up the OpenAI key
```python
import os
import getpass

os.environ["OPENAI_API_KEY"] = getpass.getpass("Open AI API Key:")
```

### Using web scraping to fetch the data
Now we'll use Selenium to fetch the reviews and the ratings:

```python
import numpy as np
import pandas as pd
from scrapy.selector import Selector
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
import time
from tqdm import tqdm
import warnings
warnings.filterwarnings("ignore")

chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')
driver = webdriver.Chrome(options=chrome_options)

url = "https://www.imdb.com/title/tt1517268/reviews/?ref_=tt_ov_rt"
driver.get(url)

sel = Selector(text = driver.page_source)
review_counts = sel.css('.lister .header span::text').extract_first().replace(',','').split(' ')[0]
more_review_pages = int(int(review_counts)/25)

for i in tqdm(range(more_review_pages)):
    try:
        css_selector = 'load-more-trigger'
        driver.find_element(By.ID, css_selector).click()
    except:
        pass
```

Now we format the reviews and export as a .csv:

```python
rating_list = []
review_date_list = []
review_title_list = []
author_list = []
review_list = []
review_url_list = []
error_url_list = []
error_msg_list = []
reviews = driver.find_elements(By.CSS_SELECTOR, 'div.review-container')

for d in tqdm(reviews):
    try:
        sel2 = Selector(text = d.get_attribute('innerHTML'))
        try:
            rating = sel2.css('.rating-other-user-rating span::text').extract_first()
        except:
            rating = np.NaN
        try:
            review = sel2.css('.text.show-more__control::text').extract_first()
        except:
            review = np.NaN
        try:
            review_date = sel2.css('.review-date::text').extract_first()
        except:
            review_date = np.NaN
        try:
            author = sel2.css('.display-name-link a::text').extract_first()
        except:
            author = np.NaN
        try:
            review_title = sel2.css('a.title::text').extract_first()
        except:
            review_title = np.NaN
        try:
            review_url = sel2.css('a.title::attr(href)').extract_first()
        except:
            review_url = np.NaN
        rating_list.append(rating)
        review_date_list.append(review_date)
        review_title_list.append(review_title)
        author_list.append(author)
        review_list.append(review)
        review_url_list.append(review_url)
    except Exception as e:
        error_url_list.append(url)
        error_msg_list.append(e)

res = {
    'Review_Date':review_date_list,
    'Author':author_list,
    'Rating':rating_list,
    'Review_Title':review_title_list,
    'Review':review_list,
    'Review_Url':review_url
}
review_df = pd.DataFrame(res)

review_df.to_csv("reviews.csv")
```

Here's what our data looks like:
![Reviews of Barbie movie](/assets/images/2024-05-15-llmops-course/reviews_df.png)

### Loading the data
We can use Langchain's [CSVLoader](https://python.langchain.com/v0.1/docs/integrations/document_loaders/csv/) to load the data as a series of documents.

```python
from langchain.document_loaders.csv_loader import CSVLoader

loader = CSVLoader(
    file_path="reviews.csv",
    source_column="Review"
)

data = loader.load()

print(data)
```

Now we have to split our documents into chunks. We'll use the [RecursiveCharacterTextSplitter](https://python.langchain.com/v0.1/docs/modules/data_connection/document_transformers/recursive_text_splitter/), which works by splitting on delimiting characters such as "\n" and " " recursively until we hit the required chunk size. This has the effect of creating chunks that have the highest chance of keeping together paragraphs, sentences, and then individual words together.

For example, if we have a document such as:

```plaintext
This is a sentence.
This is another sentence
```
We can set a chunk size and the character splitter will ideally give us the following two chunks:
```plaintext
1. This is a sentence.
2. This is another sentence.
```

We can set up the text splitter and apply it to our dataset:

```python
from langchain.text_splitter import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size = 1000, # the character length of the chunk
    chunk_overlap = 100, # the character length of the overlap between chunks
    length_function = len, # the length function
)
```

We can use this to transform the documents:
```python
documents: list[Document] = text_splitter.transform_documents(data)
```

This'll give us a list of Langchain `Document` objects. If we look at what one of these looks like, we'll see that it's just the regular text.

```python
documents[0]
```

```plaintext
Document(page_content=': 0\nReview_Date: 22 July 2023\nAuthor: Natcat87\nRating: 6\nReview_Title: Too heavy handed\nReview: As a woman that grew up with Barbie, I was very excited for this movie. I was curious to see how they would evolve the "stereotypical Barbie" into something more. But the messaging in this movie was so heavy handed that it completely lost the plot. I consider myself a proponent of gender equality, and this ain\'t the way to get it.\nReview_Url: /review/rw9200852/?ref_=tt_urv', metadata={'source': 'As a woman that grew up with Barbie, I was very excited for this movie. I was curious to see how they would evolve the "stereotypical Barbie" into something more. But the messaging in this movie was so heavy handed that it completely lost the plot. I consider myself a proponent of gender equality, and this ain\'t the way to get it.', 'row': 0})
```

However, we can see that for longer texts, that the splitter will actually split the text. Let's take the longest review in the dataset to see how the splitter would work:
```python
texts = review_df["Review"]
max_text = max(texts, key=len)
len(max_text) # 2383
```

If we run the splitter on this, we get the following:
```python
max_text_docs = text_splitter.split_text(max_text)
len(max_text_docs)
```

```plaintext
>>> max_text_docs[0]
Let me start by saying that I'm by no means a conservative or traditional woman. I believe that inequality for women still is rampant even in first world countries and that we go through many problems. But damn...this movie is so corporate and shallow. It's clearly ordered and tailored by executives at Mattel to rebrand and sell more dolls. The movie starts well and has the potential of becoming something very good, but then it just goes downhill: the premise of the owner's feeling being projected onto the doll was going to lead to a very nice build up about a person's life journey and then Barbie and Ken go to the human world and cue in the speeches and preaching. This movie suffers from a bad case of showing and not telling, they keep repeating the same dialogues: patriarchy bad patriarchy bad patriarchy bad sexism sexism sexism, not even one time but MULTIPLE TIMES. These characters go into random rants like its a ted talk! The chemistry between the mother and daughter is absolutely
>>> max_text_docs[1]
into random rants like its a ted talk! The chemistry between the mother and daughter is absolutely nonexistent and their relationship is so cringe, You cant root for them or feel their feelings at all. Barbie should have understood how Kens had been feeling in the Barbieland the whole time which is a mimic of how many women are still treated in the society: just existing for the gaze of a woman (man). Yet what do they do? They manipulate them (with a super nonsense plan) and force them back to their original position while she gives him a half assed apology that could have been done better and end up to a more accurate point about equality. (I guess this is again meta and a reflection of how women are pitted against each other while people in charge benefit from that but its so badly done that it feels meh.)
```

```python
print(len(max_text_docs[0])) # 1000
print(len(max_text_docs[1])) # 819
print(len(max_text_docs[2])) # 662
```

We see that the splitter does split longer texts.


### Creating our index
Now that we have our documents, we can create an index using these documents. An index lets us quickly access our documents as embeddings so that when we have queries, we know immediately which documents are most similar to the query.

#### Selecting a vector store
For our use case, we'll use [FAISS](https://ai.meta.com/tools/faiss/#:~:text=FAISS%20(Facebook%20AI%20Similarity%20Search,more%20scalable%20similarity%20search%20functions.)), a vector store out of Facebook.

For our embeddings, we'll be using [OpenAI's embeddings](https://api.python.langchain.com/en/latest/embeddings/langchain_community.embeddings.openai.OpenAIEmbeddings.html). We'll also be using [caching](https://python.langchain.com/v0.1/docs/modules/data_connection/text_embedding/caching_embeddings/) to make sure that we don't have to recompute our embeddings (so we don't have to pay more money for duplicate embeddings).

```python
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.embeddings import CacheBackedEmbeddings
from langchain.vectorstores import FAISS
from langchain.storage import LocalFileStore

core_embeddings_model = OpenAIEmbeddings()

store = LocalFileStore("./cache/")

embedder = CacheBackedEmbeddings.from_bytes_store(
    core_embeddings_model,
    store,
    namespace=core_embeddings_model.model
)

vector_store = FAISS.from_documents(documents, embedder)
```

We can now see if the vector store works by embedding a query and seeing what reviews are near it.

```python
query = "What does Barbie do in this movie?"
embedding_vector = core_embeddings_model.embed_query(query)
docs = vector_store.similarity_search_by_vector(embedding_vector, k = 4)

for page in docs:
  print('Review:')
  print(page.page_content[:100])
  print('-' * 10)
```

```plaintext
Review:
Review: I personally expected the movie to be fun and adventurous. Correct me if I'm wrong, but isn't that what it is supposed to be? It's literally a
----------
Review:
: 27
Review_Date: 10 August 2023
Author: kadiarula
Rating: 10
Review_Title: Loved it!
Review: Barbie comes to life and is surprised by all the possibi
----------
Review:
: 29
Review_Date: 21 November 2023
Author: sandy777
Rating: 4
Review_Title: Some humorous moments, but the story fell flat
Review: Barbie, a movie abo
----------
Review:
Review: Let me start by saying that I'm by no means a conservative or traditional woman. I believe that inequality for women still is rampant even in 
----------
```

Using the `CacheBackedEmbeddings` is also helpful for us, not only in saving costs, but also giving us results more quickly:
```python
%%timeit -n 1 -r 1
query = "I really wanted to enjoy this and I know that I am not the target audience but there were massive plot holes and no real flow."
embedding_vector = core_embeddings_model.embed_query(query)
docs = vector_store.similarity_search_by_vector(embedding_vector, k = 4)
```
```plaintext
223 ms ± 0 ns per loop (mean ± std. dev. of 1 run, 1 loop each)
```

Now, we can repeat the query:
```python
%%timeit
query = "I really wanted to enjoy this and I know that I am not the target audience but there were massive plot holes and no real flow."
embedding_vector = core_embeddings_model.embed_query(query)
docs = vector_store.similarity_search_by_vector(embedding_vector, k = 4)
```
```plaintext
124 ms ± 11.2 ms per loop (mean ± std. dev. of 7 runs, 10 loops each)
```

### Creating a chain
Now that we have the data and our index, we can create a chain. We'll create a retrieval chain that will let us ask questions on top of our index.

First, we'll set up our [OpenAI chat model](https://python.langchain.com/v0.1/docs/modules/model_io/chat/). We'll use GPT3.5 for this.

```python
from langchain.llms.openai import OpenAIChat

llm = OpenAIChat()
```

To use our vector store in the chain, we'll create a [retriever](https://python.langchain.com/v0.1/docs/modules/data_connection/retrievers/) out of it. Retrievers are built on top of a regular vector store and let us return documents. Retrievers don't have to be used with vector stores, but we'll be using it with a vector store for our case so we can return documents.
```python
retriever = vector_store.as_retriever()
```

Now we create our retrieval chain:

```python
from langchain.chains import RetrievalQA
from langchain.callbacks import StdOutCallbackHandler

handler = StdOutCallbackHandler()

qa_with_sources_chain = RetrievalQA.from_chain_type(
    llm=llm,
    retriever=retriever,
    callbacks=[handler],
    return_source_documents=True
)
```

We can then run this to see how the retrieval looks. It does the following steps:
1. Convert the query to an embedding
2. Fetch the documents from the index that are most related to the query
3. Pass in the query and the documents to the OpenAI chat.
4. Get the answer to the question.

```python
qa_with_sources_chain({"query" : "How was Will Ferrell in this movie?"})
```

```plaintext
> Entering new RetrievalQA chain...

> Finished chain.
{'query': 'How was Will Ferrell in this movie?',
 'result': "There are mixed opinions about Will Ferrell's performance in the movie. Some reviewers felt that his character was unnecessary and his presence detracted from the film, while others felt that his talents were underutilized.",
 'source_documents': [Document(page_content=": 76\nReview_Date: 23 July 2023\nAuthor: a-hilton\nRating: 10\nReview_Title: Had me smiling all the way through\nReview: Okay maybe it was a 9.5 because of two flaws: First was the Will Ferrell character and his board that made their point but then became superfluos. Second was that it is definitely not a kids' movie (although maybe they would see things that I didn't - I mean to be fair, the few kids in the theatre were well behaved so perhaps the movie got their full attention as well).\nReview_Url: /review/rw9199947/?ref_=tt_urv", metadata={'source': '/review/rw9199947/?ref_=tt_urv', 'row': 76}),
  Document(page_content=': 85\nReview_Date: 23 July 2023\nAuthor: hyllus-01262\nRating: 6\nReview_Title: Overhyped movie, had its moments though\nReview: The first half was pretty enjoyable, fun, light, but it took itself too seriously by the second half. No longer allowing the talented cast, especially Gosling, to shine and make us laugh. It felt like the talents of Will Ferrell and Michael Cera were also somewhat underutilized. Interesting concept, had potential, but later in the movie, it definitely started to fall flat for me.\nReview_Url: /review/rw9199947/?ref_=tt_urv', metadata={'source': '/review/rw9199947/?ref_=tt_urv', 'row': 85}),
  Document(page_content=": 61\nReview_Date: 23 July 2023\nAuthor: agjbull\nRating: 6\nReview_Title: Just a little empty\nReview: I really wanted to enjoy this and I know that I am not the target audience but there were massive plot holes and no real flow. The film was very disjointed. Ryan Gosling as good as he is seemed to old to play Ken and Will Ferrell ruined every scene he was in. I just didn't get it, it seemed hollow artificial and hackneyed. A waste of some great talent. It was predictable without being reassuring and trying so hard to be woke in the most superficial way in that but trying to tick so many boxes it actually ticked none. Margo Robbie looks beautiful throughout, the costumes and the sets were amazing but the story was way too weak and didn't make much sense at all.\nReview_Url: /review/rw9199947/?ref_=tt_urv", metadata={'source': '/review/rw9199947/?ref_=tt_urv', 'row': 61}),
  Document(page_content="Review: The movie was very funny and really enjoyable to laugh at with the full theatre. However, the messages of the movie were the problem. I was never really sure what I was supposed to take away, there was nothing about finding equality or love it was all about how every man cat falls every woman or women can't be anything. It was really silly because there was no accurate reflection of America at any single point except for Barbie getting called a Fascist for no reason by a 14 year old. I enjoyed how they called out women for hating women and how they really tried to preach empowerment and the ability to be anything, but at the same time there was so much resentment and they ended the movie by reinstating hate. The majority of the movie was hating men as much as possible. That's just whatever because what really matters is the story. Well it fell short on that mark and it was really disappointing. The pacing was horrible, the villain won and was pretty irrelevant in the long run,", metadata={'source': '/review/rw9199947/?ref_=tt_urv', 'row': 17})]}
```

### Conclusion
This is a simple RAG application that (1) scrapes and processes text, (2) creates an index out of it, and (3) creates a retrieval chain that can be used so that we can ask questions on top of the data.
