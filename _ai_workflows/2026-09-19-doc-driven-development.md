---
layout: single
title: "Doc-Driven Development and how I think about working with AI"
date: 2026-09-19 16:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2026-09-19-doc-driven-development
---

# Doc-Driven Development and how I think about working with AI

Everyone has their own ways of working with AI. What I see, though, as a common pattern are two extremes:

1. The people who try to avoid using AI or are very quick to find all the ways that it goes wrong.
2. People who are overly enthusiastic about using AI without maintaining control, thereby surrendering their cognitive abilities to the latest LLMs.

I think both of these are actually quite poor approaches. Right now, no one's really pinned down a good way to be able to work with AI agents in a way that is able to leverage the best of both human expertise and also what the LLMs do best. The examples that I found online tend to be promotional marketing or really big examples of how people actually use AI in their work. Some other examples I find tend to be a little too abstracted or don't explain how they got to where they were.

What I'll talk about here is something that I'm calling "doc-driven development," which is how I think about developing with AI agents.

I work as both a researcher and a software engineer, and both these worlds are using AI in different ways, each of which is adapted to how they think about their world.

- I find that researchers tend to not own the code whenever they use LLMs, unless they themselves are quite technical. This leads to a scenario where it's difficult to own the results of the experiments because it's difficult to ascertain attribution as well as know where your expertise comes in. For example, if you rely solely on Claude to generate ideas for experiments or ablations, then it makes it difficult for you to actually own what it is that you are experimenting with. However, I also don't think that it's necessary for researchers to know every single bit of detail of the code that they've written, and I think LLMs being able to write code has actually been net good for research.
- On the engineering side, I think plenty has been said about engineers not knowing how to code anymore or just being glorified prompt engineering automatons. There's still a lot of open work on how to best use AI and software engineering without it descending into slop. However, I think that it's easy to accept slop when you don't have to own the result of the work that you're building, or you don't spend time looking at the larger picture.

My work requires that I have to build systems that are robust, reliable, and scalable. To do so for research applications, it doesn't do much good if I build an app that's used for a paper if that app's results are not reliable. The constraints I have are from both the research and engineering sides, given that I've had to spend a lot of time thinking about how to create robust engineering solutions while doing so under the fast-paced, experimental, and iterative constraints of research.

To do so, I spend a lot of time decomposing what the research process looks like for me. I've spent the past few years in this capacity, even before there were coding agents, and I systematize the way that I think through the process of research. For me, there's actually a lot of overlap between how to think about research and also how to think about product and features in the software engineering world. Broadly speaking, the work itself can be decomposed into a series of steps:

1. **Ideation**: figuring out what needs to be built and why.
2. **Design**: given the idea, design the architecture and the solution needed. The details of the architecture are different depending on whether it has to live for a long time or just needs to create a visualization that's used in the appendix of a paper.
3. **Plan the implementation**: once the architecture is decided, plan the actual implementation to get there. This can take the form of a series of tickets. It can also take the form of a series of steps in an experimental design. Either way, plan the implementation.
4. **Execute the implementation**: given the architecture, the idea, and the plan, actually execute the implementation. I think this part is the part that LLMs are most commonly used for and actually do the best on. However, people overinvest in this part and underinvest in the first three parts.  What this means in practice is that LLMs are acting, but they don't have the guardrails or the goals or the expectations needed to align what they're doing to what the human actually wanted in the first place.
5. **Review the implementation**. Given the execution, the plan, the original design, and the intention for what we wanted to do in the first place, review the implementation. See if it makes sense. Look at the code itself to see if the core interfaces look right and if the result looks sufficiently efficient or meets whatever constraints you have. Then get the result, understand it, and figure out next steps.

Too much time is often spent on using LLMs to execute the implementation, and not enough time is spent on the other parts of the process. I think this prioritization is actually inverted. Most of my effort and attention is spent as the human in the loop. I come up with the designs and the different ways to do something. I own the idea, the architecture, and the intended results of what it is that I'm building, whether it be a research experiment or a new microservice for an app. I then review the implementation, and I think about things like:

