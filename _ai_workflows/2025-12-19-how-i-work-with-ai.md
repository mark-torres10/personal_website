---
layout: single
title: "Lessons from a year of shipping code with AI agents"
date: 2025-12-19 13:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-12-19-how-i-work-with-ai
---

# Lessons from a year of shipping code with AI agents

Earlier this year, as part of my excitement at the capabilities of coding agents, I would let AI propose code and I would blindly ship it without oversight. In practice, this led to more headaches, as I would spend so much more time refactoring the AI-generated code, deleting lots of lines of duplicated or malformed logic, and asking (and re-asking) the AI to do things in different ways. It would've been faster if I had done these tasks by hand.

I've been embracing a different collaboration model with AI now. This workflow is now more methodical, with me at the center carefully guiding and inspecting the work of AI agents. It seems like many seasoned AI-native developrs are encountering the same hard lessons as me, and my writing this is motivated by [this Substack](https://substack.com/home/post/p-181957927) as I've found that their tips resonate with lessons I've also had to learn along the way. The Substack sums up better than I could the sorts of experiences I've had with using coding agents, and I added a few quick notes below about my own personal experiences.

## Change your mindset about AI-driven development

- "Vibe-coding" doesn't work when your code needs to ship
  - Vibe-coding is great until you've got 5,000 of Claude-generated slop that breaks as soon as you need to add a new feature. I've tried vibe-coding plenty of apps, and I've found that they're great for showing a client or end user a demo or PoC, but the codebase is basically unusable once I get stakeholder approval and then need to actually ship a working product.
- LLMs love writing code *a little too much*
  - LLMs love writing code. They also love spitting out lots of plausible-looking code that in reality fails very quickly. It writes code with complete conviction, including bugs or nonsense, and it won't tell you that something is wrong unless you tell it first (it may also tell you that something is wrong even if it's not actually wrong).
- Treat an LLM as both an encyclopedic coding wizard and also as an overly ambitious junior developer
  - LLMs know A LOT of things. I learn from Cursor every day about new patterns, APIs, languages, and ways of doing things. That being said, LLMs are also overly ambitious and apt to write more lines of code than are needed to solve a problem. I've even found that they sometimes try to solve problems that don't exist.
- **AI tools amplify your expertise** (for better or for worse)
  - The leverage you get from AI coding agents is in proportion to the quality of your own software engineering fundamentals. If you have strong software engineering fundamentals and discipline (e.g., TDD, clean code, good architecture design), then coding agents allow you to focus on all of those rather than the minutae of implementation. The kind of skills that make you a good senior engineer will help you leverage coding agents even more.
  - In contrast, having a poor foundation will give you a false sense of confidence when LLMs ship plausible-looking code.

## On working with coding agents

- Plan, plan, and plan some more.
  - My workflow generally consists of spending 10-15 minutes iterating on a `spec.md`, which includes not only a proposed implementation, but 2-3 alternate ways of doing the same task and pros/cons of each. This `spec.md` is also used to generate a `todo.md` file that Cursor can check off as it implements the tasks (the tasks in the `todo.md` are also designed to be small enough in scope to be 1 Git commit). This is similar to how Google's Antigravity IDE scopes its work ([I write more about that here](https://markptorres.com/ai_workflows/2025-11-19-first-impressions-antigravity)).
- Commit early and often. Also ask Cursor to set up the tasks in small, piecemeal ways that are the size of individual commits.
- Use git worktrees.

## On steering and enforcing code quality

- Have opinionated prompts about code style, testing specs. I have opinionated prompts about [tech stack](https://github.com/mark-torres10/ai_tools/blob/main/agents/task_instructions/engineering/PREFERRED_TECH_STACK.md), [how to debug](https://github.com/mark-torres10/ai_tools/blob/main/agents/task_instructions/project_management/HOW_TO_DEBUG.md), [how to do unit testing](https://github.com/mark-torres10/ai_tools/blob/main/agents/task_instructions/rules/UNIT_TESTING_STANDARDS.md), and [how to set up a dev environment](https://github.com/mark-torres10/ai_tools/blob/main/agents/task_instructions/rules/CODING_REPO_CONVENTIONS.md).
- Use CodeRabbit, CI/CD, static analysis, linters, and other tools. The more you leverage these tools, the better you guardrail your coding agents from shipping nonsense code.
- Having an AI code reviewer review AI code seems strange, but it's actually been a massive productivity gain. But don't rely on them blindly; I often find errors or poorly designed interfaces that even the strictest settings of CodeRabbit don't catch.
- Be VERY careful about keeping context windows cleaned. Don't be afraid to start a new agent thread. The cleaner the context window, the better the coding agent performs.

## On understanding AI-generatd code

- Don't commit code that you don't understand.
- Comb through the PRs yourself in Github.
- Even if Cursor suggests a code completion, write out the code yourself. By having to type it up yourself, you end up having to think about it more.
- Put Cursor in "Ask" mode by default unless the solution is trivial or you already know exactly what the code should look like (in which case, "Agent" mode is OK)."
- Ask the AI to give you 2-3 ways of solving a problem and to describe the pros and cons of each.
- Reiterate to the AI your own understanding of the code, and then ask it to see where your understanding is right/wrong/incomplete.

## On maximizing coding agent performance

- Make use of design patterns and other well-established ways of writing code.
- Similar to the last point: use tried-and-true code that has a lot of documentation and examples online, as opposed to newer APIs that LLMs may be less familiar with. LLMs know React better than they do Svelte.
- Tell the LLM what NOT to do and what you previously tried but failed, so as to nudge it to consider better options.
