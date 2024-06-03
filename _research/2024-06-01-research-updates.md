---
layout: single
title:  "Research updates, 2024-06-01"
date:   2024-06-01 13:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/research-updates-2024-06-01
---
# Research updates, 2024-06-01

Today, I worked on the following:

- Completed building a batch inference pipeline for classifications with the Perspective API.
- Started sociopolitical inference via LLMs.

## Completed building a batch inference pipeline for classifications with the Perspective API
I continued building the pipeline for classification with the Perspective API.

I finished the basic skeleton of the logic:
```python
def classify_latest_posts():
    # Load posts
    posts: list[FilteredPreprocessedPostModel] = load_posts_to_classify()
    print(f"Number of posts loaded for classification using Perspective API: {len(posts)}") # noqa

    # fetch metadata of posts to be classified. Insert the metadata to DBs
    # (if not already present)
    posts_to_classify: list[RecordClassificationMetadataModel] = (
        get_post_metadata_for_classification(posts)
    )
    batch_insert_metadata(posts_to_classify)

    # validate posts
    valid_posts, invalid_posts = validate_posts(posts_to_classify)
    print(f"Number of valid posts: {len(valid_posts)}")
    print(f"Number of invalid posts: {len(invalid_posts)}")
    print(f"Classifying {len(valid_posts)} posts via Perspective API...")
    print(f"Defaulting {len(invalid_posts)} posts to failed label...")

    # insert invalid posts into DB first, before running Perspective API
    # classification
    invalid_posts_models = []
    for post in invalid_posts:
        invalid_posts_models.append(
            PerspectiveApiLabelsModel(
                uri=post["uri"],
                text=post["text"],
                was_successfully_labeled=False,
                reason="text_too_short",
                label_timestamp=current_datetime_str,
            )
        )
    print(f"Inserting {len(invalid_posts_models)} invalid posts into the DB.")
    batch_insert_perspective_api_labels(invalid_posts_models)
    print(f"Completed inserting {len(invalid_posts_models)} invalid posts into the DB.") # noqa

    # run inference on valid posts
    print(f"Running batch classification on {len(valid_posts)} valid posts.")
    run_batch_classification(posts=valid_posts)
    print("Completed batch classification.")
```

Now I need to test this logic first on the pilot data, and then run it in general.

I tried using the batch endpoint from the Google API and found out that it's an old endpoint, how lovely.

