---
layout: single
title:  "Experiments with LLM classification for political content (Part IX)"
date:   2024-05-22 04:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/llm-experiments-pt-ix
---
# Using LLMs to classify if social media posts are political or not (Part IX).
I'm working on a project that involves gathering social media posts from [Bluesky](https://bsky.app/) and analyzing them. Part of that project requires knowing which posts are about political or social topics, and if so, what political side they support.

[Previously](https://markptorres.com/research/llm-experiments-pt-viii), I worked on migrating the data pipeline to KLC. Now I'm working on more work with using KLC for the project. Specifically, I want to move the data preprocessing and filtering steps to KLC. 

Specifically, I'll do the following:
- Consolidate the data formats across the different sync types.
- Update the data preprocessing and filtering steps.
- Create a pipeline to run the data preprocessing and filtering steps.
- Run the pipeline on a cron job on the compute cluster.

## Consolidate the data formats across the different sync types
I transformed the data formats for both the posts from the firehose and the posts from the most liked feeds into consolidated formats, but I should process them similarly downstream. Because of that, I need to consolidate them to a shared format.

Specifically, I'll create the following models to consolidate the data across both sources:
```python
"""Models for consolidating post records."""
from typing import Optional
import typing_extensions as te

from pydantic import BaseModel, Field

from lib.db.bluesky_models.transformations import (
    PostMetadataModel,
    TransformedProfileViewBasicModel,
    TransformedRecordModel
)

class ConsolidatedPostRecordMetadataModel(BaseModel):
    synctimestamp: str = Field(..., description="The synctimestamp of the post.")  # noqa
    url: Optional[str] = Field(..., description="The URL of the post. Available only if the post is from feed view. Firehose posts won't have this hydrated.") # noqa
    source: te.Literal["firehose", "most_liked"] = Field(..., description="The source feed of the post. Either 'firehose' or 'most_liked'") # noqa
    processed_timestamp: str = Field(..., description="The timestamp when the post was processed.")  # noqa


class ConsolidatedMetrics(BaseModel):
    like_count: Optional[int] = Field(
        default=None, description="The like count of the post."
    )
    reply_count: Optional[int] = Field(
        default=None, description="The reply count of the post."
    )
    repost_count: Optional[int] = Field(
        default=None, description="The repost count of the post."
    )


class ConsolidatedPostRecordModel(BaseModel):
    uri: str = Field(..., description="The URI of the post.")
    cid: str = Field(..., description="The CID of the post.")
    indexed_at: str = Field(..., description="The timestamp of when the post was indexed by Bluesky.")  # noqa
    author: TransformedProfileViewBasicModel = Field(..., description="The author of the post.")  # noqa
    metadata: PostMetadataModel = Field(..., description="The metadata of the post.")  # noqa
    record: TransformedRecordModel = Field(..., description="The record of the post.")  # noqa
    metrics: Optional[ConsolidatedMetrics] = Field(default=None, description="Post engagement metrics. Only available for posts from feed view, not firehose.")  # noqa
```

To support this, I had to change the format of the firehose posts, where I initially unpacked all the record fields such as text and embeds. Instead of doing that, I repackaged them into a single "Record" object.
```python
class TransformedRecordModel(BaseModel):
    """Model for the transformed record."""
    created_at: str = Field(..., description="The timestamp of when the record was created on Bluesky.")  # noqa
    text: str = Field(..., description="The text of the record.")
    embed: Optional[ProcessedEmbed] = Field(default=None, description="The embeds in the record, if any.")  # noqa
    entities: Optional[str] = Field(default=None, description="The entities of the record, if any. Separated by a separator.")  # noqa
    facets: Optional[str] = Field(default=None, description="The facets of the record, if any. Separated by a separator.")  # noqa
    labels: Optional[str] = Field(default=None, description="The labels of the record, if any. Separated by a separator.")  # noqa
    langs: Optional[str] = Field(default=None, description="The languages of the record, if specified.")  # noqa
    reply_parent: Optional[str] = Field(default=None, description="The parent post that the record is responding to in the thread, if any.")  # noqa
    reply_root: Optional[str] = Field(default=None, description="The root post of the thread, if any.")  # noqa
    tags: Optional[str] = Field(default=None, description="The tags of the record, if any.")  # noqa
    py_type: te.Literal["app.bsky.feed.post"] = Field(default="app.bsky.feed.post", frozen=True)  # noqa

    @validator('synctimestamp')
    def validate_synctimestamp(cls, v):
        if not re.match(r'^\d{4}-\d{2}-\d{2}-\d{2}:\d{2}:\d{2}$', v):
            raise ValueError("synctimestamp must be in 'YYYY-MM-DD-HH:MM:SS' format (e.g., '2024-04-23-04:41:17')")  # noqa
        return v


class PostMetadataModel(BaseModel):
    url: str = Field(..., description="The URL of the post.")
    source_feed: str = Field(..., description="The source feed of the post.")
    synctimestamp: str = Field(..., description="The synctimestamp of the post.")  # noqa


class TransformedRecordWithAuthorModel(BaseModel):
    """Model for the transformed record post, with author information.

    Note: the author isn't guaranteed to exist. For example, we might want to
    get posts in this format based on their Record object, which doesn't have
    author information hydrated, so in those cases, the author field would be
    blank.
    """
    uri: str = Field(..., description="The URI of the post.")
    cid: str = Field(..., description="The CID of the post.")
    author: str = Field(..., description="The DID of the author of the post.")
    metadata: PostMetadataModel = Field(..., description="The metadata of the post.")  # noqa
    record: TransformedRecordModel = Field(..., description="The record of the post.")  # noqa

    @validator('author')
    def validate_author(cls, v):
        if not v.startswith("did:"):
            raise ValueError("Author must start with 'did:'")
        return v

    @validator('uri')
    def validate_uri(cls, v):
        if not v.startswith("at://did:plc:"):
            raise ValueError("URI must start with 'at://did:plc:'")
        return v
```

This more closely tracks the format of the feed view posts:
```python
class TransformedFeedViewPostModel(BaseModel):
    uri: str = Field(..., description="The URI of the post.")
    cid: str = Field(..., description="The CID of the post.")
    metadata: PostMetadataModel = Field(..., description="The metadata of the post.")  # noqa
    author: TransformedProfileViewBasicModel = Field(..., description="The author of the post.")  # noqa
    record: TransformedRecordModel = Field(..., description="The record of the post.")  # noqa
    indexed_at: str = Field(..., description="The timestamp of when the post was indexed by Bluesky.")  # noqa
    like_count: Optional[int] = Field(default=None, description="The like count of the post.")  # noqa
    reply_count: Optional[int] = Field(default=None, description="The reply count of the post.")  # noqa
    repost_count: Optional[int] = Field(default=None, description="The repost count of the post.")  # noqa
```

Now I can convert the posts from both sources into a consolidated format.
```python
"""Consolidates different types of post records into a single format."""
from typing import Union

from lib.constants import current_datetime_str
from lib.db.bluesky_models.transformations import (
    TransformedRecordWithAuthorModel, TransformedFeedViewPostModel
)
from lib.helper import track_performance
from lib.log.logger import get_logger
from services.consolidate_post_records.models import (
    ConsolidatedMetrics, ConsolidatedPostRecordModel,
    ConsolidatedPostRecordMetadataModel
)

logger = get_logger(__file__)


def consolidate_firehose_post(post: TransformedRecordWithAuthorModel) -> ConsolidatedPostRecordModel:  # noqa
    """Transforms the firehose posts into the consolidated format."""
    metadata_dict = {
        "synctimestamp": post.metadata.synctimestamp,
        "url": post.metadata.url,
        "source": "most_liked_feeds",
        "processed_timestamp": current_datetime_str
    }
    metadata: ConsolidatedPostRecordMetadataModel = (
        ConsolidatedPostRecordMetadataModel(**metadata_dict)
    )
    metrics_dict = {
        "like_count": None, "reply_count": None, "repost_count": None
    }
    metrics: ConsolidatedMetrics = ConsolidatedMetrics(**metrics_dict)
    res = {
        "uri": post.uri,
        "cid": post.cid,
        "indexed_at": post.indexed_at,
        "author": post.author,
        "metadata": metadata,
        "record": post.record,
        "metrics": metrics
    }
    return ConsolidatedPostRecordModel(**res)


def consolidate_feedview_post(post: TransformedFeedViewPostModel) -> ConsolidatedPostRecordModel:  # noqa
    """Transforms the feed view posts into the consolidated format."""
    metadata_dict = {
        "synctimestamp": post.metadata.synctimestamp,
        "url": post.metadata.url,
        "source": "most_liked_feeds",
        "processed_timestamp": current_datetime_str
    }
    metadata: ConsolidatedPostRecordMetadataModel = (
        ConsolidatedPostRecordMetadataModel(**metadata_dict)
    )
    metrics_dict = {
        "like_count": post.like_count,
        "reply_count": post.reply_count,
        "repost_count": post.repost_count
    }
    metrics: ConsolidatedMetrics = ConsolidatedMetrics(**metrics_dict)
    res = {
        "uri": post.uri,
        "cid": post.cid,
        "indexed_at": post.indexed_at,
        "author": post.author,
        "metadata": metadata,
        "record": post.record,
        "metrics": metrics
    }
    return ConsolidatedPostRecordModel(**res)


def consolidate_post_record(
    post: Union[TransformedFeedViewPostModel, TransformedRecordWithAuthorModel]
) -> ConsolidatedPostRecordModel:
    if isinstance(post, TransformedFeedViewPostModel):
        return consolidate_feedview_post(post)
    elif isinstance(post, TransformedRecordWithAuthorModel):
        return consolidate_firehose_post(post)
    else:
        raise ValueError(f"Unknown post type: {type(post)}")


@track_performance
def consolidate_post_records(
    posts: list[
        Union[TransformedFeedViewPostModel, TransformedRecordWithAuthorModel]
    ]
) -> list[ConsolidatedPostRecordModel]:
    logger.info(f"Consolidated the formats of {len(posts)} posts...")
    res = [consolidate_post_record(post) for post in posts]
    logger.info(f"Finished consolidating the formats of {len(posts)} posts.")
    return res
```

Next, I'll have to redo the data syncs to support this new format. Luckily we're still on the MVP stage of the project so changing the schema this dramatically is pretty low-cost.

## Perform a database migration of the old firehose data
With the updates, the old firehose data has to be migrated to the format of the new firehose data. I wrote the following migration script to do so. This migrates the data to a temp table, then drops the original table and renames the temp table as the original table.

```python
import json

class TransformedRecordWithAuthor(BaseModel):
    """Class for the (transformed) raw posts."""
    uri = peewee.CharField(unique=True)
    created_at = peewee.TextField()
    # for long text. Technically a post can just be an image or video and
    # not have text.
    text = peewee.TextField(null=True)
    embed = peewee.TextField(null=True)  # for embedded content
    langs = peewee.CharField(null=True)  # sometimes the langs aren't provided
    entities = peewee.CharField(null=True)
    facets = peewee.CharField(null=True)  # https://www.pfrazee.com/blog/why-facets # noqa
    labels = peewee.CharField(null=True)
    reply = peewee.CharField(null=True)
    reply_parent = peewee.CharField(null=True)
    reply_root = peewee.CharField(null=True)
    tags = peewee.CharField(null=True)
    py_type = peewee.CharField()
    cid = peewee.CharField(index=True)
    author = peewee.CharField()
    synctimestamp = peewee.CharField()


class TransformedRecordsTmp(BaseModel):
    """Temporary table for the `TransformedRecordWithAuthor` table."""
    uri = peewee.CharField(unique=True)
    cid = peewee.CharField(index=True)
    author = peewee.CharField()
    metadata = peewee.TextField()
    record = peewee.TextField()


def prepare_records_batch(records):
    prepared_records: list[dict] = []
    for record in records:
        metadata = {
            'synctimestamp': record.synctimestamp
        }
        record_data = {
            'created_at': record.created_at,
            'text': record.text,
            'embed': record.embed,
            'langs': record.langs,
            'entities': record.entities,
            'facets': record.facets,
            'labels': record.labels,
            'reply': record.reply,
            'reply_parent': record.reply_parent,
            'reply_root': record.reply_root,
            'tags': record.tags,
            'py_type': record.py_type,
        }
        prepared_records.append({
            'uri': record.uri,
            'cid': record.cid,
            'author': record.author,
            'metadata': json.dumps(metadata),
            'record': json.dumps(record_data)
        })
    return prepared_records

def migrate_to_new_schema():
    db.drop_tables([TransformedRecordsTmp])
    db.create_tables([TransformedRecordsTmp])
    print("Finished creating temp table.")
    old_records = TransformedRecordWithAuthor.select()
    original_table_count = TransformedRecordWithAuthor.select().count()
    BATCH_SIZE = 1000

    # Insert data in batches
    print("Starting insertion of records to temp table.")
    for i in range(0, len(old_records), BATCH_SIZE):
        batch_records = old_records[i:i + BATCH_SIZE]
        prepared_batch = prepare_records_batch(batch_records)
        with db.atomic():
            TransformedRecordsTmp.insert_many(prepared_batch).execute()
        print(f"Completed insertion at index {i} out of {original_table_count}") # noqa
    print("Completed insertion of records to temp table.")
    record_count = TransformedRecordsTmp.select().count()
    print(f"Original table count: {original_table_count}")
    print(f"Tmp record count: {record_count}")
    print(f"Original table count == tmp table count? {original_table_count == record_count}") # noqa
    if original_table_count == record_count:
        print("Dropping old table and renaming temp table to new table.")
        db.drop_tables([TransformedRecordWithAuthor])
        db.execute_sql('ALTER TABLE TransformedRecordsTmp RENAME TO TransformedRecordWithAuthor')
    print(f"Migration completed.")


if __name__ == "__main__":
    migrate_to_new_schema()
```

This now migrate the old firehsoe data to conform to the new data format.

## Update the data preprocessing and filtering steps.
I [already](https://markptorres.com/research/llm-experiments-pt-v) built out the service to do the preprocessing and filtering steps. Now I need to expand on that to make it work on this new consolidated post format.

The broad steps are:
1. Load the data

First, I'll load the data from both sources:

```python
@track_performance
def load_latest_raw_posts(
    sources: list[str] = ["firehose", "most_liked"]
) -> list[ConsolidatedPostRecordModel]:
    """Loads raw data from the firehose DB.
    """
    logger.info("Loading latest raw data.")
    latest_preprocessing_timestamp: str = load_latest_preprocessing_timestamp()
    consolidated_raw_posts: list[ConsolidatedPostRecordModel] = []
    for source in sources:
        if source == "firehose":
            posts: list[ConsolidatedPostRecordModel] = load_firehose_posts(
                latest_preprocessing_timestamp=latest_preprocessing_timestamp
            )
        elif source == "most_liked":
            posts: list[ConsolidatedPostRecordModel] = load_feedview_posts(
                latest_preprocessing_timestamp=latest_preprocessing_timestamp
            )
        else:
            raise ValueError(f"Data source not recognized: {source}")
        consolidated_raw_posts.extend(posts)
    consolidated_raw_posts: list[ConsolidatedPostRecordModel] = (
        filter_previously_preprocessed_posts(posts=consolidated_raw_posts)
    )
    return consolidated_raw_posts
```

Each data source has their own logic:
```python
def load_firehose_posts(
    latest_preprocessing_timestamp: Optional[str] = None
) -> list[ConsolidatedPostRecordModel]:
    """Loads latest synced firehose posts from SQLite and then consolidates
    their format."""
    posts: list[dict] = get_latest_firehose_posts(
        k=None, latest_preprocessing_timestamp=latest_preprocessing_timestamp
    )
    transformed_posts: list[TransformedRecordWithAuthorModel] = [
        TransformedRecordWithAuthorModel(**post) for post in posts
    ]
    return consolidate_post_records(posts=transformed_posts)


def load_feedview_posts(
    latest_preprocessing_timestamp: Optional[str] = None
) -> list[ConsolidatedPostRecordModel]:
    """Loads latest synced feedview posts from MongoDB and then consolidates
    their format."""
    posts: list[dict] = load_collection(
        collection=mongo_collection,
        limit=None,
        latest_timestamp=latest_preprocessing_timestamp,
        timestamp_fieldname="metadata.synctimestamp"
    )
    transformed_posts: list[TransformedFeedViewPostModel] = [
        TransformedFeedViewPostModel(**post) for post in posts
    ]
    return consolidate_post_records(posts=transformed_posts)
```

We get the latest posts from the firehose.

```python
def get_latest_firehose_posts(
    k: int=None, latest_preprocessing_timestamp: str=None
) -> List[dict]:
    """Get the latest firehose posts from the database."""
    query = TransformedRecordWithAuthor.select()
    if latest_preprocessing_timestamp:
        query = query.where(
            TransformedRecordWithAuthor.synctimestamp
            > latest_preprocessing_timestamp
        )
    if k:
        query = query.limit(k)
    res = list(query)
    res_dicts = [r.__dict__['__data__'] for r in res]
    return res_dicts
```

2. Filter and preprocess the data

We previously created code to filter posts, but we can extend that to this use case.

Here is the top-level function to handle preprocessing data:
```python
def preprocess_raw_data() -> None:
    """Preprocesses raw data.

    We'll preprocess using the following steps:
    1. Filter the raw data.
    2. Preprocess the filtered data.
    3. Validate the preprocessed data.
    4. Write the filtered, preprocessed, validated data to the database.
    """
    filtered_posts, total_raw_posts,  num_posts_passed_filters = (
        filter_latest_raw_data()
    )
    batch_create_filtered_posts(filtered_posts)
    print(f"Filtered data written to DB. After filtering, {num_posts_passed_filters} posts passed the filters (out of {total_raw_posts} original posts).")  # noqa
```

First, we filter the latest raw data (latest defined as being synced later than the synctimestamp of the most recently processed post):
```python
def filter_latest_raw_data() -> tuple[list[FilteredPreprocessedPostModel], int, int]:  # noqa
    """Filters the latest raw data.

    Loads the latest posts, filters them, and writes the filtered data to the
    database. Writes all posts and their filtered status, so we can track
    which posts passed the filters and which ones didn't and so we don't
    duplicate filtering in the future.
    """
    latest_raw_posts: list[ConsolidatedPostRecordModel] = load_latest_raw_posts()  # noqa
    num_posts: int = len(latest_raw_posts)
    print(f"Loaded {num_posts} posts for filtering.")
    filtered_posts: list[FilteredPreprocessedPostModel] = filter_posts(
        posts=latest_raw_posts
    )
    total_raw_posts = len(latest_raw_posts)
    num_posts_passed_filters = sum(
        post.passed_filters for post in filtered_posts
    )
    return filtered_posts, total_raw_posts, num_posts_passed_filters
```

We load the latest raw posts:
```python
def load_firehose_posts(
    latest_preprocessing_timestamp: Optional[str] = None
) -> list[ConsolidatedPostRecordModel]:
    """Loads latest synced firehose posts from SQLite and then consolidates
    their format."""
    posts: list[dict] = get_latest_firehose_posts(
        k=None, latest_preprocessing_timestamp=latest_preprocessing_timestamp
    )
    transformed_posts: list[TransformedRecordWithAuthorModel] = [
        TransformedRecordWithAuthorModel(**post) for post in posts
    ]
    return consolidate_post_records(posts=transformed_posts)


def load_feedview_posts(
    latest_preprocessing_timestamp: Optional[str] = None
) -> list[ConsolidatedPostRecordModel]:
    """Loads latest synced feedview posts from MongoDB and then consolidates
    their format."""
    posts: list[dict] = load_collection(
        collection=mongo_collection,
        limit=None,
        latest_timestamp=latest_preprocessing_timestamp,
        timestamp_fieldname="metadata.synctimestamp"
    )
    transformed_posts: list[TransformedFeedViewPostModel] = [
        TransformedFeedViewPostModel(**post) for post in posts
    ]
    return consolidate_post_records(posts=transformed_posts)


def filter_previously_preprocessed_posts(
    posts: list[ConsolidatedPostRecordModel]
) -> list[ConsolidatedPostRecordModel]:
    previous_uris: set[str] = get_previously_filtered_post_uris()
    # OK for now, and will prob be OK, but in case this doesn't scale,
    # I could explore something like a Bloom filter.
    return [post for post in posts if post.uri not in previous_uris]


@track_performance
def load_latest_raw_posts(
    sources: list[str] = ["firehose", "most_liked"]
) -> list[ConsolidatedPostRecordModel]:
    """Loads raw data from the firehose DB.
    """
    logger.info("Loading latest raw data.")
    latest_preprocessed_post_timestamp: str = load_latest_preprocessed_post_timestamp()  # noqa
    consolidated_raw_posts: list[ConsolidatedPostRecordModel] = []
    for source in sources:
        if source == "firehose":
            posts: list[ConsolidatedPostRecordModel] = load_firehose_posts(
                latest_preprocessing_timestamp=latest_preprocessed_post_timestamp  # noqa
            )
        elif source == "most_liked":
            posts: list[ConsolidatedPostRecordModel] = load_feedview_posts(
                latest_preprocessing_timestamp=latest_preprocessed_post_timestamp  # noqa
            )
        else:
            raise ValueError(f"Data source not recognized: {source}")
        consolidated_raw_posts.extend(posts)
    consolidated_raw_posts: list[ConsolidatedPostRecordModel] = (
        filter_previously_preprocessed_posts(posts=consolidated_raw_posts)
    )
    return consolidated_raw_posts
```

Once these are loaded, we then pass them through filtering steps. In this filtering step, we do the following:
1. We pass the posts through a series of filters
- The first filter is on language, where we filter out posts that are not English. This is the filter that will remove the most posts, hence why it is first.
- We then pass through other filters, such as if a post has spam, if it has hate speech, and others. Some of these (e.g., hate speech) are not implemented yet so I just have pass-through functions for these.
- At each stage, we record which posts pass the filters and we move those to the next filters. We also record which posts fail at the given filter and note not only that they failed, but at which step they failed
2. We then consolidate the posts with the results of filtering, to record if a given post passed filtering and if it failed, at which step.

```python
@track_performance
def filter_posts(
    posts: list[ConsolidatedPostRecordModel]
) -> list[FilteredPreprocessedPostModel]:
    """Applies the filtering steps and returns the posts along with their
    status.

    Returns the following fields per dictionary:
    :uri: str: The URI of the post.
    :passed_filters: bool: Whether the post passed the filters or not.
    :filtered_at: datetime: The timestamp of when the post was filtered.
    :filtered_by_func: if filtered out, which function filtered it out.

    Each filtering function returns the following for a given post:
    :uri: str: The URI of the post.
    :<filter_name>: bool: Whether the post passed the filter or not.

    Example:
        - {"uri": post["uri"], "has_spam": has_spam}

    For posts, we run the filtering function and return two sets of URIs:
    :passed_filters: set[str]: the URIs of the posts that passed the filter.
    :failed_filters: set[str]: the URIs of the posts that failed the filter.

    We add the URIs of those that failed that filter to the output. We then
    pass to the next filter only the URIs that passed the previous filter.

    After all the filters are done, we add the remaining URIs to the output
    as the URIs of the posts that have passed all the filters.
    """  # noqa
    # do any preprocessing for posts before filtering
    posts: list[ConsolidatedPostRecordModel] = preprocess_posts(posts)

    # we need to run the language filter first since this will filter the
    # majority of posts (60% - 80% of posts per batch).
    results_after_english_filter = filter_posts_with_filter_func(
        posts=posts, filter_func=filter_text_is_english, label="is_english"
    )
    english_post_uris = results_after_english_filter["passed_filters"]

    # for the posts that have been filtered out, let's add them to our
    # output.
    res = [
        {
            "uri": uri,
            "passed_filters": False,
            "filtered_at": current_datetime_str,
            "filtered_by_func": filter_text_is_english.__name__
        }
        for uri in results_after_english_filter["failed_filters"]
    ]

    # we apply downstream filters only on English posts.
    posts_to_filter = [
        post for post in posts if post.uri in english_post_uris
    ]

    # for each filter, we apply the filter and add the results to the output.
    # the order of these filters doesn't particularly matter, unless we have
    # a specific reason to prefer one ordering over another.
    filter_funcs_with_labels: list[tuple] = [
        (filter_posts_not_written_by_bot, "is_not_from_possible_bot_account"),
        (filter_posts_have_no_nsfw_content, "has_no_nsfw_content"),
        (filter_posts_have_no_spam, "has_no_spam"),
        (filter_posts_have_no_hate_speech, "has_no_hate_speech")
    ]

    for (filter_func, label) in filter_funcs_with_labels:
        results = filter_posts_with_filter_func(
            posts=posts_to_filter, filter_func=filter_func, label=label
        )
        res.extend([
            {
                "uri": uri,
                "passed_filters": False,
                "filtered_at": current_datetime_str,
                "filtered_by_func": filter_func.__name__
            }
            for uri in results["failed_filters"]
        ])
        # update the posts to filter if it has passed all the filters so far.
        posts_to_filter = [
            post
            for post in posts_to_filter
            if post.uri in results["passed_filters"]
        ]

    # whatever posts are left, are the ones that have passed all filters.
    res.extend([
        {
            "uri": post.uri,
            "passed_filters": True,
            "filtered_at": current_datetime_str,
            "filtered_by_func": None
        }
        for post in posts_to_filter
    ])

    # we now create a hash map of the results, with URI as the key.
    uri_to_results_map = {result["uri"]: result for result in res}

    # we then work through the original list of posts and create the resulting
    # objects accordingly.
    filtered_posts: list[FilteredPreprocessedPostModel] = []
    for post in posts:
        uri = post.uri
        filtering_results = uri_to_results_map[uri]
        filtered_post_result = {
            "uri": uri,
            "cid": post.cid,
            "indexed_at": post.indexed_at,
            "author": post.author,
            "metadata": post.metadata,
            "record": post.record,
            "metrics": post.metrics,
            "passed_filters": filtering_results["passed_filters"],
            "filtered_at": filtering_results["filtered_at"],
            "filtered_by_func": filtering_results["filtered_by_func"],
            "synctimestamp": post.metadata.synctimestamp,
            "preprocessing_timestamp": current_datetime_str
        }
        filtered_post = FilteredPreprocessedPostModel(**filtered_post_result)
        filtered_posts.append(filtered_post)

    return filtered_posts
```

3. Write the data to the database
```python
def batch_create_filtered_posts(
    posts: list[FilteredPreprocessedPostModel]
) -> None:
    """Batch create filtered posts in chunks.

    Uses peewee's chunking functionality to write posts in chunks.
    """
    with db.atomic():
        for idx in range(0, len(posts), DEFAULT_BATCH_WRITE_SIZE):
            FilteredPreprocessedPosts.insert_many(
                posts[idx:idx + DEFAULT_BATCH_WRITE_SIZE]
            ).on_conflict_ignore().execute()
    print(f"Batch created {len(posts)} posts.")
```

## Create a pipeline to run the data preprocessing and filtering steps.
Now that we have these steps, we can create a pipeline that, on trigger, runs the preprocessing and filtering steps.

```python
"""Pipeline for running preprocessing steps.

Loads the latest data for preprocessing and runs the preprocessing steps.

Run via `typer`: https://pypi.org/project/typer/

Example usage:
>>> python main.py
"""
import sys
import typer

from lib.log.logger import Logger
from services.preprocess_raw_data.helper import preprocess_raw_data

logger = Logger(__name__)

def main():
    try:
        logger.info("Starting preprocessing pipeline.")
        preprocess_raw_data()
        logger.info("Completed preprocessing pipeline.")
    except Exception as e:
        logger.error(f"Error in preprocessing pipeline: {e}")
        sys.exit(1)

if __name__ == "__main__":
    typer.run(main)
```

## Run the pipeline on a cron job on the compute cluster.
