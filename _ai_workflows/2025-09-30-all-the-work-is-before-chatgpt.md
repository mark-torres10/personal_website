---
layout: single
title: "The hard work comes before you start asking ChatGPT to write (code, papers, docs, etc.)"
date: 2025-09-30 05:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-09-30-all-the-work-is-before-chatgpt
---

# The hard work comes before you start asking ChatGPT to write (code, papers, docs, etc.)

Vibe coding promises to make it really easy to write an app. But there's fundamentally 2 things that are flawed with this premise:

1. Writing an app != having a viable business
2. It's really easy to write AI slop
3. Using coding agents well implies that you know what to ask it.

I'm incredibly bullish on "vibe-coding"-esque things in the long-term (I think that my own kids won't have to learn how to code like I do, just like I didn't have to learn COBOL like the previous generation of programmer).

Good software comes before you write any code:

- articulating why you're even doing this task in the first place.
- clearly define what you need and don't need to do
- knowing what success looks like
- define what tests need to look like before you write code

These are generic and common software principles that still hold true now, and are especially true with AI.

As I've learned, there are actually hundreds of ways to write the same app, and also hundreds of ways to write it wrong. The "No free lunch" theorem in machine learning, which says that there's no 1 universally superior algorithm for all possible problems, is true here more generally, as the "right way" to make an app depends on understanding needs, constraints, desired outcomes, and the like.

There's such a big learning curve with writing code that people think you've "hit the summit" when you've learned how to code. But in practice, learning code is like learning a new language: once you know how to make your own sentences, it becomes a matter of knowing what to say, how to say it, and when. Code is the same way.

What I do in my work is I try a first pass of the above questions, then I have various [AI personas](https://github.com/mark-torres10/ai_tools/tree/main/agents/personas) critique my work and give feedback. Each has their own expertise and each has their own checklist of criteria and success metrics and they drill my plan on them. Sometimes they bring up things that are unrelated to what I’m working on, and I ignore the advice, but often times they bring up valid points and push back on if what I’m suggesting is reasonable, if it’s under specified, etc. But, by and large, having this additional step in my workflow has been tremendously helpful. as it improves my ability to specify to the AI agents what I actually want it to accomplish and how I want it to accomplish it in the first place.

I think hard about how the data models look like, how different services interact with each other, and failure modes. I have the AI help expedite this process, by suggesting things, but the problem is getting it to conform to patterns present in my own codebase. An optimization that I want to eventually make is writing down design docs to describe exactly how my codebase works and the patterns present in the code written. Currently I have all this info in folder-specific README files that I use to help AI agents route their search. I'm experimenting with having AI agents write a "PATTERNS.md" to codify design patterns used in the codebase, as well as "ARCHITECTURE.md" to describe the filed and different services. AI models are also quite great at rubric-based and criteria-based development, so instead of asking it to "write good code" or "do analysis" or "write well", I find it helpful to say "write good code, and here's what I mean by that, and here's the criteria for it, and here's how I want it scored".

AI-driven development is a really hands-on process, and we've not arrived at (and I'm doubtful we fully will) a place where AI agents supplant critical thinking. But, AI models have made it significantly easier to do the parts of the work that don't require significant thinking, as long as you steer them in the right direction.

**Caveat: doesn't apply as much to demos, throwaway code, etc.
