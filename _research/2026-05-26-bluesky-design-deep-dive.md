---
layout: single
title: "How I built the infrastructure for a large-scale social media field experiment during the 2024 US election"
date: 2026-05-26 02:00:00 +0800
classes: wide
toc: true
mermaid: true
categories:
- research
- all_posts
permalink: /research/2026-05-26-bluesky-design-deep-dive
---

## What we built

## Why did we need to build this in the first place?

## How did Bluesky make this possible?

## Architecture at a glance

```mermaid
flowchart LR
  B[Bluesky firehose and APIs] --> S[Sync pipelines]
  S --> P[Preprocessing]
  P --> C[Fan-out to integrations]
  C --> M[ML classifiers]
  C --> SP[Superposter calculation]
  C --> V[Offline FAISS embeddings]
  M --> U[Unify integrations]
  SP --> U
  V --> U
  U --> R[Generate feeds]
  R --> A[Feed API]
  A --> U2[Bluesky users]
```

At a high-level, the app is designed around the following components:

1. Data ingestion: connecting to the real-time Bluesky data stream and getting live records.
2. Data processing: taking the ingested records, filtering invalid records, and enqueueing to downstream services.
3. Integrations fan-out: running the enrichment integrations (ML classifiers, superposter calculation, vector embedding generation) and labeling the latest batches of posts.
4. Consolidate/unify enriched posts: aggregating ...
5. Generating feeds: ...
6. Serving feeds to users: ...

## Deeper dive into the architecture

### Data ingestion

For the data ingestion module, we connected to the Bluesky real-time data stream (firehose). Bluesky publishes events that happen on the platform ("user X likes post Y", "user Z made a new post") in real time. In a move unprecedented, Bluesky chose to make this data publicly available.

#### What alternatives did we consider?

We considered, but eventually moved away from, a few alternatives.

##### Bluesky API

Bluesky maintains a public API. We used this for small-scale tasks (e.g., getting the current profile for a user). The endpoints from Bluesky are much more fully hydrated (e.g., the GET /posts endpoint has post text, engagement data, etc) but the throughput was too low for our scale (I forget the exact rate limits, but I think the total data we could have collected was on the level of low thousands per day at most, when we were collecting easily 1,000x that for our pipelines). In addition, if we manually polled users we wouldn't be able to know when to scrape data for a user. Most users weren't particularly active, so we could likely have pinged the API for their data every few days. However, there were a few power users who posted and engaged with content every single day. In addition, we needed to know what content users liked, and the like data isn't readily available in the API.

##### Web scraping

I tried a few experiments with Selenium to develop web scrapers that crawled a user's feed to get their posts and liked posts. This did not work particularly well, as it both failed to capture content from power users and also was prone to false negatives. I retried this when AI computer agents were first introduced. Those experiments yielded perhaps 30 posts in 30 minutes of waiting for the AI agent to scroll the page. They've very likely gotten much better, but I anticipate that doing it at scale would've gotten the AI agent session throttled by Bluesky (plus racked up quite a tab for myself).

##### PDS backfills

The Bluesky data stream is a live firehose of real-time data. However, if one isn't able to gather data in real-time, Bluesky provides ways to [traverse the PDSes](https://github.com/bluesky-social/pds) which store the data publicly.

I used this approach to backfill records that I did not have. For example, we tracked the like records from the real-time firehose. However, these like records were very sparse, only having the like ID and the user who liked the post. To analyze anything about the posts that were liked, I had to figure out how to backfill records by traversing the PDSes.

The way to do this wasn't documented at all online when I was building it out, and I had to piece it together from checking the Discord chats and asking questions to the Bluesky devs. Frankly I'm unsure if I truly understand how it works, as I still don't 100% understand Bluesky's underlying architecture, but [I implemented a solution to backfill records](https://github.com/METResearchGroup/bluesky-research/tree/main/services/backfill/pds_backfills).

This approach worked OK but had low throughput and often disconnected. It did not capture data at the scale that we would've wanted for the study.

##### Bluesky Jetstream

Bluesky recently introduced [Jetstream](https://github.com/bluesky-social/jetstream), a streamlined way to backfill records. This did not exist during the study, though we've experimented with it in the lab for [other use cases](https://github.com/METResearchGroup/lab_data_integrations_interface/pull/5). This is much more feasible to do than the manual backfill that I was doing before and has a much higher QPS. Backfills are now no longer as gruesome and I actually would likely couple this with the firehose to have a hybrid ingestion architecture with both real-time ingestion and batch backfills.

#### Tradeoffs from using the data stream

The choice to use the data stream came with a few tradeoffs.

##### App behavior coupled to spikes/crashes from the Bluesky side

The expected load on our system was closely coupled to the load from the data stream. When there was heavy traffic on Bluesky, our app experienced heavy traffic as well. When Bluesky crashed

We learned to manage this in the following ways:

- Decouple data persistence from live ingestion: during times of high throughput, the ingestion job was rate-limited by write throughput. I resolved this by splitting ingestion into 2 parallel jobs: a job that writes the data stream in-memory to a `temp/` output as .json files and a job that takes those records and writes them to permanent .parquet storage. This also had the additional benefit of reducing the number of .parquet files we had to write (thereby better leveraging parquet's columnar format).
- Decouple downstream services from data ingestion: ingested records were both written to permanent parquet storage as well as enqueued for downstream services. Rather than triggering downstream services after the data ingestion pipeline or creating an event-driven architecture, I chose to instead explicitly decouple the runtimes for the data ingestion and data processing services. I formalized this through the use of separate orchestration DAGs for both layers, meaning that the data processing and downstream jobs could just run on whatever data, if any, were enqueued from the upstream data ingestion layer.

##### Have to know upfront what data you want

Using the data stream requires that you know upfront which records you want. If I had to collect a new data type, I would have to restart the connection to the data stream and manually add in a filter for that data type, and then if we wanted records prior to today, I would have to backfill those manually.

##### Inevitable downtime periods

I needed long-lived persistent servers, but this wasn't available on HPC. I ran the longest jobs I could, 7-day persistent jobs, and then manually restarted them. I set up the jobs to email me whenever they finished, and I also created a small cronjob that polled the job scheduler to see if the job was finished or not, and automatically submitted a new job once one ended.

Inevitably, there would be a few minutes of downtime between when a data ingestion job finished and when the job scheduler would enqueue and run the request for the next data ingestion job. This meant that for a few minutes, we may have missed some activity. However, this likely had a negligible impact, as (1) it was a few minutes in a week (<10 minutes) and (2) I scheduled this to happen in the afternoon Asia time, which would mean late evening US time, and I doubted anyone would have meaningful social media activity at 3am (and if they do, they need to sleep!). Even with this downtime, we still had three nines of availability.

### Preprocessing

### Enrichment

#### ML classifiers

We used a variety of ML classifiers for the project ...

### Recommendation algorithms

### Serving

### Observability and Ops

### Infra and deployment

## Tradeoffs we made

## Where I'd likely change it up now

### DevOps from the start
