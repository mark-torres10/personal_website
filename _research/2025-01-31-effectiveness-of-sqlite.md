---
layout: single
title:  "The surprising effectiveness of SQLite for non-persistent use cases."
date:   2025-01-31 05:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/2025-01-31-effectiveness-of-sqlite
---

I've been using SQLite as a buffer for some microservices that I'm building out. Normally I would've thought of using something like SQS or Kafka for something like this, but I'm trying to make this as low-cost as possible and I have to work within the constraints of bare-metal servers instead of using cloud services. Sometimes it feels like coding without an IDE, since I've had to build out so much from scratch, but it's also taught me a lot about how to create things from the ground up and not necessarily jump to or rely on off-the-shelf tools.

I had initially tried to manage my own caching and buffering system, but I kept encountering problems, especially around multiple processes concurrently writing as well as transaction integrity. As it turns out, SQLite has a good solution for this: single-writer ACID transactions. Why introduce the need for concurrency when the application doesn't need it?

I've been building out this system for using SQLite as these buffers, and have been pleasantly surprised by how well it's working. I've been using the following batching scheme:

- Grouping 1,000 records at a time into a single row
- Writing 25 rows at a time to the database (could possibly tune this, though I've been happy with it so far).
- Using WAL mode for the database.

For the dummy records, these only have fields `{"uri": <uri>, "text": <text>}` . I JSON-dump these into payload strings and create a composite record that looks something like:

```python
{
    "payload": "{'uri': <uri>, 'text': <text>}",
    "timestamp": "<timestamp>",
    "metadata": "{'service': <service>}"
}
```

Using these settings, I've been getting the following performance (see testing script [here](https://github.com/METResearchGroup/bluesky-research/blob/main/lib/db/tests/queue_load_testing.py), courtesy of Claude and Cursor).

Here are the results from writing 1,000 records per row, and 25 rows per transaction.

| Records | Seconds | Records/Second | Final DB size (MB) |
|---------|---------|----------------|-------------------|
| 1,000 | 0 | 645,079 | 0.13 |
| 10,000 | 0.1 | 1,213,910 | 1.26 |
| 100,000 | 0.08 | 1,210,124 | 12.51 |
| 1,000,000 | 0.96 | 1,015,605 | 125 |
| 10,000,000 | 10.01 | 998,764 | 1,250 (1.25GB) |
| 50,000,000 | 62.39 | 801,376 | 6,250 (6.25GB) |
| 100,000,000 | 422.72 | 236,560 | 12,500.98 (12.5GB) |

Surprisingly, writing 100 rows at a time seemed to be faster for the 50M run but was faster for the rest? It seems like there's probably a sweet spot for how many records to write as part of the same transaction:

| Records | Seconds | Records/Second | Final DB size (MB) |
|---------|---------|----------------|-------------------|
| 1,000 | 0 | 488505 | 0.13 |
| 10,000 | 0.01 | 1,214,473 | 1.26 |
| 100,000 | 0.08 | 1,224,832 | 12.51 |
| 1,000,000 | 0.71 | 1,403,378 | 125.02 |
| 10,000,000 | 7.68 | 1,302,649 | 1,250 (1.25GB) |
| 50,000,000 | 92.54 | 540,315 | 6,250 (6.25GB) |
| 100,000,000 | N/A | N/A | N/A |

I ran the job for 100M with 25 rows at a time locally and it worked fine (though my MacBook was humming weird at the end). I ran the job for 100M with 100 rows at a time on Slurm and, even after allocating 40GB of memory, it kept crashing with an OOM error. Oof.

Writing 10 rows at a time seems strictly worse than the previous two:

| Records | Seconds | Records/Second | Final DB size (MB) |
|---------|---------|----------------|-------------------|
| 1,000 | 0.13 | 7,643 | 0.13 |
| 10,000 | 0.12 | 81,117 | 1.26 |
| 100,000 | 3.75 | 26,675 | 12.51 |
| 1,000,000 | 20.18 | 49,555 | 125.02 |
| 10,000,000 | 204.40 | 48,923 | 1,250 (1.25GB) |
| 50,000,000 | N/A | N/A | N/A |
| 100,000,000 | N/A | N/A | N/A |

I’m unsure to what degree writing 25 rows at a time is different from writing 100 rows at a time. In practice, I likely will be writing at most 1M records at a time anyways, though it’s good to know how it would scale in the worst case.

Another observation was that it ended up taking my test script more time to generate dummy records than it did to write them to the database. This is something to consider as just having a large amount of records in memory is in itself a bottleneck, especially if you're not careful about memory management. I've been able to largely get around this by making my data pipelines operate in batches; this batch approach is also why creating a good buffering model is important, since I can detach the microservices from each other and separate out the data pipeline batching from the database writes.

There's still the con of SQLite being single-writer. I've gotten around this by giving every service its own SQLite instance. This way, they can each scale separately. SQLite is pretty performant at scale across a variety of dimensions (see [this](https://www.sqlite.org/limits.html) link from the SQLite docs). Plus, SQLite is simple to implement, comes out-of-the-box with Python, requires no setup or server, and is easy to manage. I re-implemented some logic of the data pipelines to work around the single-writer constraint. But also, is it really a constraint? Or can I just simplify my problem to not make it harder than it needs to be? I also clear the DB whenever I get to writing the records to a more permanent storage (using .parquet and DuckDB, which is also a surprisingly great combination as well in its own right).

Overall, I'm pretty impressed by the scalability of SQLite and how flexible it can be used. Can't believe that this is something that comes out-of-the-box with very little if any setup. I think people sleep on tools like this in order to reach for the "latest and greatest" tools. Just because it's "something used at Facebook" doesn't mean it's good for your app with 20 users. I think there's a value in learning how to both build things from scratch and also really appreciate when different tools are right for the job. No need to reach for a self-hosted Postgres database or a Kubernetes cluster if you've not built anything of scale yet.

I'm curious to see how my thoughts on this will change over time. The constraint of working on a high-performance bare-metal server has forced some creative engineering solutions, and I want to see how much I can push it before reaching to something like AWS.
