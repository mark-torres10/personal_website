---
layout: single
title:  "Creating custom feeds in Bluesky: a data scientist's experience"
date:   2023-11-29 15:00:00 +0800
---
Currently, as part of my research, I am investigating the possibility of manipulating
people's social media algorithms and measuring their perceptions of political events around
them. Unlike most social media platforms, who have their own proprietary feed algorithms
designed to maximize engagement, [Bluesky](https://bsky.app/) allows developers to actually
create their own custom feeds and then users can subscribe to these feeds instead of using
a default one provided by Bluesky. The Bluesky team provides a [tutorial](https://github.com/bluesky-social/feed-generator#overview) on how to do this and there have been a [few community examples](https://atproto.com/community/projects) posted as well. I've never worked with a distributed decentralized application before, besides
a few simple Web3 projects, and I have very little actual web development experience, so I knew
that I had a lot to learn (and a lot to use ChatGPT for). But I think that the ability to break
down a software problem is largely the same across different subfields of tech, so with that in mind
I started figuring out how to solve this problem of creating a custom feed in Bluesky.

## Figuring out how to approach this problem

When starting this problem, I wanted to figure out what the end result would be and then to
work backwards from there. After looking at a few [example custom feeds](https://atproto.com/community/projects),
I learned that when you create a "custom feed", it is attached to your profile and others can also subscribe to it
as well. From looking at the Bluesky team's [documentation](https://github.com/bluesky-social/feed-generator#overview),
I learned that, in short, what happens when a user subscribes to your feed is:

1. They click the "feeds" tab and see the feed that they subscribed to.
2. When they click this feed, Bluesky makes a request to the feed's host server.
3. Bluesky expects a skeleton feed, which contains the primary identifiers of the
posts to put in the feed.
4. Bluesky hydrates those posts for the user.

The author of a feed can make multiple custom feeds, so when the Bluesky client sends a request
to fetch a certain feed, the endpoint contains information about the feed author as well as the
feed type itself. It's up to the host server to (a) get the possible posts to use and (b)
filter those posts and figure out which ones to return to the Bluesky client. According to
the documentation, the flow, on the host side is:

1. Get all posts.
2. Figure out which posts to save into a database (by default, the template code
saves posts into memory, but for persistence it's better to use a database).
3. Upon receiving a request, figure out which of the posts saved in the database
will be returned as a response.

This gives us two clear problems to figure out first:

1. How do we get the posts in the first place?
2. How do we filter the posts and figure out which to include?

### How do we get the posts in the first place?

Firehose...

### How do we filter the posts and figure out which to include?

## First attempt: building a server using Python and Flask

The Bluesky team's original code was built in Typescript, but there was a Python version
that was available as well.

Lessons:

- Use an official, maintained version of a package, if available.
- If an API is brand-new, use the SDK that is most actively supported by the core
developers, and use that one specifically.
- If there are notes like "we don't guarantee backwards compatibility", "THINGS CAN BREAK",
and "TODO" interspersed throughout both the code and the documentation, tread carefully.
- Use SDK abstractions where necessary and beneficial, but also don't be afraid to make
requests to the API directly (especially something as standard as a REST API).
- If possible, lean towards the language that you know best, where possible, but know when
to break that principle. I generally choose to work in Python because it's the language
that I know the best and because 99% of things that I would work on, I could do in Python. But,
in this scenario, the Python package that I assume would work, in fact didn't work, and instead
of fixing those bugs in the Python package and learning the inner details of decentralized software,
I can use the supported SDK instead, even if I don't know the language (to be fair, I already knew
Javascript, so Typescript wasn't a huge leap, plus for any code that I didn't want to spend over 2 minutes
understanding, I just asked ChatGPT to translate it into Python).

## Second attempt: rebuilding the server using Typescript

Success! It works (locally). This is the classic experience of "it works on
my computer" that every programmer has.

### Next steps

## Setting up a server on Digital Ocean

## Putting it all together

foo bar
