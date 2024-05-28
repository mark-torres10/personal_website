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

2. Filter the data
3. Preprocess the data
4. Write the data

### Load the data

### Filter the data

## Create a pipeline to run the data preprocessing and filtering steps.

## Run the pipeline on a cron job on the compute cluster.


