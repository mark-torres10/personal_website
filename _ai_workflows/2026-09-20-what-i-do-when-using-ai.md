---
layout: single
title: "What I spend my time on now that AI writes all my code"
date: 2026-09-20 21:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2026-09-20-what-i-do-when-using-ai
---

# What I spend my time on now that AI writes all my code

Coding models have advanced to the point where they can code much better than I can code. It doesn't remove the need for expertise, but it highlights how there's actually more to creating software than just writing code. In my work as a researcher and as a software engineer, I find that, although knowing how to code has still been an exponentially advantageous skill, in practice, my time is spent doing all the other things that bring more value to what I can deliver (while the AI takes care of coding and more routine tasks).

Some of the things that I spend more attention on now that I can do, given the time I get back from coding and other things AI has automated away.

## Meetings

I generally don't like meetings, and I think that most people who identify as builders tend to not like them as well. Now I like to have meetings to get unstuck and to unblock people earlier on in the process and at higher intervals.

LLMs make it so easy now to have demos or to try multiple versions of implementation. Instead of hypothesizing about a particular set of features, we can now have Cursor or Claude implement a couple of variations, and we can discuss them either during the meeting or in follow-up.

I also find that LLMs make it easy for everyone to be on the same page for meetings. It's easy to have an agent that gives you a briefing before or after a meeting. Now we can also record meetings, which allows us to align on things that were discussed in previous meetings. This allows meetings to be a lot more opinionated and goal-oriented, as opposed to having to get everyone up to speed or hypothesizing about things that would have taken days or weeks to prototype.

## Docs, docs, and more docs

In my individual work, I spend a significant amount of time on documents. I find doc-driven development to be what I spend most of my time doing. I think of this as beyond what has been coined "spectrum development." I think of docs beyond just the initial specs. For me, I spend my days writing, and I often joke that my favorite coding language is English and my favorite IDE is Microsoft Word.

I think of writing as the most critical part of my individual contributor work. This is the part where I begin to own what is being built. I don't believe in the idea of LLMs being a compiler of intent because I think LLMs are actually poor at this task, and I think that comparisons to compiled languages are fraught. However, I do believe strongly in writing as a way for you to own the software that's being built and for you to make sure that you are accountable to what the AI is doing at the end of the day.

### Figuring out what the AI agent should code: spec-driven development

Much of my time is spent on writing tickets or issues that define units of work that the AI agent can do Oftentimes, I collaborate with the AI agent on different variations of how something can be implemented, but at the end of the day, I do own the creation of the specs. I find that when AI makes its own specs, it tends to deviate from what I had intended. This is partly due to the AI misunderstanding my intention, but also in large part due to me being unable to own the software if I myself don't create the specs myself. What may feel good to ship today eventually rots into slop and tech debt because the AI is given too much freedom. The layer where human control of the low-level details should be is at the level of the specs.

### Figuring out what software should be built: architecture planning and design docs

In addition to writing specs, I take a step back and also do a lot of architecture planning and design docs. These docs form the basis for longer product surfaces and feature builds. I find that LLMs tend to go awry with planning larger units of architecture. I trust my own knowledge of the domain and context a lot more.

Though I do collaborate and co-design with LLMs, I find that I still need to critically own it and create a first draft myself, and then I can consider insights that the LLMs might have. I find that LLMs are actually poor at giving suggestions without any sort of human feedback because they tend to overcomplicate or overarchitect. This is a place that requires taste and judgment, which I think that humans can still continue to own in the future.

I make sure to write my own architecture docs and design docs. I have to actively resist the temptation to just blindly copy and paste what the LLM said I should be doing. I've tried that, and it's led to really poorly designed architectures. I like to start at the highest level, where I zoom out at the system I want to build. I think of it as a series of Mermaid diagrams where I incrementally zoom into what I am implementing. This allows me to have the scaffolding of the design in place, and then I can consider advice for specific pieces of implementation or for cross-cutting concerns. Without this, if I just ask the LLM to do the architecture, architecture design itself tends to propose overly complicated designs that I find myself wanting to build enterprise-grade solutions when a simple V1 or prototype would have sufficed.