```plaintext
You have constructed a BatchHttpRequest using the legacy batch endpoint https://www.googleapis.com/batch. This endpoint will be turned down on August 12, 2020. Please provide the API-specific endpoint or use service.new_batch_http_request(). For more details see https://developers.googleblog.com/2018/03/discontinuing-support-for-json-rpc-and.htmland https://developers.google.com/api-client-library/python/guide/batch.
Traceback (most recent call last):
  File "/projects/p32375/bluesky-research/demos/2024-05-30-analyze-pilot-data/classify_pilot_data.py", line 124, in <module>
    main(num_samples=num_samples)
  File "/projects/p32375/bluesky-research/lib/helper.py", line 100, in wrapper
    result = func(*args, **kwargs)
  File "/projects/p32375/bluesky-research/demos/2024-05-30-analyze-pilot-data/classify_pilot_data.py", line 118, in main
    run_batch_classification(posts=valid_posts)
  File "/projects/p32375/bluesky-research/ml_tooling/perspective_api/model.py", line 324, in run_batch_classification
    loop.run_until_complete(
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/asyncio/base_events.py", line 649, in run_until_complete
    return future.result()
  File "/projects/p32375/bluesky-research/ml_tooling/perspective_api/model.py", line 307, in batch_classify_posts
    responses = await process_perspective_batch(request_batch)
  File "/projects/p32375/bluesky-research/ml_tooling/perspective_api/model.py", line 250, in process_perspective_batch
    batch.execute()
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/googleapiclient/_helpers.py", line 130, in positional_wrapper
    return wrapped(*args, **kwargs)
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/googleapiclient/http.py", line 1566, in execute
    self._execute(http, self._order, self._requests)
  File "/home/jya0297/.conda/envs/bluesky_research/lib/python3.10/site-packages/googleapiclient/http.py", line 1501, in _execute
    raise HttpError(resp, content, uri=self._batch_uri)
googleapiclient.errors.HttpError: <HttpError 404 when requesting https://www.googleapis.com/batch returned "Not Found". Details: "
<!DOCTYPE html>
<html lang=en>
  <meta charset=utf-8>
  <meta name=viewport content="initial-scale=1, minimum-scale=1, width=device-width">
  <title>Error 404 (Not Found)!!1</title>
  <style>
    *{margin:0;padding:0}html,code{font:15px/22px arial,sans-serif}html{background:#fff;color:#222;padding:15px}body{margin:7% auto 0;max-width:390px;min-height:180px;padding:30px 0 15px}* > body{background:url(//www.google.com/images/errors/robot.png) 100% 5px no-repeat;padding-right:205px}p{margin:11px 0 22px;overflow:hidden}ins{color:#777;text-decoration:none}a img{border:0}@media screen and (max-width:772px){body{background:none;margin-top:0;max-width:none;padding-right:0}}#logo{background:url(//www.google.com/images/logos/errorpage/error_logo-150x54.png) no-repeat;margin-left:-5px}@media only screen and (min-resolution:192dpi){#logo{background:url(//www.google.com/images/logos/errorpage/error_logo-150x54-2x.png) no-repeat 0% 0%/100% 100%;-moz-border-image:url(//www.google.com/images/logos/errorpage/error_logo-150x54-2x.png) 0}}@media only screen and (-webkit-min-device-pixel-ratio:2){#logo{background:url(//www.google.com/images/logos/errorpage/error_logo-150x54-2x.png) no-repeat;-webkit-background-size:100% 100%}}#logo{display:inline-block;height:54px;width:150px}
  </style>
  <a href=//www.google.com/><span id=logo aria-label=Google></span></a>
  <p><b>404.</b> <ins>That’s an error.</ins>
  <p>  <ins>That’s all we know.</ins>
"
```

I thought that the docs were up-to-date, but I suppose not. Also I guess that I can't rely on ChatGPT to necessarily also give up-to-date code, since the docs are still online even though the endpoint is deprecated.

