---
layout: single
title: "What's the registry pattern in programming and why do we use it?"
date: 2025-12-02 15:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /ai_workflows/2025-12-02-registry-pattern
---

# What's the registry pattern in programming and why do we use it?

I've been working on intentionally integrating more design principles into my code. One of them is the `Registry` pattern. I'll go over what this is and why we use it.

## What is the Registry pattern?

A registry is a mapping of a key to a specific implementation of something.

I "register" the implementation (e.g., the function to do something) into a registry. Then, at runtime, I can go to the registry and grab/use the implementation that I want.

It looks something like this:

```python
REGISTRY: dict[str, type[BaseThing]] = {}

def register(name: str):
    def decorator(cls: type[BaseThing]):
        REGISTRY[name] = cls
        return cls
    return decorator
```

Then everywhere else, you do:

```python
custom_func = REGISTRY[name_from_config]
res = custom_func(...)
```

## What anti-pattern does it solve?

The classic anti-pattern that this solves is the giant (and often duplicated) if/elif/else dispatches scattered everywhere.

This can often happen if you, for example, incorporate a new algorithm or feature, and you want to include its functionality. Eventually, this grows into a huge and unwieldy if/elif/else chain.

Some problems that come up are:

- The list of "what exists" is not obvious or discoverable. You can define different if/elif/else chains that all do slightly different things.
- New behavior requires editing a lot of call sites. This means a lot of duplicated changes and the possibility of messing up a migration.
- Without a registry, you often get ad-hoc maps and if/else logics in random modules.

A registry centralizes that: “If you want to know all supported X, look here.”

## A worked-out example

### Without a registry

Let's say we're working with text data, say from a social media application. You can imagine having different filters for cleaning the data:

```python
def filter_posts(posts, filter_name):
    if filter_name == "has_url":
        return [p for p in posts if "http" in p.text]
    elif filter_name == "long_text":
        return [p for p in posts if len(p.text) > 280]
    elif filter_name == "english_only":
        return [p for p in posts if p.lang == "en"]
    # ... and so on ...
    else:
        raise ValueError(f"Unknown filter: {filter_name}")

```

What are some problems with this?

- Adding a filter requires editing this function. This in turn means that anything requiring this function to work downstream can be affected.
- Lots of repeated `if filter_name == ...` logic in multiple places.
- Can't easily ask the question of "what filters exist?" (because it's hardcoded in the code AND because different places can have their own `if filter_name == ...`).

### With a registry

We can avoid these problems if we use a registry pattern:

```python
from typing import Protocol, Iterable

class Post:
    def __init__(self, text: str, lang: str = "en"):
        self.text = text
        self.lang = lang


class PostFilter(Protocol):
    def __call__(self, posts: Iterable[Post]) -> list[Post]:
        ...


FILTER_REGISTRY: dict[str, PostFilter] = {}


def register_filter(name: str):
    def decorator(fn: PostFilter):
        FILTER_REGISTRY[name] = fn
        return fn
    return decorator


@register_filter("has_url")
def has_url(posts: Iterable[Post]) -> list[Post]:
    return [p for p in posts if "http" in p.text]


@register_filter("long_text")
def long_text(posts: Iterable[Post]) -> list[Post]:
    return [p for p in posts if len(p.text) > 280]


@register_filter("english_only")
def english_only(posts: Iterable[Post]) -> list[Post]:
    return [p for p in posts if p.lang == "en"]


def filter_posts(posts: list[Post], filter_name: str) -> list[Post]:
    try:
        fn = FILTER_REGISTRY[filter_name]
    except KeyError:
        raise ValueError(f"Unknown filter: {filter_name}. "
                         f"Available: {list(FILTER_REGISTRY)}")
    return fn(posts)
```

Some benefits that we get from this include:

- Adding a filter = define function + decorator; no central if/elif.
- You can introspect FILTER_REGISTRY in tests, docs, or a CLI.
- You can serialize filter_name in config / DB and resolve it centrally.

The registry becomes the central place to define all the filters that are available globally. Different parts of the code can use the filters that they need. Meanwhile, it's easy to (1) see all the filters that are available, (2) change existing filters without breaking other code, and (3) add new filters.

## How this pertains to current work

I recently encountered this in [this PR](https://github.com/METResearchGroup/bluesky-research/pull/242) for refactoring some research code. This was the classic antipattern that a registry is designed to solve: I had, in a variety of places, strewn hardcoded if/else blocks for different functionalities related to record types. A dispatch pattern, plus other design patterns, was critical for migrating this code to use clean code architecture.