This is also a place where I use regular pen and paper, as that is a good way for me to make sure that I can understand and own the design.

Over time, I do have patterns that come up, so I do inject these as skills into the AI agent for how I design, as well as things that I think about when designing. At the end of the day, I still do have to have the expertise of doing that myself.

### Synthesizing out what was built: creating runbooks and API docs

I find that most people who actively build AI agents focus on largely writing specs. The people who are tasked with owning larger pieces of software, I find, spend time on the design docs in addition to the specs. However, I hardly find people going back and figuring out what was actually built, which is a key piece that I think comes back to bite most software teams.

A key part of my work is in developing runbooks, API docs, and other pieces of writing to consolidate and define the system that was built. This has a variety of benefits:

1. **It allows me to write about what was built, not just what was intended**. I have to go back and actually explain what was built and how it fits into a larger system. I have to actually synthesize what the code does, which forces me to think about code review in an active way.
2. **It allows me to think about cross-cutting concerns**. It's very likely that, as an app goes in complexity, a new feature or a series of PRs affects multiple user journeys, runbooks, flows, and things like that. By having to write and update the runbooks myself, it forces me to think about cross-cutting concerns.
3. **I can see where my intent and the implementation have diverged**. It's easy to assume that the LLMs convert my intent to compiled code in a one-to-one manner. I hardly find this to actually be the case in practice. If my understanding of the architecture is one way and the code implementation actually says something different, then my attempt to write up what was actually implemented will bring that to light. I often ask an AI agent to review my runbooks and compare it to the actual implementation to see if there's any divergence or inconsistencies.
4. **I can give developers and AI agents context on how the software is intended to work, not just what's currently in the code**. Rather than wasting context and tokens on having the AI agents route across different code files and test files to piece together a picture of the software architecture, I like to use docs and runbooks as a compressed form of context. Rather than having LLMs try to trace a complicated hierarchy of inheritance to piece together a logic picture, I can just explain in a sentence or two what they should be doing (assuming that my write-up was well done). That greatly reduces the context usage as well as reduces confusion for both human developers and AI agents.

I've tried to have AI agents write their own docs and runbooks, and I find this to actually be quite terrible.

1. **It doesn't help me understand and own the software**. Part of the exercise is making sure that I understand what's been built and why, and having AI agents write their own documentation absolves that burden from me, which means that I never have to understand how the software works.
2. **It expands on the proliferation of AI slop**. Whenever I see documentation that's been clearly created by AI, my eyes gloss over it. It tends to be wordy, verbose, overly complicated, and uses vocabulary that no one else is going to use in the real world. I find AI writing to be difficult to understand and unappealing to read.
3. **Having AI generate docs just creates a new surface area for more AI slop**. If your code is already full of AI slop, then having the AI write a Markdown file of how the slop works just increases the amount of characters and therefore tokens spent describing slop.

My speed of shipping software is rate-limited by my ability to synthesize what was built while it's being built. This is an intentional limitation. I think most people are obsessed with the speed gains from using AI agents. However, increasing speed now comes at the cost of additional slop in the future. I want something that I can continue to understand and build upon long term. High-quality software architecture for me is important. In addition, building with AI agents in the future becomes increasingly difficult as the amount of AI slop compounds. AI does very well when given very clear instructions, context, guardrails, and a well-architected codebase. In contrast, AI slop proliferates when there's already AI slop present.

## Learning

The more that you know, the larger the substrate that you can pull from for leveraging AI agents. As a result, a lot of my time is spent trying to learn foundational components and building blocks that give me frameworks for solving problems. For example, I'm generally uninterested in learning the latest libraries or packages or trying different LLMs. Instead, I think about different kinds of problems that I'm interested in solving. For example:

