---
layout: single
title: "What's the strategy pattern in programming and why do we use it?"
date: 2025-12-03 18:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /ai_workflows/2025-12-03-strategy-pattern
---

# What's the strategy pattern in programming and why do we use it?

I've been working on intentionally integrating more design principles into my code. One of them is the `Strategy` pattern. I'll go over what this is and why we use it.

## What is the Strategy pattern?

A strategy encapsulates a family of algorithms behind a common interface, and let the caller (or configuration) supply which algorithm to use.

The Strategy Pattern always has:

1. A Strategy interface (shared contract). This can be implemented multiple ways, depending on the language. In Python, this is often an abstract base class (ABC) or a protocol.
2. Multiple Concrete Strategies
3. A Context that calls the strategy without caring which one it is

## What anti-patterns does it solve?

The Strategy pattern helps us solve a few anti-patterns common in code (and often when paired with the [Registry pattern](https://markptorres.com/ai_workflows/2025-12-02-registry-pattern)). We build off the filtering example that we discussed in the [Registry pattern](https://markptorres.com/ai_workflows/2025-12-02-registry-pattern) because these are commonly used together.

### Anti-pattern 1: Giant if/elif dispatch (“Stringly Typed Factories”)

We've all seen code that looks something like this:

```python
if filter_type == "has_url": ...
elif filter_type == "long_text": ...
elif filter_type == "english_only": ...
elif filter_type == "toxicity_openai": ...
elif filter_type == "toxicity_hf": ...
elif filter_type == "toxicity_fasttext": ...
```

#### Anti-pattern 1: Why is this bad?

1. Every new behavior requires editing this function.
2. Logic is scattered. Normally, there's a few of these giant if/elif code chunks across the repo.
3. Easy to break things when you add a new path.
4. Impossible to add lint or type checking. No IDE autocompletion.
5. Difficult for others to maintain or collaborate on this code.
6. Can't "list available filters".
7. Can't validate configs until something breaks at runtime.

#### Anti-pattern 1: How does the Strategy pattern fix this?

1. Any new behavior is defined as a new class/function and a decorator. This means that code changes are additive and don't touch existing code.
2. Core dispatch logic (the "if/elif/else" part) doesn't change. No more adding "elif" branches.
3. Implementations are declared in one place, so they can be discovered programatically (e.g., via IDE autocompletion) and are easy to build on top of and maintain.

### Anti-pattern 2: Tight coupling, hidden dependency webs, and tangled modules

Imagine an orchestrator `main.py` file with the following:

```python
def process(posts):
    if strategy == "openai_embed":
        # call OpenAI embedding logic here
    elif strategy == "hf_embed":
        # HuggingFace embedding details here
```

#### Anti-pattern 2: Why is this bad?

This means that the orchestration file (e.g., your `main.py` that runs everything) has to know the algorithm details. Changes to algorithmic implementation can cause the orchestration to fail, when those two should be decoupled. Your orchestration becomes a god function and you can't reuse the algorithm independently (e.g., in tests, utilities). This also makes refactors extremely painful, as, for example, moving embedders into their own modules would break imports in the orchestration code.

When filtering logic lives everywhere, you end up, for example, importing `filters.py` into `main.py`, but `main.py` imports something that imports `filters.py` again.

#### Anti-pattern 2: How does the Strategy pattern fix this?

The Strategy pattern helps us to implement dependency inversion:

```python
embedder = make_embedder(config.embedder)
vectors = embedder.embed(texts)
```

This way, within the orchestration code there is zero knowledge of how embedding happens. High-level code depends on abstractions, not implementations.

### Anti-pattern 3: Incomplete partial abstractions

Partial abstractions can happen when we try to abstract something by wrapping it in a function, but we continually add conditionals for special cases. This is a very common anti-pattern across codebases, as it's often easier to just add a one-off `if` block for a special case than to redesign a module, but to the extreme this can lead to a lot of tech debt. This isn't an antipattern that Strategy patterns uniquely solve, as it's a broad antipattern, but some of them can indeed be resolved with a Strategy pattern

Here's an example:

```python
def apply_filter(posts, filter_name):
    if filter_name == "has_url":
        return filter_has_url(posts)
    elif filter_name == "english_only":
        return filter_english(posts)
    return posts
```

#### Anti-pattern 3: Why is this bad?

This seems fine at first blush. But as it grows, you can run into a few problems:

- You need parameters for a new filter, so you hack them into the signature.
- Some filters are stateful, so you end up having to cram state into global variables

Soon you get something like this:

```python
def apply_filter(posts, filter_name, context, *args, **kwargs):
    if filter_name == "has_url":
        return filter_has_url(posts)
    elif filter_name == "english_only":
        return filter_english(posts)
    elif context["do_something_flag"]:
        # <do something special>
        ...
    elif context["lang"] == "es":
        # <do filter logic for Spanish-only>
        ...
    return posts
```

#### Anti-pattern 3: How does the Strategy pattern fix this?

With the Strategy pattern, the abstraction actually encapsulates the behavior:

```python
class LongTextFilter(PostFilterStrategy):
    def __init__(self, min_length=280):
        self.min_length = min_length

    def apply(self, posts):
        return [p for p in posts if len(p.text) > self.min_length]
```

### Anti-pattern 4: No discoverability or configurability

#### Anti-pattern 4: Why is this bad?

When dispatch logic is scattered or buried in code, you cannot reliably:

- List available filters, transformers, features, embedders, agents
- Validate configs
- Generate documentation from code
- Auto-load plugins
- Build dynamic UIs

This makes feature development more difficult (since it's harder to know all the possible branches in the dispatch logic), complicates onboarding (more places to look at to understand logical flow) and doesn't allow IDE autocompletion and static type checking.

#### Anti-pattern 4: How does the Strategy pattern fix this?

The Strategy pattern (in conjunction with the Registry pattern) solves this:

```python
>>> FILTER_REGISTRY.keys()
["has_url", "long_text", "english_only"]
```

This gives us:

- CLI help
- config validation
- auto-generated docs
- plugin systems
- dynamic pipelines

Many big frameworks (HuggingFace, Ray, PyTorch Lightning, LangChain) rely on registries for exactly this reason.

### Anti-pattern 5: Difficult-to-extend code

This is when third-party contributors or internal teammates need to touch centralized code to add new behavior.

#### Anti-pattern 5: Why is this bad?

A typical messy workflow when adding to an ML codebase without design patterns would look something like this:

- New feature encoder? Modify a big file.
- New agent behavior? Edit a shared dispatcher.
- New post-classifier? Duplicate boilerplate.

#### Anti-pattern 5: How does the Strategy pattern fix this?

The Strategy pattern turns the architecture into a plugin-based one:

```python
@register_filter("toxicity_llama")
class ToxicityLLamaFilter(PostFilterStrategy):
    ...
```

## A worked-out example

Here, we'll use a Strategy + Registry pattern, as they're often used together.

### Without a Strategy pattern

Let's revisit our filtering example.

```python
def filter_posts(posts, filter_type: str):
    if filter_type == "has_url":
        return [p for p in posts if "http" in p.text]

    elif filter_type == "long_text":
        return [p for p in posts if len(p.text) > 280]

    elif filter_type == "english_only":
        return [p for p in posts if p.lang == "en"]

    else:
        raise ValueError(f"Unknown filter type: {filter_type}")
```

Some problems that we've discussed that come up for something like this include:

- Adding a filter requires editing this function.
- You’ll end up with similar `if filter_name == ...` logic in multiple places (metrics, feature extractors, etc.).
- Can’t easily ask “what filters exist?” in a structured way.

### With a Strategy pattern

Recall that a Strategy pattern has 3 parts:

1. A Strategy interface (shared contract). This can be implemented multiple ways, depending on the language. In Python, this is often an abstract base class (ABC) or a protocol.
2. Multiple Concrete Strategies
3. A Context that calls the strategy without caring which one it is

#### 1. A Strategy Interface

```python
from abc import ABC, abstractmethod
from typing import List, Iterable

class PostFilterStrategy(ABC):
    @abstractmethod
    def apply(self, posts: Iterable[Post]) -> List[Post]:
        ...
```

#### 2. Multiple Concrete Strategies

```python
class HasURLFilter(PostFilterStrategy):
    def apply(self, posts):
        return [p for p in posts if "http" in p.text]

class LongTextFilter(PostFilterStrategy):
    def apply(self, posts):
        return [p for p in posts if len(p.text) > 280]

class EnglishOnlyFilter(PostFilterStrategy):
    def apply(self, posts):
        return [p for p in posts if p.lang == "en"]
```

#### 3. Context that calls the strategy

```python
class PostFilterer:
    def __init__(self, strategy: PostFilterStrategy):
        self.strategy = strategy

    def run(self, posts: list[Post]) -> list[Post]:
        return self.strategy.apply(posts)
```

#### Putting it to use

We can see these parts all coming into play:

```python
posts = [
    Post("Check this out http://example.com", lang="en"),
    Post("no url here", lang="es"),
    Post("Long text " * 100, lang="en"),
]

filterer = PostFilterer(strategy=HasURLFilter())
result = filterer.run(posts)

print([p.text for p in result])
```

We can swap strategies at runtime without changing the pipeline.

```python
filterer.strategy = EnglishOnlyFilter()
english_only = filterer.run(posts)

filterer.strategy = LongTextFilter()
long_text = filterer.run(posts)
```

## How this pertains to current work

I recently encountered this in [this PR](https://github.com/METResearchGroup/bluesky-research/pull/242) and in [this PR](https://github.com/METResearchGroup/bluesky-research/issues/231) for refactoring some research code. I had different functionalities per record type, and I used the strategy pattern in conjunction with the registry, protocol, and dependency injection patterns to help migrate the code to one that uses clean code principles.
