---
layout: single
title: "How I'm working with AI to ship code"
date: 2025-12-05 13:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-12-19-how-i-work-with-ai
---

# How I'm working with AI to ship code

Earlier this year, as part of my excitement at the capabilities of coding agents, I would let AI propose code and I would blindly ship it without oversight. In practice, this led to more headaches, as I would spend so much more time refactoring the AI-generated code, deleting lots of lines of duplicated or malformed logic, and asking (and re-asking) the AI to do things in different ways. It would've been faster if I had done these tasks by hand.

I've been embracing a different collaboration model with AI now. It seems like many seasoned AI-native developrs are encountering the same hard lessons as me, and my writing this is motivated by [this Substack](https://substack.com/home/post/p-181957927) as I've found that their tips resonate with lessons I've also had to learn along the way.

## Tip 1: Change your mindset about AI-driven development

- "Vibe-coding" doesn't work when your code needs to ship. Vibe-coding is great until you've got 5,000 of Claude-generated slop that breaks as soon as you need to add a new feature.
- LLMs love writing code. They also love spitting out lots of plausible-looking code that in reality fails very quickly. It writes code with complete conviction, including bugs or nonsense, and it won't tell you that something is wrong unless you tell it first (it may also tell you that something is wrong even if it's not actually wrong).
- You are the expert. Treat an LLM as both an encyclopedic coding wizard and also as an overly ambitious junior developer
- Keep humans in the loop. You need to check the outputs of LLM-generated code at each step of the way.
- **AI tools amplify your expertise** (for better or for worse, emphasis from [this Substack](https://substack.com/home/post/p-181957927)). The leverage you get from AI coding agents is in proportion to the quality of your own software engineering fundamentals.
- The best way to get the most out of AI coding agents is to work as a senior engineer guiding the overly ambitious junior developer.

## Tip 2: Plan, plan, and plan some more

## Tip 3: Break down work. Then break it down some more

## Tip 4: Make sure that you understand each and every single line of code

## Tip 5: Keep your context windows clean

## Tip 6: Use tools to your advantage

### Guardrail model quality with static tools

### Use CI/CD and TDD

### Leverage AI code review agents

- Commit early and often. Also ask Cursor to set up the tasks in small, piecemeal ways that are the size of individual commits.
- Use git worktrees.
- Have opinionated prompts about code style, testing specs (I have the UNIT_TESTING_STANDARDS.md and CODING_REPO_CONVENTIONS.md files for this).
- Use CodeRabbit, CI/CD, static analysis, linters, and other tools.
- Be VERY careful about keeping context windows cleaned. Don't be afraid to start a new agent thread.
- Track progress in simple spec.md (high-level plan) and todo.md (checklist) files.
- Don't commit code that you don't understand.
- Comb through the PRs yourself in Github.
- Even if Cursor suggests a code completion, write out the code yourself. By having to type it up yourself, you end up having to think about it more.
- Put Cursor in "Ask" mode by default unless the solution is trivial or you already know exactly what the code should look like (in which case, "Agent" mode is OK)."
- Ask the AI to give you 2-3 ways of solving a problem and to describe the pros and cons of each.
- Reiterate to the AI your own understanding of the code, and then ask it to see where your understanding is right/wrong/incomplete.
- Make use of design patterns and other well-established ways of writing code.
- Similar to the last point: use tried-and-true code that has a lot of documentation and examples online, as opposed to newer APIs that LLMs may be less familiar with.
- Tell the LLM what NOT to do and what you previously tried but failed, so as to nudge it to consider better options.

AI has actually pushed me to want to improve my own software engineering foundations. I've been reading various books on software architecture and design, and I'm currently reading the classic [Clean Code](https://www.goodreads.com/book/show/3735293-clean-code) book.