- What are the core problems, architectures, and building blocks for building a chatbot, as opposed to an image classifier, as opposed to a voice agent?
- What goes wrong when you deploy an endpoint without the necessary security?
- What's the latest research on fine-tuning, and where is the field going?
- What problems are people interested in solving?

I benefit from working as a researcher, where I'm incentivized to always stay up to date on the field and to have my own research taste and direction. Not only is that a part of my job, but it also makes me better at being able to properly leverage AI agents. I have better opinions and tastes for what should be built and why. That allows me to be more opinionated in my usage of AI agents and what I can have them do.

## Experimentation

In addition to learning, I also spend my time on experimentation. Rather than being dogmatic about what the best approach to do something is, I kick off experiments to see what the best approach should be. A lot of my time is spent defining experiments and defining what kinds of approaches I want to consider, what success looks like, and how to define the scaffolds of an experiment.

I prefer to experiment on a few approaches and figure out what works best, rather than having to be locked into a single way of doing things. This also gives me judgment on what to build and why, which allows me to direct AI agents in a more opinionated way.

## Reviewing PRs

I still review PRs myself. Luckily, I'm not at the scale of my work where I am overwhelmed by a deluge of PRs. However, I think that attempts to automate away code review are fraught. I think code review is actually a good place to slow down and to make sure that people actually understand the software. I think code review is actually a good rate limiter for shipping software. I understand that there should be a triaging process for what needs review, and I observe that in my work as well; maintenance PRs, for example, get merged in by themselves, and I have a sense for which PRs need more attention than others. 

PRs are also important for team dynamics. PRs are a good way to upskill other developers and to align people on best practices. I find that without PR reviews, developers are hesitant to admit that they don't understand how a piece of software is working, which means that they continue to depend on AI agents to try to understand and they themselves exercise cognitive surrender. PR reviews also seem to be a way for them to be clued in on what software other people are working on, which is becoming increasingly important in AI. This allows individual developers to ship more often and to own larger chunks of work at a time.

I also find PRs to be a good place to refine my agentic workflows. For example, if I see a consistent comment that I have to leave, or something in the PRs themselves that I find to be stylistically incorrect, I can update my skills and prompts accordingly. I notice that some people don't like to review AI-generated PRs because it's overwhelming and full of slop, which I think is indicative of a broader problem of wanting to ship larger chunks of work because AI makes it very simple. I make sure to exercise the appropriate amount of care whenever I need to review PRs. I break up the PRs into chunks of work that I know I could review as a human. I also design prompts to write PR descriptions in ways that I can easily verify. I took the time to implement a few features myself and to codify how I think about breaking down problems into tickets and individual commits. I put that into a scale that I have the AI agents review. That way, when AI agents combine commits into a PR and PRs into stacks of PRs, the implementation unfolds in a way that's aligned to how I think about implementation.

Overall, I find reviewing PRs to still be a very important part of my workflow. I think it's good for team alignment, establishing best practices, making sure that we all own the software, and for evaluating my own agentic workflows. I think that my take is a little against the grain as compared to the larger software world.

## Intentionally slowing down

The temptation for shipping slop is so high. Right now, it seems like a common bet being made in software is that models are going to get good enough that they don't ship slop in the future, so therefore it's OK that software is shipped with slop. I generally have a very strict anti-slop policy. Of course, this depends on whether it's production code or experimentation code, but if software needs to be long-lived, I have a low tolerance for slop.

I think that LLMs do a great job of making you feel efficient and allowing you to get more done on a given day, but that piles up much faster than ever before. What may feel fast to me today will come back to haunt me in a month or two. I've experienced this cycle so often that I find it's actually easier for me now to intentionally own the software layer and do it right the first time than to have to come back later to fix the slop that my overly ambitious self shipped two months ago.

These practices I've listed are ways for me to intentionally limit my speed of shipping software but exponentially improve the quality. I'm still learning and improving along the way, as everyone is with their agentic workflows, but this is part of my broader belief towards finding collaboration between humans and AI that allows both to do what they do best.
