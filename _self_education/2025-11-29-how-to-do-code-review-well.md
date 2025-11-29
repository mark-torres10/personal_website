---
layout: single
title: "How to do code reviews well"
date: 2025-11-29 18:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /ai_workflows/2025-11-29-how-to-do-code-review-well
---

# How to do code reviews well

As I'm growing in my career, I want to make sure that I create good habits and have strong skills to further my growth. I'm a part of [Taro](https://www.jointaro.com/), an organization for career guidance in software engineering (they're really awesome, I highly recommend!!), and I'm going through the resources to make sure that I've got the fundamentals down solid. This is especially important for me since because I work in a nontraditional field (academia), and so I still want to make sure that I've nailed the best-practices in software engineering and that my career as a practitioner still advances and grows, even as my work is primarily academic and it's easier for my career to gravitate naturally towards academia.

What I just watched was Taro's course on [code review](https://www.jointaro.com/course/how-to-make-code-review-amazing/) and below are my notes and learnings from it.

## What is code review for?

I was surprised to learn that code review isn't really for catching bugs. When I was a junior data scientist, that was the focus of code reviews, and because I was at a startup, I don't think there was as much time or bandwidth put on mentorship as was probably helpful. I was under the perception that you do code review to catch bugs.

But, through this course, I learned that code review isn't really for catching bugs. You're supposed to catch bugs during your self-review, and at the end of the day, you're responsible for catching your own bugs.  Over time, this has become an implicit assumption for my own code, but I suppose I never formally understood that until now.

Instead, the point of code review is to:

1. **Increase code quality**: doing code review makes sure that the quality of the code (patterns/anti-patterns, code style, etc) is high. You can have code that is bug-free today but is poorly refactorted, has high cognitive load, or might cause bugs later on. This also matches what I'm currently reading about now in [A Philosophy of Software Design](https://www.amazon.com/Philosophy-Software-Design-John-Ousterhout/dp/1732102201).
2. **Increases awareness**: by doing code reviews with people, you (and reviewers) become more aware about what others are working on.
3. **Foster discussion and feedback**: code review lets you discuss architectural decisions, design principles, implementation details, and lets you learn from other people.

Coding is a team sport. Admittedly I often forget this because academia is notoriously siloed. But this course from Taro is a nice reminder of that fact. In industry, you develop software together, so code review is one great way to make sure that we're all on the same page about the software that we're building.

## What should/shouldn't be in a code review?

### SHOULD: code quality, testing, future-proofing, and knowledge transfer

Code review should cover things like:

- Code quality: things like naming conventions (since people will be reading this code later on), input assumptions (a function may not be used in a certain way now, but how do you handle adversarial inputs?) and logic validation. These are all things that improve the code quality itself without being overly nitpicky.
- Validating testing strategies
- Future-proofing: code review can be a place to discuss generalizing or modularizing an implementation in anticipation of certain features being added in the future.
- Shared learning: code reviews let the team accumulate wisdom and share knowledge with each other.

### SHOULD NOT: Desired behavior, module responsibility, and architecture

Concerns about functionality, modularity, and higher-level architecture should've been pinned down already in other discussion meetings, rather than at the point of code review.

### SHOULD NOT: nitpicks

Things like spaces and formatting and commit message practices should be enforced with a linter. The team should also preemptively agree on style conventions and formatting checks.
