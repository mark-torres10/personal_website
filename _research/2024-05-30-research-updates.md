---
layout: single
title:  "Research updates, 2024-05-30"
date:   2024-05-29 04:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/research-updates-2024-05-30
---
# Research updates, 2024-05-30

Today, I worked on the following:

- Scrapped introduction of DB migration tooling.
- Fixed problems with preprocessing pipeline.
- Experimented with GPU inference on KLC
- Starting analysis of pilot data posts
- Improved basic logging and backing up logging.

## Scrapped the introduction of DB migration tooling.
Previously, I had problems with inserting preprocessed posts into a SQLite database because of integrity errors. Specifically, the "indexed_at" field, which I had changed to be nullable (firehose posts don't have an "indexed_at" field) wasn't nullable in the database. The most appropriate solution would be a database migration to add nullability to this field.

However, for this specific problem, I decided that I could just delete the database and try again. It had n~5,000 rows, and that data was data that I already had separately stored in MongoDB and could just reprocess. I deleted the database and rebuilt it to now have the correct schema (I don't recommend this in production or at scale, but when you're iterating and trying to ship fast like me, you have to do what it takes!).

It's very likely that I'll have to do a migration at some point soon though. I already see that the synctimestamp lookup is taking too long. I'll probably come back to that in a week or two, when I've synced millions more posts and doing a for-loop lookup is infeasible. There's a few ways that I could solve the problem, but the most straightforward is to have synctimestamp as a top-level field instead of within the "metadata" struct (especially since SQLite doesn't natively support structs). I am hoping that at that point I can (1) figure out how to host a NoSQL store like MongoDB on the compute cluster, and (2) that a NoSQL solution will run faster than a tabular database. If not that, I can do the database migration solution, which would just involve (1) adding the new column and (2) backfilling all posts and setting `synctimestamp = json.loads(metadata.synctimestamp)` (in Python, and I'll have to convert this command to either SQL-specific or ORM-specific commands).

## Fixed problems with preprocessing pipeline.
Now that the DB migration is off and I've deleted the database so that the data formatting problems are fixed, I can continue with fixing the preprocessing pipeline. The pipeline to sync the raw data is already working as a cron job, and now I need to get the preprocessing pipeline to work correctly as well.

```plaintext
(bluesky_research) [jya0297@quser31 preprocess_raw_data]$ python main.py
2024-05-30 00:33:51,695 INFO [_client.py]: HTTP Request: POST https://bsky.social/xrpc/com.atproto.server.createSession "HTTP/1.1 200 OK"
HTTP Request: POST https://bsky.social/xrpc/com.atproto.server.createSession "HTTP/1.1 200 OK"
2024-05-30 00:33:51,831 INFO [_client.py]: HTTP Request: GET https://bsky.social/xrpc/app.bsky.actor.getProfile?actor=mindtechnologylab.bsky.social "HTTP/1.1 200 OK"
HTTP Request: GET https://bsky.social/xrpc/app.bsky.actor.getProfile?actor=mindtechnologylab.bsky.social "HTTP/1.1 200 OK"
Warning : `load_model` does not return WordVectorModel or SupervisedModel any more, but a `FastText` object which is very similar.
2024-05-30 00:33:53,500 INFO [main.py]: Starting preprocessing pipeline.
2024-05-30 00:33:55,512 INFO [load_data.py]: Loading latest raw data.
> /projects/p32375/bluesky-research/services/preprocess_raw_data/load_data.py(32)load_firehose_posts()
-> transformed_posts: list[TransformedRecordWithAuthorModel] = [
(Pdb) continue
2024-05-30 00:37:07,497 INFO [helper.py]: Consolidated the formats of 286490 posts...
2024-05-30 00:37:14,829 INFO [helper.py]: Finished consolidating the formats of 286490 posts.
Execution time for consolidate_post_records: 0 minutes, 8 seconds
Memory usage for consolidate_post_records: 781.7109375 MB
2024-05-30 00:37:17,892 INFO [helper.py]: Consolidated the formats of 5875 posts...
2024-05-30 00:37:17,967 INFO [helper.py]: Finished consolidating the formats of 5875 posts.
Execution time for consolidate_post_records: 0 minutes, 1 seconds
Memory usage for consolidate_post_records: 1.734375 MB
Execution time for load_latest_raw_posts: 3 minutes, 25 seconds
Memory usage for load_latest_raw_posts: 3396.8984375 MB
2024-05-30 00:37:20,096 INFO [helper.py]: Loaded 292365 posts for filtering.
2024-05-30 00:37:21,103 INFO [filters.py]: Starting post filtering in preprocessing pipeline.
2024-05-30 00:37:22,392 INFO [filters.py]: Total posts for filtering: 292365
Execution time for classify_posts: 0 minutes, 2 seconds
Memory usage for classify_posts: 0.08203125 MB
Execution time for filter_posts_with_filter_func: 0 minutes, 4 seconds
Memory usage for filter_posts_with_filter_func: 0.09765625 MB
2024-05-30 00:37:27,846 INFO [filters.py]: After English filtering, number of posts that passed filter: 168740
2024-05-30 00:37:27,846 INFO [filters.py]: After English filtering, number of posts that failed filter: 123423
Execution time for classify_posts: 0 minutes, 1 seconds
Memory usage for classify_posts: 0.0 MB
Execution time for filter_posts_with_filter_func: 0 minutes, 3 seconds
Memory usage for filter_posts_with_filter_func: 0.0 MB
2024-05-30 00:37:32,317 INFO [filters.py]: Finished filtering with function: filter_posts_not_written_by_bot
2024-05-30 00:37:32,317 INFO [filters.py]: Posts passing filter: 168740. Posts failing filter: 0.
2024-05-30 00:37:32,317 INFO [filters.py]: Posts remaining after filtering with filter_posts_not_written_by_bot: 168941
Execution time for classify_posts: 0 minutes, 2 seconds
Memory usage for classify_posts: 0.0 MB
Execution time for filter_posts_with_filter_func: 0 minutes, 4 seconds
Memory usage for filter_posts_with_filter_func: 0.0 MB
2024-05-30 00:37:37,034 INFO [filters.py]: Finished filtering with function: filter_posts_have_no_nsfw_content
2024-05-30 00:37:37,034 INFO [filters.py]: Posts passing filter: 166077. Posts failing filter: 2663.
2024-05-30 00:37:37,034 INFO [filters.py]: Posts remaining after filtering with filter_posts_have_no_nsfw_content: 166256
Execution time for classify_posts: 0 minutes, 1 seconds
Memory usage for classify_posts: 0.0 MB
Execution time for filter_posts_with_filter_func: 0 minutes, 3 seconds
Memory usage for filter_posts_with_filter_func: 0.0 MB
2024-05-30 00:37:41,576 INFO [filters.py]: Finished filtering with function: filter_posts_have_no_spam
2024-05-30 00:37:41,576 INFO [filters.py]: Posts passing filter: 165163. Posts failing filter: 914.
2024-05-30 00:37:41,576 INFO [filters.py]: Posts remaining after filtering with filter_posts_have_no_spam: 165342
Execution time for classify_posts: 0 minutes, 1 seconds
Memory usage for classify_posts: 0.0 MB
Execution time for filter_posts_with_filter_func: 0 minutes, 3 seconds
Memory usage for filter_posts_with_filter_func: 0.0 MB
2024-05-30 00:37:45,885 INFO [filters.py]: Finished filtering with function: filter_posts_have_no_hate_speech
2024-05-30 00:37:45,885 INFO [filters.py]: Posts passing filter: 165163. Posts failing filter: 0.
2024-05-30 00:37:45,885 INFO [filters.py]: Posts remaining after filtering with filter_posts_have_no_hate_speech: 165342
2024-05-30 00:37:52,080 INFO [filters.py]: Completed post filtering in preprocessing pipeline.
Execution time for filter_posts: 0 minutes, 32 seconds
Memory usage for filter_posts: 33.6015625 MB
2024-05-30 00:37:53,220 INFO [helper.py]: After filtering, 165342 posts passed the filters (out of 292365 original posts).
Execution time for filter_latest_raw_data: 4 minutes, 0 seconds
Memory usage for filter_latest_raw_data: 3430.5 MB
2024-05-30 00:37:54,367 INFO [preprocessing_database.py]: Total number of posts before inserting into preprocessed posts database: 0
2024-05-30 00:37:54,368 INFO [preprocessing_database.py]: Total posts to attempt to insert: 292365
2024-05-30 00:38:58,352 ERROR [main.py]: Error in preprocessing pipeline: UNIQUE constraint failed: filteredpreprocessedposts.uri
2024-05-30 00:38:58,405 ERROR [main.py]: Traceback (most recent call last):
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/peewee.py", line 3291, in execute_sql
    cursor.execute(sql, params or ())
sqlite3.IntegrityError: UNIQUE constraint failed: filteredpreprocessedposts.uri

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/projects/p32375/bluesky-research/pipelines/preprocess_raw_data/main.py", line 24, in main
    preprocess_raw_data()
  File "/projects/p32375/bluesky-research/services/preprocess_raw_data/helper.py", line 53, in preprocess_raw_data
    batch_create_filtered_posts(filtered_posts)
  File "/projects/p32375/bluesky-research/lib/db/sql/preprocessing_database.py", line 94, in batch_create_filtered_posts
    ).execute()
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/peewee.py", line 2011, in inner
    return method(self, database, *args, **kwargs)
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/peewee.py", line 2082, in execute
    return self._execute(database)
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/peewee.py", line 2887, in _execute
    return super(Insert, self)._execute(database)
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/peewee.py", line 2600, in _execute
    cursor = database.execute(self)
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/peewee.py", line 3299, in execute
    return self.execute_sql(sql, params)
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/peewee.py", line 3289, in execute_sql
    with __exception_wrapper__:
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/peewee.py", line 3059, in __exit__
    reraise(new_type, new_type(exc_value, *exc_args), traceback)
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/peewee.py", line 192, in reraise
    raise value.with_traceback(tb)
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/peewee.py", line 3291, in execute_sql
    cursor.execute(sql, params or ())
peewee.IntegrityError: UNIQUE constraint failed: filteredpreprocessedposts.uri
```

It looks good, except that I want it to ignore rows that can't be inserted due to being duplicate. I have other processing that will prevent this later by loading the list of URIs that exist and then only preprocessing rows that aren't duplicates. But I suppose that I don't have anything to prevent that on write, so I have to add that in. We can do this by just adding a simple set lookup of URIs.

```python
def batch_create_filtered_posts(
    posts: list[FilteredPreprocessedPostModel]
) -> None:
    """Batch create filtered posts in chunks.

    Uses peewee's chunking functionality to write posts in chunks. Adds
    deduping on URI as well. This should already be done in various upstream
    places but is here for redundancy.
    """
    total_posts = FilteredPreprocessedPosts.select().count()
    logger.info(f"Total number of posts before inserting into preprocessed posts database: {total_posts}") # noqa
    seen_uris = set()
    unique_posts = []
    for post in posts:
        if post.uri not in seen_uris:
            seen_uris.add(post.uri)
            unique_posts.append(post)
    logger.info(f"Total posts to attempt to insert: {len(posts)}")
    logger.info(f"Total unique posts: {len(unique_posts)}")
    logger.info(f"Total duplicates: {len(posts) - len(unique_posts)}")
    with db.atomic():
        for idx in range(0, len(unique_posts), DEFAULT_BATCH_WRITE_SIZE):
            posts_to_insert = unique_posts[idx:idx + DEFAULT_BATCH_WRITE_SIZE]
            posts_to_insert_dicts = [post.dict() for post in posts_to_insert]
            FilteredPreprocessedPosts.insert_many(
                posts_to_insert_dicts
            ).on_conflict_ignore().execute()
    total_posts = FilteredPreprocessedPosts.select().count()
    logger.info(f"Total number of posts after inserting into preprocessed posts database: {total_posts}") # noqa
```

Looks to be fixed! (the memory usage seems quite high, yes, but I'll address that later)

```plaintext
2024-05-30 01:04:01,059 INFO [helper.py]: After filtering, 165342 posts passed the filters (out of 292365 original posts).
Execution time for filter_latest_raw_data: 1 minutes, 30 seconds
Memory usage for filter_latest_raw_data: 3383.85546875 MB
2024-05-30 01:04:02,202 INFO [preprocessing_database.py]: Total number of posts before inserting into preprocessed posts database: 0
2024-05-30 01:04:02,359 INFO [preprocessing_database.py]: Total posts to attempt to insert: 292365
2024-05-30 01:04:02,359 INFO [preprocessing_database.py]: Total unique posts: 292163
2024-05-30 01:04:02,359 INFO [preprocessing_database.py]: Total duplicates: 202
2024-05-30 01:04:59,651 INFO [preprocessing_database.py]: Total number of posts after inserting into preprocessed posts database: 292163
2024-05-30 01:04:59,697 INFO [helper.py]: Filtered data written to DB.
2024-05-30 01:05:01,566 INFO [main.py]: Completed preprocessing pipeline.
```

To test that the deduplication works, I reran immediately again and observed that there were no posts left to preprocess, which is what I expected:
```python
(bluesky_research) [jya0297@quser31 preprocess_raw_data]$ python main.py
2024-05-30 01:13:19,089 INFO [_client.py]: HTTP Request: POST https://bsky.social/xrpc/com.atproto.server.createSession "HTTP/1.1 200 OK"
HTTP Request: POST https://bsky.social/xrpc/com.atproto.server.createSession "HTTP/1.1 200 OK"
2024-05-30 01:13:19,228 INFO [_client.py]: HTTP Request: GET https://bsky.social/xrpc/app.bsky.actor.getProfile?actor=mindtechnologylab.bsky.social "HTTP/1.1 200 OK"
HTTP Request: GET https://bsky.social/xrpc/app.bsky.actor.getProfile?actor=mindtechnologylab.bsky.social "HTTP/1.1 200 OK"
Warning : `load_model` does not return WordVectorModel or SupervisedModel any more, but a `FastText` object which is very similar.
2024-05-30 01:13:20,944 INFO [main.py]: Starting preprocessing pipeline.
2024-05-30 01:13:22,956 INFO [load_data.py]: Loading latest raw data.
2024-05-30 01:13:23,665 INFO [load_data.py]: Latest preprocessed post timestamp: 2024-05-28-18:23:56
2024-05-30 01:14:05,083 INFO [helper.py]: Consolidated the formats of 285933 posts...
2024-05-30 01:14:13,114 INFO [helper.py]: Finished consolidating the formats of 285933 posts.
Execution time for consolidate_post_records: 0 minutes, 9 seconds
Memory usage for consolidate_post_records: 780.25 MB
2024-05-30 01:14:16,484 INFO [helper.py]: Consolidated the formats of 782 posts...
2024-05-30 01:14:16,502 INFO [helper.py]: Finished consolidating the formats of 782 posts.
Execution time for consolidate_post_records: 0 minutes, 1 seconds
Memory usage for consolidate_post_records: 0.00390625 MB
> /projects/p32375/bluesky-research/services/preprocess_raw_data/load_data.py(94)load_latest_raw_posts()
-> return consolidated_raw_posts
(Pdb) len(consolidated_raw_posts)
```

We can see that we've preprocessed ~300,000 posts. ~50% passed our filters.
```plaintext
(base) [jya0297@quser32 sql]$ sqlite3 filtered_posts.db
SQLite version 3.7.17 2013-05-20 00:56:22
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> .tables
filteredpreprocessedposts
sqlite> select count(*) from filteredpreprocessedposts;
292163
sqlite> select passed_filters, count(*) from filteredpreprocessedposts group by passed_filters;
0|127000
1|165163
```

I wish that I could group by source feed, but since `metadata` is a text field, I can't do that. I suppose that this is another use case for having proper database migrations in place, so I'm sure that I'll have to solve that problem soon, probably in the coming weeks, but not today.

## Experimented with GPU inference on KLC
I want to figure out how to do inference on KLC. There are resources [here](https://github.com/rs-kellogg/krs-openllm-cookbook/tree/main/scripts/slurm_basics) on how to do it.


### Running a simple example
To start, I want to just send a simple job and see if I can get a result back as intended. I'll load some data and then just print out some stuff so I can verify that the script works as intended and can load the data.

First, I'll rerun the same script as what is in the Slurm basics.
```python
"""First test.

Intended to rerun the same test script as in the "Slurm basics" repo.

Reference: https://github.com/rs-kellogg/krs-openllm-cookbook/blob/main/scripts/slurm_basics/pytorch_gpu_test.py
""" # noqa

import torch

# Check if CUDA is available
if torch.cuda.is_available():
    print("CUDA is available!")
    print("Number of GPUs available:", torch.cuda.device_count())
    print("GPU:", torch.cuda.get_device_name(0))
else:
    print("CUDA is not available.")

# Check if CUDA is available and set the device accordingly
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# Print whether a GPU or CPU is being used
if device.type == 'cuda':
    print("Using GPU")
else:
    print("Using CPU")

# Create two random tensors
tensor1 = torch.randn(1000, 1000, device=device)
tensor2 = torch.randn(1000, 1000, device=device)

# Add the two tensors, the operation will be performed on the GPU if available
result = tensor1 + tensor2

print(result)
```

Now I'll submit to the cluster using a bash script. I'm basing it off the example file and [this](https://services.northwestern.edu/TDClient/30/Portal/KB/ArticleDet?ID=1796). 
```bash
#!/bin/bash
# Resources for submitting jobs: https://services.northwestern.edu/TDClient/30/Portal/KB/ArticleDet?ID=1796

#SBATCH --account=p32375 # project ID
#SBATCH --partition gengpu
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --gres=gpu:a100:1
#SBATCH --time 0:10:00
#SBATCH --mem=2G
#SBATCH --constraint=pcie
#SBATCH --job-name=test_job_jya0297
#SBATCH --output=/projects/p32375/bluesky-research/lib/log/logfile_test.log

# Get the current directory
DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# load conda env
CONDA_PATH="/hpc/software/mamba/23.1.0/etc/profile.d/conda.sh"

# set pythonpath
PYTHONPATH="/projects/p32375/bluesky-research/:$PYTHONPATH"

PYTHONFILE_NAME="test_script_1.py"

source $CONDA_PATH
conda activate bluesky_research
export PYTHONPATH=$PYTHONPATH
cd $DIR
python $PYTHONFILE_NAME

echo "Completed slurm job."
```

Let me check the results.

```plaintext
(bluesky_research) [jya0297@quser31 2024-05-30-klc-experiments]$ sbatch submit_job.sh
Submitted batch job 1946558
(bluesky_research) [jya0297@quser31 2024-05-30-klc-experiments]$ squeue -u jya0297
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
           1946558    gengpu test_job  jya0297 PD       0:00      1 (None)
```

```plaintext
Starting slurm job.
DIR: /var/spool/slurmd/job1947318
CONDA_PATH: /hpc/software/mamba/23.1.0/etc/profile.d/conda.sh
PYTHONPATH: /projects/p32375/bluesky-research/:/projects/p32375/bluesky-research/:$PATH
python: can't open file '/var/spool/slurmd/job1947318/test_script_1.py': [Errno 2] No such file or directory
Completed slurm job.

```

I tried variations of this for an hour and didn't get much luck. I'll wait until my meeting with Research Support later today for some clarity. No need to keep pushing it, I've given it a decent shot so far and I'm sure I'll get resolution in a few hours.

## Starting analysis of pilot data posts
After running the preprocessing pipeline, I now have n~150,000 posts that passed filtering (out of n~300,000 original posts). I'll use these posts for my basic analysis.

What I want to do is the following:
- Export the pilot data from the databases (probably as a .csv file of some sort).
- Load the data from .csv and then run inference:
    - Perspective API, to track toxicity and constructiveness.
    - Some LLM (probably some variant of Llama), for sociopolitical and political ideology classifications.
- Write the inference results to a database (think I can just use SQLite again, so I can ship quickly).
- Do analysis on the results (e.g., plotting histograms of the probability distributions).
- Planned for meeting with Research Support

### Exporting the pilot data from the databases

## Improved basic logging and backing up logging.

## Planned for meeting with Research Support
I am meeting with Research Support later to go over how to use KLC. What I'd like to learn during that call is how to successfully submit jobs to the compute cluster. That way, I can begin to run inference. In addition, I'd like to know best practices for classifying with the LLMs on the compute cluster, such as ideal job configurations, how much memory and nodes to use, expected runtime, etc.

This should also unblock me from my previous experiments with LLMs so that I can start classifications tomorrow. In theory, even if I can't figure that out, I could just use staggered calls to an external API (e.g., Groq, Gemini), but I'd like to figure out how to use the models on KLC properly if possible.
