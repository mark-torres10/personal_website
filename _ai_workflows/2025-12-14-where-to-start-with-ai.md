---
layout: single
title: "Where to Start with AI: A Practical Framework for Getting Started (Without the Hype)"
date: 2025-12-14 13:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-12-14-where-to-get-started-with-ai
---

# Where to Start with AI: A Practical Framework for Getting Started (Without the Hype)

I've worked with various clients in my AI consulting practice, many of whom are just starting out. Lots of people are enamored by flashy demos and hype and want to see how AI works for them.

They come to me with questions like:

- "We saw this demo and it looks amazing—can you help us implement it?".
- "I want an agentic AI multi-agent workflow [insert other buzzwords here] to automate away XYZ using OpenAI".
- "Should we use [insert latest AI vendor]?".
- Everyone else is using AI, and we need to come up with our AI strategy, but we don't know how to.

They're excited (or feeling a bit of FOMO), they're ready to invest, but they don't know where to start (and are worried that everyone else is already starting and that they've fallen behind).

I tell them: "Before we talk about vendors or enterprise solutions, let's figure out if AI can actually solve your problem. And the best way to do that? Start with prompts."

At the end of the day, **any AI tool is fundamentally a series of prompts, stitched together clever ways.** I've built many agentic AI applications myself and you can see the "chain" of prompts that are working under the hood to do all the magical AI things that people see.

There's an art and skill of creating these seamless experiences with AI, and I'm not brushing that off. However, when you're starting out, it's good to remember that at a high level, AI applications are all powered by ChatGPT (or some other similar LLM) under the hood. So, the best way to know whose AI applications are best at doing *your* tasks is to try it yourself.

{% include figure
   image_path="/assets/images/2025-12-14-where-to-start-with-ai/1.png"
   alt="A visualization of the AI agent workflow"
   caption="**Same workflow, different packaging:** Whether you're manually using ChatGPT or paying for a vendor's AI platform, you're following the same process. The vendor just automates the steps you'd do yourself and adds their own secret sauce (e.g., including proprietary instructions based on what they've learned across all their clients, splitting up a single prompt into multiple prompts (this is what \"agentic AI\" means)). Picture credits: ChatGPT (they've really improved their image generation)"
%}

## The Framework I use to advise

After working with and advising many clients, I've found that successful AI adoption for this sort of use case follows a predictable pattern. Here's the framework I use, and why each stage is necessary.

**The 4-stage progression:**

| Stage | What You're Doing | Why It Matters | Typical Duration |
|-------|------------------|----------------|------------------|
| **Part 1: Prompts** | Writing and iterating on manual prompts | Builds intuition for AI capabilities and limitations | 2-4 weeks |
| **Part 2: Tools** | Using off-the-shelf AI applications | Validates use cases before major investment | 2-4 weeks |
| **Part 3: Automation** | Building simple workflows (optional) | Scales validated workflows when needed | 2-4 weeks |
| **Part 4: Vendors** | Evaluating enterprise solutions | Only when you have clear requirements and scale needs | Ongoing |

### What each part looks like

**Part 1: Write your own prompts** - You're learning what AI can do, what it can't, and what inputs produce what outputs. It's manual, it's slow, but it builds the intuition you need for everything else.