- how well it actually executed on the intention
- whether there was something that I had missed that wasn't a part of my initial design
- where this goes wrong in 3 months

To do all this, I actually spend most of my time writing and defining documents. In lieu of writing code, I instead write very detailed specs and design docs, and then I spend a lot of time writing comments and reviews on PRs to make sure that the standard for the work is high. I then spend time trying to find patterns and themes in the work I'm doing, in the prompts that I used, and in the workflows that I did (to see if there's anything that can be abstracted out as an instruction or a skill or a runbook for a future agent to use).

I've found that this doc-driven development approach is able to leverage the best of both humans and LLMs:

1. By focusing my attention on the design, ideation, planning, and review steps and codifying it in documents, I know and keep track of what I intended to build and why. LLMs are really good at following instructions, but I have to be the one to figure out where the LLMs' work fits into the larger picture of what I'm trying to work on in the first place. Importantly, it also allows me to maintain control and ownership over what the LLM does.
2. The LLMs get to own what they do best, which is being an execution engine. They are really good at executing a well-defined plan, and oftentimes I find that when the LLMs deviate or do something unintended, it is 98% of the time a result of me having a poorly planned-out plan or design. I take that as feedback for improving my designs in the future.

I write in very verbose detail what my intention and my plan is, and the LLM compiles that into a working implementation. I may not have to know every single line, though I am responsible for every single line of code. I do know and can own, at a higher level, what has been built.

Some caveats to note here:

1. **This is a very cognitively demanding process**. I can't do this for 8 hours a day. That being said, this, I think, is the most important part of the work anyway, and is something that you tend to do as you move up in your career, whether it be as a researcher or as a tech lead.
2. **This makes it difficult to context-switch**. I find that I can really only focus on a few projects at a time, and on a given day, maybe one or two. I give those projects my deep attention, but doing so means that I have to avoid context-switching as much as possible.
3. **It's very easy to still be susceptible to AI sycophancy**. I use LLMs to give me iterative feedback as I'm doing the doc-driven development process. However, it's very easy for the LLM to ask misleading questions, to guide you awry, or to tell you how brilliant and awesome your idea is without actually having the context to know whether it's any good or not.

At the end of the day, you have to own what you build. It's okay if the recommendation came from Claude, but you have to understand why, and you have to probe and inspect the idea to see what other alternatives could have been. I am not perfect at this, though. What I have found to help is to give fresh context every single time to different agents, ask agents to intentionally find mistakes, and then just try things myself and see where things break.  One specific example is that oftentimes when I ask AI to create a plan or to help me create a plan for a new microservice, it tends to dive into the minutia of things like permissions, which are very much out of scope for a V1 greenfield project. I have to have the context to know what parts of the development process are important where and to be able to qualify the AI's recommendations accordingly. This also applies in the research process, where I still find that AI does a poor job at hypothesis generation. Even if you pass in a web search tool to use in order for it to find papers, it still does poorly at hypothesis generation and has poor taste. Therefore, it's up to you to read the papers yourself, to synthesize them yourself, and to have your own opinions on research direction and what is worth trying to research in the first place.

We're also just trying to figure out the right way to work with AI, and a lot of these insights may be out of date in a year or two. However, for me, it's really important to still take ownership of what it is that the AI is doing and to avoid cognitive surrender. These are just some thoughts and ways that I have found for how to make sure that I still stay close to the process of working with AI (while also allowing AI to function where it works best in a collaborative process that leads to the best outcomes possible).

It's very telling when someone has outsourced their entire work to Claude because they themselves can't explain what's going on or why something was built. However, it's also very tempting to do so. Having these processes in place is how I make sure that I can still own the work, even if AI is writing all the code.

Here's a video of me doing this on a real task: [video link](https://zoom.us/clips/share/67fDB9PURQaBBKitzVTU3Q).
