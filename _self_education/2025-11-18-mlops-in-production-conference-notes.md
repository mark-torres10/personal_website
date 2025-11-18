---
layout: single
title: "Notes from the 2025 'AI Agents in Production' Conference"
date: 2025-11-18 15:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /ai_workflows/2025-11-18-ai-agents-in-production-conference-notes
---

# Notes from the 2025 "AI Agents in Production" Conference

I attended this year's ["AI Agents in Production"](https://home.mlops.community/home/events/agentsinproduction2025-mlops-prosus) conference, hosted by the "MLOps Community" group and wrote down some of my notes.

## Talk 1: "Using Agents in Production: Past Present and Future"

### Talk 1 Abstract

Prosus has shipped over 7949 agents. 15% have worked. The rest have been learning experiences. Let's talk about what we have learned, and where we see things going.

### Talk 1 Notes

- engineers don't build agents. People build agents. Adoption is best if it comes from bottoms-up.
- need to upskill at scale and figure out how to empower people to create agents to do so.
  - Give people the tools and guardrails to experiment and iterate at scale.
- A lot of the hard work comes in encoding domain knowledge.

## Talk 2: "Real-Time Voice Agents in Production: Lessons from Building Human-Like Conversations"

### Talk 2 Abstract

At Elyos AI, we’re not building AI to pretend to be human - we’re building AI that can perform like one: completing real tasks, with real quality and real transparency. Our agents will tell you they’re AI. We’re not replacing humans - we’re building trust that AI can help humans work better.

In this talk, I’ll share what it actually takes to make that vision real in production. From designing low-latency pipelines, to managing dialogue context and emotional tone across long calls, we’ve learned that delivering natural, useful conversations is as much an infrastructure problem as it is a language one.

We’ll cover:

How Elyos built a resilient real-time stack combining STT, LLM orchestration, and neural TTS.

Engineering patterns for error recovery, context engineering, context retention and conversational coherence.

The metrics (and human feedback) that actually predict trust and task completion.

If you’re building or deploying AI agents that interact with people, this is a behind-the-scenes look at what breaks, what works, and how we’re bringing transparent, high-performance voice AI to the real world.

### Talk 2 Notes

- Voice is more complicated than speech-to-text -> LLM.
- Real-world interactions come with noise.
- Conversations are more complicated than simple tasks.
- Long conversations can stress LLMs.
- Latency is key for this use case:
  - Have warm starts: workers need to be ready to go.
  - Try different TTS providers.
  - Invest early in observability.
  - Keep tools close to the orchestration layer, to avoid network latency.
- Make workflows deterministic
- Avoid unnecessary RAG. In this use case, (1) models think pretty well already, and (2) people expect voice agents to be lower latency.
- Benchmark against human agents
- Define expected outcomes very well.
- For long conversations, summarize well and often. Leave tool calls outside of the context and push for lots of context compression.
- Transcription of email and post code and things like that is still meh for speech-to-text. More broadly, don't annoy users when input is unclear, have a path for dealing with uncertainty.
- Always have an escalation path to a human.
- Metrics:
  - time to first token (TTFT)
  - groundedness
  - interruptions
  - repetitions
  - outcome success
  - sentiment
  - most common failures

## Talk 3: "Simulate to Scale: How realistic simulations power reliable agents in production"

### Talk 3 Abstract

In this session, we’ll explore how developing and deploying AI-driven agents demands a fundamentally new testing paradigm—and how scalable simulations deliver the reliability, safety and human-feel that production-grade agents require.

You’ll learn how simulations allow you to:

Mirror messy real-world user behavior (multiple languages, emotional states, background noise) rather than scripting narrow “happy-path” dialogues.
Model full conversation stacks including voice: turn-taking, background noise, accents, and latency – not just text messages.
Embed automated simulation suites into your CI/CD pipeline so that every change to your agent is validated before going live.
Assess multiple dimensions of agent performance—goal completion, brand-compliance, empathy, edge-case handling—and continuously guard against regressions.
Scale from “works in demo” to “works for every customer scenario” and maintain quality as your agent grows in tasks, languages or domains.
Whether you’re building chat, voice, or multi-modal agents, you’ll walk away with actionable strategies for incorporating simulations into your workflow—improving reliability, reducing surprises in production, and enabling your agent to behave as thoughtfully and consistently as a human teammate.

### Talk 3 Notes

- Evaluating success isn't about "did it return the right value" but rather "did it help the user reach the goal?"
- The "Agent development lifecycle":
  - Build: define goals and guardrails for the agent.
  - Simulate: run simulations and regression tests.
  - Release
  - Evaluate: evaluate real-world conversations.
  - Optimize: improve agent performance.

I like the following interface for a simulation platform. This is great for creating simulated golden paths and example simulated user journeys.

![Interface for understanding user simulations](/assets/images/2025-11-18-mlops-in-production-conference-notes/1.png)

## Talk 4: Agents as Search Engineers

### Talk 4 Abstract

Search is still the front door of most digital products—and it’s brittle. Keyword heuristics and static ranking pipelines struggle with messy, ambiguous queries. Traditionally, fixing this meant years of hand-engineering and expensive labeling. Large language models change that equation: they let us deploy agents that act like search engineers—rewriting queries, disambiguating intent, and even judging relevance on the fly.

In this talk, I’ll show how to put these agents to work in real production systems. We’ll look at simple but powerful patterns—query rewriting, hybrid retrieval, agent-based reranking—and what actually happens when you deploy them at scale. You’ll hear about the wins, the pitfalls, and the open questions.

The goal: to leave you with a practical playbook for how agents can make search smarter, faster, and more adaptive—without turning your system into a black box.

### Talk 4 Notes

- Simplify simplify simplify! Simple backends are better for reasoning.

![Make tooling design simple](/assets/images/2025-11-18-mlops-in-production-conference-notes/2.png)
![Use simple backends where possible](/assets/images/2025-11-18-mlops-in-production-conference-notes/3.png)

- Query-time content classification does a great job at improving query performance.

![Results of query-time content classification](/assets/images/2025-11-18-mlops-in-production-conference-notes/4.png)

- Scoring retrieval should go beyond LLM-as-a-judge. You can look at other metrics like CTR, latency, users retrying, etc., all as other ways of measuring how good your semantic search retrieval is.


## Overall takeaways

