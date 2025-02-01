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

Using these settings, I've been getting the following performance (see testing script [here](https://github.com/METResearchGroup/bluesky-research/blob/main/lib/db/tests/queue_load_testing.py), courtesy of Claude and Cursor):

.

It ended up taking my test script more time to generate dummy records than it did to write them to the database.

There's still the con of SQLite being single-writer. I've gotten around this by giving every service its own SQLite instance. This way, they can each scale separately. SQLite is pretty performant at scale across a variety of dimensions (see [this](https://www.sqlite.org/limits.html) link from the SQLite docs). Plus, SQLite is simple to implement, comes out-of-the-box with Python, requires no setup or server, and is easy to manage. I re-implemented some logic of the data pipelines to work around the single-writer constraint. But also, is it really a constraint? Or can I just simplify my problem to not make it harder than it needs to be? I also clear the DB whenever I get to writing the records to a more permanent storage (using .parquet and DuckDB, which is also a surprisingly great combination as well in its own right).

Overall, I'm pretty impressed by the scalability of SQLite and how flexible it can be used. Can't believe that this is something that comes out-of-the-box with very little if any setup. I think people sleep on tools like this in order to reach for the "latest and greatest" tools. Just because it's "something used at Facebook" doesn't mean it's good for your app with 20 users. I think there's a value in learning how to both build things from scratch and also really appreciate when different tools are right for the job. No need to reach for a self-hosted Postgres database or a Kubernetes cluster if you've not built anything of scale yet.

I'm curious to see how my thoughts on this will change over time. The constraint of working on a high-performance bare-metal server has forced some creative engineering solutions, and I want to see how much I can push it before reaching to something like AWS.