**Part 2: Use off-the-shelf tools** - Someone else has figured out the basics, and you're adapting their approach to your needs. You're still cooking everything up yourself, but with better tools. This isn't your regular ChatGPT chat anymore. Instead, it's more specific tooling like powerpoint presentation builders, AI-embedded word docs, image generators, etc. Increasingly, big tech companies like Google and Microsoft are embedding these into their apps by default, so you likely have seen these off-the-shelf tools without even knowing it (e.g., Google's AI summaries in their search tool, Microsoft Copilot, Applie Intelligence).

**Part 3: Build simple automations** (optional) - You've done this task enough times that you want to streamline it. You're seeing repetition in how you're using the tools. You find yourself doing something like:

- Download the latest Excel reports
- Pass them to ChatGPT (or some other app).
- Paste in this specific prompt.
- Download the results.
- Paste those results into a different Excel sheet.
- Write an email with the latest results and attaching that Excel sheet that you just created.

**Part 4: Explore vendors** - Only now, when you understand the tools from first principles, you've built intuition for what AI can and cannot do, and you've tried automating things yourself, should you explore vendors. You know what features matter, what's marketing fluff, and what you're actually paying for.

### "Can't we just rush through these steps?"

I get this question a lot. Clients want to skip ahead, especially if they're under pressure to show results quickly. "Can't we just spend a day on prompts and move to tools?" or "Why can't we evaluate vendors while we're learning prompts?"

Here's why rushing backfires: The more you skip, the worse your outcomes get. Each stage builds intuition that the next stage requires.

- **Prompts → Tools:** Without prompt intuition, you can't evaluate if a tool's AI is actually good or just marketing.
- **Tools → Automation:** Without validating a use case works, you'll automate something that doesn't deliver value.
- **Automation → Vendors:** Without understanding your exact needs, you can't cut through vendor hype or negotiate effectively.

Go through these steps slowly and intentionally, documenting what works and what doesn't, keeping track of your learnings, iterating on your approach—is what builds intuition about AI tools. This isn't busywork; it's the foundation.

At the end of the day, your intuition for how AI tools work and when to use them is going to be one of your biggest competitive advantages. Having access to all the AI tools in the world doesn't matter if you don't know which ones are good for certain tasks, which ones are overhyped, and how to evaluate new tools as they emerge.

**Some things to consider after each part:**

- **After Part 1:** Do you understand what AI can do for your use case? If no, keep practicing prompts. If yes, move to Part 2.
- **After Part 2:** Does an off-the-shelf tool solve your problem? If yes, you might be done. If no, consider Part 3.
- **After Part 3:** Do you need enterprise features (security, compliance, support, scale)? If no, your automation might be sufficient. If yes, Part 4.

**The key insight:** Most organizations stop at Part 2 and never need vendors. The ones that do need vendors are much better positioned to evaluate, negotiate, and implement because they understand their requirements from first principles.

## Part 1: Write your own prompts

**Key takeaway**: If you can't write a prompt to do it yourself, neither can anyone else.

Some things to keep in mind:
- What information do you have to include in the prompt for it to give you the answer you want?
- What is the *least* amount of information that you can give to an AI for it to tell you the right answer? (AI models, like humans, can get distracted and confused when you give them unnecessary extra information).
- How many times do you have to prompt the AI for it to give you the answer you want?
- 

Some benefits you get from this are:
- You build an "intuition" for what any AI answering your specific questions should do.
- ...

### Benefits

#### Benefit 1: Creating your own personal prompt library

As you build your own prompts, you create your own personal "prompt library". This has MANY benefits, including:

- **Being a part of your own proprietary AI secret sauce**: as you experiment and create prompts, you create your own internal repository that becomes a part of your edge. These prompts become a part of how you get your results and are prompts that your competitors don't have.
- **Gives you a set of prompts for different use cases**: some clients I've worked with have folders to organize their prompts, and then when they need to do a specific type of task, such as due diligence or analyzing reports, they go to that folder and they just copy and paste those same prompts into ChatGPT.
- **Lets you build your own "test" to test against any new LLMs**: new versions of ChatGPT and other LLMs come out all the time, and there's so much hype and press around how these LLMs are now smarter than humans, curing diseases, automating away everyone's jobs, and so on. AI companies all have their [own internal prompts](https://openai.com/index/gdpval/) that they use to test any new LLM, and so should you. Having your own personal prompt library helps you get past the fluff and see when a new LLM actually does better for your specific use case.

#### Benefit 2: Intuition for how AI works on *your* use case

A lot of press and hype focuses on how AI is, for example, smarter than the [smartest mathematicians](https://deepmind.google/blog/advanced-version-of-gemini-with-deep-think-officially-achieves-gold-medal-standard-at-the-international-mathematical-olympiad/). But unless you're working on building math proofs, writing code, or other specific tasks that these benchmarks care about, those test scores don't matter to you (and even if you do work in those fields, AI doing well on those doesn't mean they've automated you away). Having your own prompt library lets you build intuition for how the different AI models do for the things that you care about.

**It doesn't matter how well AI can solve other people's problems. It matters how well it solves yours**

#### Benefit 3: Separate reality from hype

For all the hype about how great ChatGPT is, I still sometimes find it messing up on really obvious mistakes (e.g., punctuation, not knowing how to interpret a word, etc.) that a human wouldn't make. This doesn't discount how powerful these tools are, but it also shows you where they still have room to grow.

#### Benefit 4: Spotting common issues in AI

You'll learn some of the common problems and gotchas that happen with LLM applications, such as hallucination, context drift, and formatting. You'll see, for example, that when you provide LLMs tables, LLMs like tables formatted in a very [specific way](https://github.com/BoundaryML/baml).

#### Benefit 5: Confidence

By doing it consistently yourself, you build up confidence to know when AI is the right tool versus when to use traditional methods. You also have more grounding on when to believe statements about AI versus when to discount the hype.

### Common Mistakes to Avoid

- **Too vague:**
  - Avoid: "Help me with marketing"
  - Do: "Create a social media post for our new product launch"

- **Don't attach every single document into ChatGPT:** Despite all the demos that dump in entire troves of books into ChatGPT and answer questions about it, in reality LLMs do significantly better when you give it *just* the information that it needs to learn.
  - Avoid: Dumping entire documents when you only need a summary.
  - Do: Pick just the snippets/pages of documents that you need in order to answer the question.
Not iterating: Giving up after one bad result instead of refining

- **Having super-long chat conversations** An LLM tries to compress everything that you've talked about in a given chat conversation, and it generally does pretty well (for example, it more heavily weighs recent messages over really old ones) but it can forget things that came up in old messages.
  - Avoid: having one single super-long chat conversation thread with ChatGPT.
  - Do: start a new conversation
  - Some pro tips:
    - Ask ChatGPT to summarize the current conversation with enough details to pass on to another LLM.
    - Use the ["Branch in new chat" feature](https://x.com/OpenAI/status/1963697012014215181?lang=en) from ChatGPT to continue your current conversation in a new chat.

- **Make sure the information that you give to ChatGPT is useful:** ChatGPT can only help you based on (1) information that it's learned from being trained on the Internet, (2) whatever prompt you give it, and (3) what documents, files, etc., you attach. Make sure that whatever information you give to ChatGPT is useful for answering your question. In fact, OpenAI and other big companies invest a lot of effort making sure that ChatGPT is built using only *high-quality* training data and tuned with *high-quality prompts*, because they also understand that if you give an LLM too much slop, it crashes.
  - Avoid:
    - Poorly formatted documents (e.g., tables aren't clear, images are blurry, Excel files have terrible formatting or ambiguous column names).
    - 
  - Do:
    - Give, whenever possible, well-formatted documents.
    - 

Ignoring formatting: Not specifying output format (markdown, bullet points, etc.)
Skipping documentation: Not keeping track of what works

(double-check. Don't believe everything that an LLM says. Ask it to cite specific sources. Check the links yourself.)

(don't assume that the LLM is right. In fact, ask it to figure out how it's wrong (and when it's wrong). Ask it for when it would check itself. Also pass in the prompt to another LLM. TBH this could be a separate blog post on "pro tips to avoid hallucination and sycophancy", and I can just link it here.)

**Key tip:** imagine ChatGPT is like a super-powered human intern. If you're not specific with what question you're asking, the intern might go off and do the task in an unexpected way (since they don't have the context that you have). If you give this super-intern 10 years worth of documents and you ask them to write a document for it, it might do a reasonable job (and probably be pretty impressive that it can go through 10 years worth of PDFs) but it won't know, unless you tell it, which documents are important and how they relate to each other.

### Pro tip: find and use prompts by other people

Many people across industries have found and written about ChatGPT prompts that they use for their specific workflows. If you've opened up LinkedIn for any bit of time, you've likely encountered these influencers yourself. Don't believe the overhyped "this ONE prompt will solve ALL your problems" claims, but I also suggest trying some of these prompts yourself. This will help you build your own intuition and ...

<!-- 

## Part 2: Start using basic off-the-shelf AI tools

Some of these include:

- Gamma: for building presentations with prompts.
- Cursor: for AI-assisted development.
- AlphaXiv: for reading papers with an AI-enabled interface.
- ChatGPT apps: 

## Part 3 (Optional): Build simple automations

If using ChatGPT and an off-the-shelf AI app don't do it for your use case, I recommend building your own automations

- n8n
- Zapier
- (ChatGPT has the ability to schedule prompts?)
- Basic cron jobs (if you have basic programming experience)

**Warning: Stay AWAY from vibe-coded apps and from vibe-coding your own apps.**

*Caveats*
- The apps are for fun.
- You're not relying on the outputs of the apps for anything.
- You're OK if the app gets hacked and someone steals all the data.

## Part 4: Explore vendors

(only after you've done the above do I suggest using vendors)

(vendors will often overhype and promise that their services are so amazing and AI-powered)

(but it's often really hard to get an AI service to work for your specific use case, and you'll have a hard time learning how to do it)

(you'll have signed up and spent all this time doing due diligence and there's still going to be a lot of onboarding required)

(from my experience with clients, specific vendors are overkill anyways, at least until you've reached a certain point in your AI journey where you know what AI can and can't do in your business)

(caution: many "all-in-one" vendor tools don't work for *your* business and *your* way of doing things. An "AI data analyst" at Morgan Stanley is going to look very different than an "AI data analyst" at Nike). -->
