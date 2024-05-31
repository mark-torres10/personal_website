---
layout: single
title:  "Research updates, 2024-05-31"
date:   2024-05-31 13:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/research-updates-2024-05-31
---
# Research updates, 2024-05-31

Today, I worked on the following:

- Met with Research Support.
- Removed my databases from Git tracking.
- Hunted down problems with the preprocessing pipeline.
- Thought through scalability concerns.
- Built a batch inference pipeline for classifications with the Perspective API.

## Met with Research Support

## Removed my databases from Git tracking
I keep accidentally overwriting my local databases with the old versions that I've accidentally saved to Github. I already have file backups set on a cron job so this isn't the worst, but I don't want to restore from backups if possible. So, I removed them from Git tracking. I found [this](https://git-scm.com/docs/git-rm) documentation helpful (as well as ChatGPT, of course, but for any "rm" stuff, I like to check the docs to be 100% sure that it does what I think it will do).

```bash
git rm --cached synced_record_posts.db
git rm --cached filtered_posts.db
git rm --cached '*.db'
```

From looking at VSCode, it looks like the .db files are no longer being tracked, so that's good!
![DB filetracking](/assets/images/2024-05-31-research-updates/db_filetracking_grey.png)

## Hunted down problems with the preprocessing pipeline
I checked the preprocessed posts database and it had n~6,600 posts, so it seemed like a duplicate of yesterday's problems. I investigated and it looks like when I reset my git commits in order to undo when I committed very large files into Git, I accidentally reset too far and I removed the patches that I had put in place to initially fix this issue. Luckily this wasn't too bad to figure out, and like yesterday I made the changes and deleted the database to rebuild it from scratch.

One change I did make though was add deduping prior to insertion. I've been explicitly ignoring any errors in order to avoid duplicate URI problems and overwrite them, but I realize that this actually obfuscates a lot of possible problems, such as this one, where the writes silently failed.
```
FilteredPreprocessedPosts.insert_many(
    posts_to_insert_dicts
).on_conflict_ignore().execute()
```

Instead, I removed the `on_conflict_ignore()` and just added deduping instead. Not sure how it scales, might need a Bloom filter at some point, but it works for now:

```python
def get_previously_filtered_post_uris() -> set[str]:
    """Get previous IDs from the DB."""
    previous_ids = FilteredPreprocessedPosts.select(FilteredPreprocessedPosts.uri)  # noqa
    return set([p.uri for p in previous_ids])


def dedupe_posts(
    posts: list[FilteredPreprocessedPostModel]
) -> list[FilteredPreprocessedPostModel]:
    """
    Dedupes posts to make sure that we don't insert any duplicates.

    Note: will need to see how this scales, especially at the scale of millions
    of posts.
    """
    existing_uris: set[str] = get_previously_filtered_post_uris()
    seen_uris = set()
    seen_uris.update(existing_uris)
    unique_posts = []
    for post in posts:
        if post.uri not in seen_uris:
            seen_uris.add(post.uri)
            unique_posts.append(post)
    logger.info(f"Total posts to attempt to insert: {len(posts)}")
    logger.info(f"Total unique posts: {len(unique_posts)}")
    logger.info(f"Total duplicates: {len(posts) - len(unique_posts)}")
    return unique_posts


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
    unique_posts: list[FilteredPreprocessedPostModel] = dedupe_posts(posts)

    with db.atomic():
        for idx in range(0, len(unique_posts), DEFAULT_BATCH_WRITE_SIZE):
            posts_to_insert = unique_posts[idx:idx + DEFAULT_BATCH_WRITE_SIZE]
            posts_to_insert_dicts = [post.dict() for post in posts_to_insert]
            FilteredPreprocessedPosts.insert_many(
                posts_to_insert_dicts
            ).execute()
    total_posts = FilteredPreprocessedPosts.select().count()
    logger.info(f"Total number of posts after inserting into preprocessed posts database: {total_posts}") # noqa
```

## Thought through scalability concerns
Now that I'm building out the classification pipelines, I need to start thinking more deeply about scalability. I suppose that this was true for the preprocessing pipeline, but that is more narrowly scoped and I'm more confident that that will just work OK as is. It's unlikely that I can classify all the posts in one go, (more true for LLM than perspective API), so I'm trying to work through a few ideas.

One idea is that during processing, I should sort posts by date ascending so that I process the earliest posts first, and then record the timestamp as the timestamp of the last post processed, unless I get through all the posts, in which case it should be the timestamp in which the classification was run. Another strategy is to match the URIs against the URIs of posts that have been processed, in order to see what's not been processed yet. Yet another strategy is keeping the posts in a cache and then evicting them once they've been classified.

Now that I think about it, the simplest option might be to add a field, latest_preprocessed_post, or something like that, that tracks the latest preprocessed timestamp of a post that's been classified. That way, I can split up a batch of 100,000 posts into 10 batches, for example, and run those jobs.

If I'm going to batch, I need an orchestrator file that can send jobs out in batches, and then check the status of those jobs, and then collect the results. This is more true for the LLM than the Perspective API, but it is something to keep in mind.

I think that what I actually need is to use the latest preprocessing timestamp of the latest completely successfully run batch of posts. That is the latest
guaranteed source of truth, and I can backfill everything more recent than that, even if it leads to some duplicate processing.

I could do a simple URI lookup I suppose, but I would have to cache those results and then compute it offline.

For the current problem, I only have ~n=100,000 posts, so scale-wise, this shouldn't be a problem, at least for now. I might have to build this in in
order to use the LLMs on KLC, but I'll deal with that when I get there I suppose.

My current concern is that since I have to split a single job into a batch job, I need a way to account for the case that one of those jobs fails, and when it fails, how do I make sure that the posts that weren't processed will be processed by a future job. I think that the most promising option is just recording each classification job and recording the ID of the job, the job status, and the min and max preprocessing_timestamp of a given job, so that if the job fails, I can just pull those same posts and re-do inference again. Another idea that I think is promising is having an offline job that goes through the preprocessed database and checks to see which posts were preprocessed earlier than the latest classification job, since these are posts that we must have tried to classify before but somehow failed, and adding those to a cache of some sort that we can then pull from in a subsequent classification job.

But, I might, in the coming weeks, have to really investigate scalability concerns to make sure that everything works as intended. Right now I just need something that works for the present problem. For a naive implementation, I'll just use the latest classification timestamp as my filter and then assume that when I split up a job into multiple batches, that it won't fail.