I went to the link mentioned in the error message and it redirects to the Google Python client Github page, which also still has the same [page](https://github.com/googleapis/google-api-python-client/blob/main/docs/batch.md?plain=1#L2) that I saw before which mentions the (apparently) deprecated endpoint.

This was fixed by a simple change.
```python
batch = google_client.new_batch_http_request()
#batch = BatchHttpRequest()
```

That simple fix was the last bug required to get a working implementation. This is the result on a subset of ~500 posts from the pilot.
```python
(bluesky_research) [jya0297@quser33 2024-05-30-analyze-pilot-data]$ python classify_pilot_data.py
HTTP Request: POST https://bsky.social/xrpc/com.atproto.server.createSession "HTTP/1.1 200 OK"
HTTP Request: GET https://bsky.social/xrpc/app.bsky.actor.getProfile?actor=mindtechnologylab.bsky.social "HTTP/1.1 200 OK"
file_cache is only supported with oauth2client<4.0.0
Attempting to process 165163 pilot posts.
Attempting to insert 468 metadata objects into the database.
Number of deduped metadata objects to insert: 0
No metadata to insert.
Number of valid posts: 468
Number of invalid posts: 0
Classifying 468 posts via Perspective API...
Defaulting 0 posts to failed label...
Inserting 0 invalid posts into the DB.
Attempting to insert 0 labels into the database.
Number of deduped labels to insert: 0
No labels to insert.
Completed inserting 0 invalid posts into the DB.
Running batch classification on 468 valid posts.
Execution time for batch_classify_posts: 0 minutes, 1 seconds
Memory usage for batch_classify_posts: 0.0 MB
Attempting to insert 50 labels into the database.
Number of deduped labels to insert: 50
Labels count prior to insertion: 32
Finished inserting 50 labels into the database.
Labels count after insertion: 82
Attempting to insert 50 labels into the database.
Number of deduped labels to insert: 50
Labels count prior to insertion: 82
Finished inserting 50 labels into the database.
Labels count after insertion: 132
Attempting to insert 50 labels into the database.
Number of deduped labels to insert: 50
Labels count prior to insertion: 132
Finished inserting 50 labels into the database.
Labels count after insertion: 182
Attempting to insert 50 labels into the database.
Number of deduped labels to insert: 50
Labels count prior to insertion: 182
Finished inserting 50 labels into the database.
Labels count after insertion: 232
Attempting to insert 50 labels into the database.
Number of deduped labels to insert: 50
Labels count prior to insertion: 232
Finished inserting 50 labels into the database.
Labels count after insertion: 282
Attempting to insert 50 labels into the database.
Number of deduped labels to insert: 50
Labels count prior to insertion: 282
Finished inserting 50 labels into the database.
Labels count after insertion: 332
Attempting to insert 50 labels into the database.
Number of deduped labels to insert: 50
Labels count prior to insertion: 332
Finished inserting 50 labels into the database.
Labels count after insertion: 382
Attempting to insert 50 labels into the database.
Number of deduped labels to insert: 50
Labels count prior to insertion: 382
Finished inserting 50 labels into the database.
Labels count after insertion: 432
Attempting to insert 50 labels into the database.
Number of deduped labels to insert: 50
Labels count prior to insertion: 432
Finished inserting 50 labels into the database.
Labels count after insertion: 482
Attempting to insert 18 labels into the database.
Number of deduped labels to insert: 18
Labels count prior to insertion: 482
Finished inserting 18 labels into the database.
Labels count after insertion: 500
Completed batch classification.
Execution time for main: 0 minutes, 21 seconds
Memory usage for main: 207.01171875 MB
```

A basic look at the database confirms that we have valid labels:
```python
def get_perspective_api_labels() -> list[PerspectiveApiLabelsModel]:
    query = PerspectiveApiLabels.select()
    res = list(query)
    res_dicts: list[dict] = [r.__dict__['__data__'] for r in res]
    transformed_res: list[PerspectiveApiLabelsModel] = [
        PerspectiveApiLabelsModel(
            uri=r["uri"],
            text=r["text"],
            was_successfully_labeled=r["was_successfully_labeled"],
            reason=r["reason"],
            label_timestamp=r["label_timestamp"],
            prob_toxic=r["prob_toxic"],
            prob_severe_toxic=r["prob_severe_toxic"],
            prob_identity_attack=r["prob_identity_attack"],
            prob_insult=r["prob_insult"],
            prob_profanity=r["prob_profanity"],
            prob_threat=r["prob_threat"],
            prob_affinity=r["prob_affinity"],
            prob_compassion=r["prob_compassion"],
            prob_constructive=r["prob_constructive"],
            prob_curiosity=r["prob_curiosity"],
            prob_nuance=r["prob_nuance"],
            prob_personal_story=r["prob_personal_story"],
            prob_reasoning=r["prob_reasoning"],
            prob_respect=r["prob_respect"],
            prob_alienation=r["prob_alienation"],
            prob_fearmongering=r["prob_fearmongering"],
            prob_generalization=r["prob_generalization"],
            prob_moral_outrage=r["prob_moral_outrage"],
            prob_scapegoating=r["prob_scapegoating"],
            prob_sexually_explicit=r["prob_sexually_explicit"],
            prob_flirtation=r["prob_flirtation"],
            prob_spam=r["prob_spam"]
        )
        for r in res_dicts
    ]
    return transformed_res

if __name__ == "__main__":
    # create_initial_tables()
    metadata_count = RecordClassificationMetadata.select().count()
    perspective_api_labels_count = PerspectiveApiLabels.select().count()
    print(f"Metadata count: {metadata_count}")
    print(f"Perspective API labels count: {perspective_api_labels_count}")

    perspective_api_labels = get_perspective_api_labels()
    print(perspective_api_labels[0])
    print(perspective_api_labels[5])
```

```plaintext
Metadata count: 500
Perspective API labels count: 500
uri='at://did:plc:clmngjpnuxhgyfxy7tpuitfm/app.bsky.feed.post/3ktkyyfcefs2m' text='\U0001faf6 You x' was_successfully_labeled=False reason='text_too_short' label_timestamp='2024-06-01-06:59:52' prob_toxic=None prob_severe_toxic=None prob_identity_attack=None prob_insult=None prob_profanity=None prob_threat=None prob_affinity=None prob_compassion=None prob_constructive=None prob_curiosity=None prob_nuance=None prob_personal_story=None prob_reasoning=None prob_respect=None prob_alienation=None prob_fearmongering=None prob_generalization=None prob_moral_outrage=None prob_scapegoating=None prob_sexually_explicit=None prob_flirtation=None prob_spam=None
uri='at://did:plc:pxfdh7sharqirjgccn7vxqmi/app.bsky.feed.post/3ktkyyjch4y2x' text='' was_successfully_labeled=False reason='text_too_short' label_timestamp='2024-06-01-06:59:52' prob_toxic=None prob_severe_toxic=None prob_identity_attack=None prob_insult=None prob_profanity=None prob_threat=None prob_affinity=None prob_compassion=None prob_constructive=None prob_curiosity=None prob_nuance=None prob_personal_story=None prob_reasoning=None prob_respect=None prob_alienation=None prob_fearmongering=None prob_generalization=None prob_moral_outrage=None prob_scapegoating=None prob_sexually_explicit=None prob_flirtation=None prob_spam=None
```

For 10,000 posts, this is what I got for the results.
```plaintext
Labels count after insertion: 10000
Completed batch classification.
Execution time for main: 5 minutes, 13 seconds
Memory usage for main: 242.0859375 MB
```

For 160,000 posts (all the pilot posts), this is what I got for the results.
```plaintext
Labels count after insertion: 165163
Completed batch classification.
Execution time for main: 78 minutes, 22 seconds
Memory usage for main: 839.1796875 MB
```

Perspective API classification for the pilot posts is done! I'll next set up a cron job to do this over time with the latest batches of preprocessed posts.

## Started sociopolitical inference via LLMs
Next, I'll set up inference with LLMs for sociopolitical classification. I wanted to do this via

## Final thoughts
More and more scaled inference stuff today. I definitely am not an expert in this type of engineering and it's proving to be a challenging but intellectually rewarding experience. Definitely a useful skill to have. I think it's correct to pre-emptively invest in scale now, since I think that the scale needed to classify ~200,000 posts for the pilot is not too far off from the scale needed to classify 1M posts per day, so I might as well invest in trying to do this right, even if it takes more time.

I think that this week has actually been super productive. I’ve worked 39 hours, which is less than my usual 50 hours, but I (1) didn’t work Monday, since I was coming back from vacation, and (2) the hours that I’ve worked, I’ve been super focused and locked-in. I didn’t get distracted at all this week like I normally do. I’ve noticed that although it’s been good that I’ve been aiming to work many hours (~10 hours/day), that I’ve found myself cheating myself (e.g,. by taking excessively long breaks, by “watching lectures”, reading emails, or other things where I wasn’t really moving the ball forward but I could justify myself as “doing work” since “my mind was still on work stuff”). I did stuff in 7 hours that I typically would’ve counted for 10-11 hours, due to just being much more efficient and focusing on work that explicitly is moving the ball forward. Very efficient and productive work week thus far, and I’ve definitely gotten a lot of stuff done.
