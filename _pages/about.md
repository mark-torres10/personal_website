---
layout: single
title: "About Me"
collection: about
permalink: /about/
classes: wide
author_profile: true
---

# Hi!
My name is Mark Torres. I'm currently a researcher and ML engineer at the [Kellogg School of Management](https://www.kellogg.northwestern.edu/) at [Northwestern University](https://www.northwestern.edu/), studying how ML algorithms affect people's perceptions of politics on social media. I develop end-to-end LLM-powered apps and AI agents in order to learn more about people's online behaviors. In a past life, I worked for a few years as a data scientist for [SetSail](https://www.setsail.co/), a Series A startup acquired by ZoomInfo. Before that, I also worked as a data scientist and researcher at Yale. I am also completing my Master's in Computer Science at UT Austin, after completing my Bachelor's in Statistics at Yale.

## Current: Researcher and ML engineer @Northwestern (2023-present)

Highlights include:
- **Served as lead engineer.** Designed and developed system architecture, pipelines, and database models. Led system design and technical requirement meetings with stakeholders. Code [here](https://github.com/METResearchGroup/bluesky-research).
- **Developed LLM-powered AI agents to perform tasks related to reporting social and political characteristics of social media content**. Used tools such as langchain and llamaindex for agent development, Supabase and MongoDB for vector database storage, [dspy](https://github.com/stanfordnlp/dspy) for optimizing LLM prompting, and both open-sourced (Llama3, Mixtral) and closed-source (Gemini, GPT3.5/GPT4) for LLM capabilities. Scaled LLM inference to classify >1M posts per day. Created model-agnostic tooling that can use and swap across various model backends.
- **Designed and developed a state-of-the-art classification model pipeline for using context when classifying political ideology of social media content**. The pipeline takes in social media posts as well as their context (who posted it, any links or images they've added, the broader conversation thread) to classify (1) whether a post contains sociopolitical (regarding social issues or politics) content and (2) if so, the political ideology of the content (left, center, right, neither). Used Llama3-70b model for inference. Achieved an accuracy of 0.906, precision of 0.8819, and recall of 0.9845 on the classification task, greatly outperforming existing fine-tuned models.
- **Designed and created a RAG pipeline for adding current events knowledge to LLMs**. The pipeline ingests daily news articles, stores them as vector databases via FAISS, and then automatically surfaces them to LLM prompts as additional context for classification tasks to provide up-to-date current events knowledge.
- **Productionized scalable inference using Google's Perspective API**. Developed a batch inference pipeline to classify the toxicity and constructiveness attributes of social media content across >1M daily posts per day.
- **Designed and developed end-to-end ETL pipelines to ingest and process >1M posts per day**. Synced real-time social media content via firehose API, ran batch filtering and preprocessing, and made data available for downstream services. Stored data in SQLite and MongoDB databases. Enforced data quality via tools such as pydantic.
- **Developed an end-to-end app for studying online moral outrage on Reddit**. The app gathers comments from online subreddits, uses a pretrained model for classifying moral outrage, messages users with a survey if they posted content with moral outrage, and collects their responses. Scraped over 2M posts on Reddit and messaged >10,000 users on Reddit. Code [here](https://github.com/mark-torres10/redditResearch).

- **Created demo apps via Streamlit in order to demonstrate functionalities to stakeholders and get project feedback and shape roadmap planning**.
- **Developed telemetry and logging tooling across database**. Created automated data backup and recovery tooling.
- **Performed data analysis**. Calculated experimental results of pilot studies.
- **Read lots (and lots) of papers, across both ML and the social sciences**.

Some things on the near-term roadmap include:
- Introducing automated LLM evaluation services: the task of LLM evaluation is notoriously domain-specific and nebulous (even more so than regular ML already is).
- Migrating from batch to micro-batch or streaming ingestion, using tooling such as Apache Kafka.
- Improving logging and visibility.
- Migrating to a more production-ready orchestration system, such as Airflow or Mage (cron jobs are working OK for now).
- Developing a browser extension to more closely track user session data on the [Bluesky](https://bsky.app/) app.

## Previous: Data Scientist @SetSail (2021-2023)

Highlights include:
- **Designed and productionized ML algorithms for entity extraction and sentiment analysis**.
- **Developed scalable data science infrastructure and implemented a series of microservices.**
- **Redesigned data ETL, especially at ingestion, to use Spark and various AWS tooling (e.g., S3, EMR, Athena).** Migrated away from JSON and towards parquet. Reworked orchestration pipelines using AWS step functions (although in hindsight, maybe Airflow would've been worth the investment).
- **Designed and developed product features, especially those build on top of the company's data lake.** Lots of SQL. Lots of AWS Athena. Developed pipelines, from ingestion to transformation to feature generation, of custom data integrations.
- **Cleaned up and standardized core Python and data science tooling and infrastructure.** Promoted DevOps best practices.
- **Deployed core data science infrastructure using Docker and Terraform.** .
- **Participated in on-call rotation, on both the data science and data engineering teams**.
- **Created an in-house data annotation platform using Prodigy and scikit-learn**.
- Wrote a lot of code. Sometimes broke production. Learned how to revert. 

I learned a lot in this startup, where being a data scientist means you do everything that vaguely involves data. I was doing a lot of data engineering, and I credit this job for greatly improving my fundamental engineering skills, teaching me great fundamentals, as well as for being a place where I was able to work with great coworkers and mentors. This experience gave me:
- Confidence that I have the core engineering capability to build anything that needs to be built.
- A startup-mindset for knowing how to make tradeoffs in what needs to be done today versus what can be done tomorrow.
- The resourcefulness to be able to work with whatever compute and resources I do (or don't) have.
- A tendency to ship, ship, and ship some more. Execution, iteration, and various MVPs > weeks and weeks of pure planning.

## Previous: Data Scientist @Yale (2020-2021)

Highlights include:
- Built an ML algorithm to predict the probability that a person contracts COVID, given demographic, network-based, and temporal information, as part of a group called [Hunala](https://news.yale.edu/2020/06/05/yale-app-hunala-aims-be-waze-coronavirus).
- Developed ML algorithms for predicting single-nucleotide polymorphisms (SNPs) using sequence models, as a part of [Gerstein Lab @Yale](https://gersteinlab.org/).

##  Published a few papers here and there:
My [Google Scholar Profile](https://scholar.google.com/citations?user=d7CzCRMAAAAJ&hl=en).
- Brady, W.J., McLoughlin, K.L., Torres, M.P. et al. Overperception of moral outrage in online social networks inflates beliefs about intergroup hostility. Nat Hum Behav 7, 917–927 (2023). https://doi.org/10.1038/s41562-023-01582-0
- Iwamoto, S.K., Alexander, M., Torres, M. et al. Mindfulness Meditation Activates Altruism. Sci Rep 10, 6511 (2020). https://doi.org/10.1038/s41598-020-62652-1

## Education:
- (Ongoing) MS, Computer Science, @UT Austin, expected completion Winter 2025
- (Completed) BS, Statistics, @Yale, 2016-2020

## Et cetera
I am a digital nomad these days, and am likely in a new country every 1-2 months. I'm based out of Chicago/Florida/Manila these days, depending on the current travel schedule.
