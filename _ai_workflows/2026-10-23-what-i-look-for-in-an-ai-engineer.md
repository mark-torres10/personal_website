---
layout: single
title: "What I look for in an AI engineer during interviews"
date: 2026-10-23 09:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2026-10-23-what-i-look-for-in-an-ai-engineer
# Jekyll excludes documents with published: false from output, feeds, and collection listings.
published: false
---

# What I look for in an AI engineer during interviews

I interview AI engineer candidates for Lemon.io. The interview is a whiteboard walkthrough of a system the candidate actually built, never leetcode, so I can see how they reason about software they had to own.

## What senior means

Senior, for me, is technical depth, ownership of software that has to live past the demo, and the ability to talk about that software with stakeholders. Most candidates I meet are technical enough to pass a basic smell test. I downlevel people much more often because they cannot own a long-lived system, or because they cannot explain the tradeoffs to someone who did not build it.

## Judgment about the system

I look for the simplest reliable approach. I want them to separate what should be generated, retrieved, classified, validated, stored, reviewed by a person, or handled with a rule. The LLM is one component in that split. A strong candidate asks what each part of the system has to do before they pick a model.

The best candidates also name the constraint before they draw the architecture. Cost, latency, privacy, licensing. My phrase for candidates who do this is "constraint-driven, not tool-chasing."

I get skeptical when a candidate proposes fine-tuning early. Fine-tuning can be the right call for style, classification, or a narrow task, and I still want the comparison: prompting, retrieval, a classifier, or rules, plus the data they would need, the eval that would have to pass, and the cost to maintain it. From the interviews I have done, people who propose fine-tuning first usually cannot justify it.

## Evals

After they pick a simple design, I still ask how they know it works.

The version I trust has test cases. They put deterministic checks in code, and they use an LLM judge for the qualitative ones. They run cases more than once so they have a pass rate, and they swap models only once those numbers hold. When something fails, they add a regression case for it.

One person I interviewed is a clearly capable builder. He had shipped full products on his own, front end to deployment, with a thoughtful spec-driven workflow where an orchestrator agent handed work to coding subagents. He had built a RAG assistant that answered shoppers' product questions. I asked the question I ask everyone who has built RAG. How do you know it works? Is there a test set? What scores do you look at?

He described a bug where a question about one product retrieved two. He fixed it by changing the prompt and reorganizing the vector store. When I pushed on how he tested the fix, he mentioned a tool whose name he couldn't remember, and the tracing mostly showed token usage. He is a good engineer, and I recommended evaluating him as a full-stack engineer, where he looked strong, rather than for the AI role. The issue wasn't the bug, since every retrieval system has that bug. He had no way to tell whether the fix made things better or just moved the problem. In AI engineering, the eval loop is the job.

## Production

Once a system is live, I listen for latency, cost, observability, and failure modes.

I want a latency budget broken down by component, a fallback when a tool fails, a staging path, and monitoring that shows whether a miss came from retrieval or from reasoning. On cost, they should be able to say why a given task gets a cheaper model or a stronger one, and what they measured. On failure, they should know what the user sees when the model is wrong, slow, or unavailable.

A candidate who builds phone-based voice agents walked through two systems at two companies. Before he drew a box, he explained why they differed. In the first, calls were small-value reminders, so the agent had to cost well under a human caller, keep calls short, and sound natural enough that people stayed on the line. In the second, each lead was worth far more, so cost mattered less and conversion mattered more: re-engaging people, reminding them why they signed up, and handing off to a human when a tool failed.

Asked about latency, he broke a reply budget of about one second down component by component: transcription, the model call, voice generation, and tools. The tools were the slow, flaky part. He also mentioned failure modes. A component-level latency breakdown is what I want from a senior engineer. Asked how he knows a change is safe, he described a model upgrade he held back because latency spiked and became inconsistent. He adopted it only once it met the targets. He uses an LLM to score call transcripts against error criteria, and he still listens to calls himself. The model was one component in a system with a business goal. The decisions came from measurements.

## Communication

Communication is the most common reason I downlevel someone. If they can't explain it to you, they can't explain it to a client.

Plenty of candidates can describe the architecture to another engineer. Fewer can explain the same tradeoff to a founder or a client as a decision about product, cost, risk, and timeline, and keep the technical point while they do it. I also want a narrow first version with a clear success metric.

The voice-agent candidate showed that skill when he explained why the two systems differed before he talked about components.

## How they use AI day to day

I also pay attention to how candidates use AI tools in their own work.

**Awareness.** They know the output is unreliable by default. They mention specific failure modes, such as hallucinations, outdated data, and false confidence, and they treat the output as a starting point.

**Delegation.** They decide what to hand off and what to keep. Boilerplate, tests, and docs can go to the model. Architecture, core logic, and security decisions stay with them, and they can say why.

**Validation.** They check the logic, the edge cases, and the API, and they spend more of that effort when the task is riskier.

**Ownership.** They treat generated code as their own, and they can walk through it in detail and defend the decisions.

**Balance.** They use the tools on purpose. They can say when AI adds value and when it does not, and they can work without it.
