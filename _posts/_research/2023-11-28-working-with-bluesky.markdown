---
layout: single
title:  "Working with Bluesky as a researcher"
date:   2023-11-28 15:00:00 +0800
classes: wide
toc: true
categories:
- research
permalink: /research/2023-11-28-working-with-bsky
---
I'm currently doing research at Northwestern, studying how people engage with political content online on social media, especially how they relate to algorithmically curated content. We look at and answer questions such as how people become [more polarized](https://www.nature.com/articles/s41562-023-01550-8) or how they end up in [echo chambers](https://journals.sagepub.com/doi/pdf/10.1177/07439156221103852). A lot of research in this topic is necessarily reactive and correlational - people get a feed of posts, they comment, like, react, etc., and we look at their actions after the fact. There are ways to perform causal inference through experiments, such as recruiting participants to [participate in controlled experiments](https://www.nature.com/articles/s41598-022-23673-0) or [artificially manipulate the recommendation algorithms](https://arxiv.org/abs/2203.10666) of social media platforms. Some research teams at larger tech firms study these phenomena themselves - for example, Twitter studied [algorithmic amplification of political information](https://cdn.cms-twdigitalassets.com/content/dam/blog-twitter/official/en_us/company/2021/rml/Algorithmic-Amplification-of-Politics-on-Twitter.pdf) on their platform, finding that right-wing news experiences higher amplification on social media than other news sources.

However, true causal inference is difficult because people cannot dictate what is in their feeds. These are all determined by proprietary algorithms for each social media platform. There are workarounds, as described before. But, in an ideal world, we could artificially modify the algorithms used to hydrate people's social media feeds in order to manipulate what information people are exposed to. This unlocks a whole new range of interesting scientific research as we would then be better able to manipulate the information on people's feeds and record their responses.

## Bluesky: an alternative to traditional social media

In the world of big-tech social media platforms, this is not possible. But, a relatively new social media platform, [Bluesky](https://bsky.app/) makes it possible. Bluesky, a distributed text-based microblogging platform (essentially Twitter with just text, which makes sense given that Bluesky is a Twitter pet project from the Jack Dorsey era), aims to provide a social media experience with user control and decentralization at its core.

### The AT Protocol

Bluesky is the first app build under the [AT Protocol](https://atproto.com/guides/overview), which dictates a series of guidelines and procedures for creating distributed social applications. [Here](https://educatedguesswork.org/posts/atproto-firstlook/), [here](https://parkerhiggins.net/2023/05/bluesky-atproto-url-usernames/) and [here](https://fedimeister.onyxbits.de/blog/bluesky-at-protocol-vs-activity-pub/) provide great overviews of the AT Protocol.

### Feed generators

For my research purposes, the important feature that the AT Protocol provides is that users can subscribe to [feed generators](https://github.com/bluesky-social/feed-generator), which are algorithms, created by other users, that can control what posts are returned to the users. Most social media apps have a default home feed created by their proprietary personalization algorithms, and at best they may also have a reverse chronological algorithm as well. However, Bluesky allows for any arbitrary feed algorithms; for example, in theory a user could subscribe to a "US sports" feed algorithm that only returns content relating to US sports.

### Getting started with feed generators

The Bluesky team has provided an example [Github repo](https://github.com/bluesky-social/feed-generator) for working with their feed generators (my language of choice is Python, and there is a convenient Python version [here](https://github.com/MarshalX/bluesky-feed-generator)).

From the Bluesky docs, here are the steps for how a user can get a custom feed:

1. The PDS of the user (the user's "personal server") sends a request to the URI of the feed.
2. The PDS resolves the URI and finds the DID ([decentralized identifiers](https://www.w3.org/TR/did-core/)) for the feed generator.
3. The PDS sends a `getFeedSkeleton` request to an endpoint defined for that feed generator.
4. The feed generator returns a skeleton of the feed to the user's PDS.
5. The PDS hydrates the feed (user info, post contents, aggregates, etc.).
6. The PDS returns the hydrated feed to the user.

As per the docs, to the end user this should feel like visiting a page in the app, but programatically the steps above are how custom feeds are given to the user.

## Using feed generators to study algorithm influence on social media engagement online

TODO: after getting actual specs of the project.
