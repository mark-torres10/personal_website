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