---
layout: single
title:  "Experiments with LLM classification for political content (Part VIII)"
date:   2024-05-21 11:00:00 +0800
classes: wide
toc: true
categories:
- research
- all_posts
permalink: /research/llm-experiments-pt-viii
---
# Using LLMs to classify if social media posts are political or not (Part VIII).
I'm working on a project that involves gathering social media posts from [Bluesky](https://bsky.app/) and analyzing them. Part of that project requires knowing which posts are about political or social topics, and if so, what political side they support. Current ML classifiers don't work that well out of the box, so I'm trying to create our own classification scheme using LLMs. I'm trying to use LLMs in order to classify [Bluesky](https://bsky.app/) posts as either having political content or not, and if so, the political ideology, and I've found that LLMs work quite well for this task. I've used Llama3-8b and Llama3-70b via [Groq](https://groq.com/) so far, but are also open to experimenting with other open-source models as well (I have the on-prem infrastructure to host our own models, which is much cheaper at scale).

I've confirmed [previously](https://markptorres.com/research/llm-experiments-pt-i) that LLMs are promising for our classification task. I also [cleaned up](https://markptorres.com/research/llm-experiments-pt-v) the code to have a more robust ETL pipeline. Now, I want to move everything over to the [Kellogg Linux Cluster (KLC)](https://www.kellogg.northwestern.edu/academics-research/research-support/computing/kellogg-linux-cluster.aspx), which will serve as a way for us to get the at-scale computing needs required for the project.

Specifically, I'll do the following
- Set up KLC access
- Create a cron job for running daily pipeline syncs of Bluesky posts

## Set up KLC access
First, I need to start by setting up KLC access. Luckily, this process is pretty well-documented, so what I'm doing is based primarily on [this](https://www.kellogg.northwestern.edu/academics-research/research-support/computing/kellogg-linux-cluster.aspx) resource. I also need to do this while on the university VPN, with instructions [here](https://services.northwestern.edu/TDClient/30/Portal/KB/ArticleDet?ID=1818).

After I've set up VPN access, I can ssh into the cluster:
```bash
ssh <netid>@quest.northwestern.edu
```

Once this was set up, I'm directed to my personal storage on the cluster. Now I need to find my project's folder, since the docs talk about how the space allocated to each user's personal folder is quite small (a few GBs) but the space allocated to a project folder can be quite large (in our case, our project was allocated 1TB). [This](https://services.northwestern.edu/TDClient/30/Portal/KB/ArticleDet?ID=2234#file-explorer) document has details on how to do this process (as well as how to request project allocation if necessary).

Now I'll load conda. I'll follow [these](https://services.northwestern.edu/TDClient/30/Portal/KB/ArticleDet?ID=2064) docs as well as play around with using `mamba` for loading conda, since `mamba` is, according to blogs like [this](https://focalplane.biologists.com/2022/12/08/managing-scientific-python-environments-using-conda-mamba-and-friends/), more efficient at package management than conda, but lets me use both conda and pip. So far, this is how that looks:
![KLC setup during conda install](/assets/images/2024-05-21-llm-experiments-viii/klc-setup-conda.png)

Now I need to set up Git. This wasn't too bad, [here](https://services.northwestern.edu/TDClient/30/Portal/KB/ArticleDet?ID=1668) are resources on how to do this on KLC. I found authentication to be easiest once I could use the Github CLI, and installing it was straightforward via [conda](https://github.com/cli/cli?tab=readme-ov-file#conda). I verified that a file that I wrote and uploaded on the cluster was successfully pushed to my repo.

![Verified that KLC is connected to Github](/assets/images/2024-05-21-llm-experiments-viii/verify-github-connected.png)

I also set up SSH keys so that I don't have to keep passing in my password each time. That was actually pretty straightforward, and I was surprised to find that ChatGPT nailed the instructions.
![ChatGPT instructions for setting up SSH keys](/assets/images/2024-05-21-llm-experiments-viii/chatgpt-ssh-keys-instructions.png)

I then set up remote SSH tunneling with VSCode so that I could access the cluster via VSCode. I'll still likely do most development locally, but this makes it much easier to do any changes that are easier to do while accessing the remote cluster (such as, for example, accessing data stored on KLC).
![VSCode connected to KLC cluster via remote SSH tunneling](/assets/images/2024-05-21-llm-experiments-viii/vscode-remote-ssh-tunneling.png)

## Create a cron job for running daily syncs of Bluesky posts
Now that we've set up access, we can use KLC to do things that require more compute and memory.

### Setting up the data pipeline
The basic interface that I'll be using for running the data pipeline is this:
```python
"""Pipeline for getting posts from Bluesky.

Can either fetch raw posts from the firehose or from the "Most Liked" feed.

Run via `typer`: https://pypi.org/project/typer/
"""
import sys
from typing import Literal

import typer

from most_liked import get_posts as get_most_liked_posts
from firehose import get_posts as get_firehose_posts

def main(sync_type: Literal["firehose", "most_liked"]):
    if sync_type == "firehose":
        get_firehose_posts()
    elif sync_type == "most_liked":
        get_most_liked_posts()
    else:
        print("Invalid sync type.")
        sys.exit(1)

if __name__ == "__main__":
    typer.run(main)
```

There are two types of posts that we want to sync:
1. The firehose posts: these are posts from the Bluesky firehose, which constantly updates whenever a new post is written on Bluesky. We'll be able to get all the posts from Bluesky as they're written in real time.
2. The most liked posts: these are the posts that have been the most liked in the past 24 hours. According to the [docs](https://github.com/skyfeed-dev/feed-generator/blob/main/lib/queries.dart#L30), the "catch-up" feed (the most-liked posts the past 24 hours) consists of the top 1,000 posts with the most likes in the past 24 hours. This feed appears to be consistently refreshed to have the latest most liked posts in the 24 hour period starting with whenever the feed is fetched, ensuring that if we collect posts 24 hours apart, we'll have completely different posts in our feed.

What I want to do is:
1. For the firehose posts, pick X hours out of the day (maybe something like 4?) and run the firehose. While the firehose is running, we'll save all the new posts that are being written to Bluesky, in real time.
2. For the most liked posts, fetch the latest posts from the most liked posts in the past 24 hours.

#### How much data should we expect to sync per day?

For the most liked posts, we know that we can expect up to 1,000 posts per day. For the firehose posts, [this](https://bsky.jazco.dev/stats) dashboard, built by one of the core Bluesky developers, gives us baseline rates, in real time, of activity on Bluesky.

According to the following graphic from that dashboard, it looks like the past month, as per the writing of this document, has averaged ~800,000 posts per day.
![Number of posts per day on Bluesky](/assets/images/2024-05-21-llm-experiments-viii/dashboard-number-posts-per-day.png)

### Setting up the cron job
