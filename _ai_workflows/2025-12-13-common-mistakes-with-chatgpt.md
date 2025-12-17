---
layout: single
title: "Common Mistakes with ChatGPT (And What to Do Instead)"
date: 2025-12-13 13:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-12-13-common-mistakes-with-chatgpt
---

# Common Mistakes with ChatGPT (And What to Do Instead)

I've worked with many clients in my AI consulting practice, and I've noticed a pattern: people are either not using ChatGPT at all, or they're using it but not getting nearly as much value as they could.

Some people try ChatGPT once or twice, get mediocre results, and assume it "doesn't work for them." Others think ChatGPT isn't as good as they are at their job, so why bother? Many more use ChatGPT regularly but don't maximize how much they get out of it—they're getting 30% of the value when they could be getting 90%. And then there are those who see colleagues or peers getting amazing results with ChatGPT but have no idea how or why.

The problem isn't that ChatGPT is limited or that some people are just "better" at using it. The problem is that most people haven't learned how to prompt effectively. They treat it like a search engine or a simple chatbot, when it's actually more like working with a smart but very oblivious intern who needs clear, specific instructions.

Taking a ["Prompting 101" class](https://www.coursera.org/specializations/prompting-essentials-google) should give you the basis of what a good prompt looks like. In addition to those fundamentals, here are the most common mistakes I've seen from years of working with LLMs and helping others learn how to use them—and what to do instead.

## Being too vague

The most common mistake I see is treating ChatGPT like a mind reader. When you give vague instructions like "help me with marketing" or "analyze this data," the AI has to guess what you actually want. It might give you a generic response that doesn't match your needs, or it might go in a completely different direction than you intended.

Think of it like giving directions to a human colleague: "Go work on that project" is useless. "Create a social media post for our Q4 product launch targeting enterprise customers" gives clear direction. The more specific you are about the task, context, audience, tone, and desired output, the better results you'll get.

**Avoid:**

- "Help me with marketing" (What kind of marketing? For what? What format?)
- "Analyze this data" (What kind of analysis? What should I look for? What format do you want the results in?)

**Do:**

- "Create a social media post for our new B2B SaaS product launch. Target audience: enterprise decision-makers. Tone: professional but approachable. Include a clear call-to-action. Format: LinkedIn post, max 300 words."
- "Analyze the attached sales data and identify: (1) top 3 revenue-generating products this quarter, (2) any products showing declining trends, (3) recommendations for next quarter. Format results as a bulleted list with specific numbers."

These example prompts are starting points that you can use, and I'd suggest diving even deeper and being even more prescriptive and detailed to improve performance.

## Attaching every single document into ChatGPT

Despite all the demos that dump in entire troves of books into ChatGPT and answer questions about it, in reality LLMs do significantly better when you give it *just* the information that it needs to learn.

Think of it like asking someone to find a specific fact in a library. If you point them to the exact shelf and book, they'll find it quickly. If you tell them "it's somewhere in this entire library," they'll spend time searching through irrelevant material. The same principle applies to LLMs: more context isn't always better—relevant context is what matters.

**Avoid:**

- Dumping entire 100-page documents when you only need information from pages 15-20
- Attaching multiple versions of the same document without explaining which one is current.
- Including every related file "just in case" the AI needs it.

**Do:**

- Extract and attach only the specific pages, sections, or data points relevant to your question
- If you need information from multiple documents, clearly explain what each document contains and why you're including it
- When possible, paste the exact text snippet you need rather than attaching the whole file

## Not iterating: Giving up after one bad result

Many people try a prompt once, get a mediocre result, and assume AI "can't do it" or "isn't good enough." But prompting is iterative and the first attempt is rarely your best. The most effective prompters treat it like a conversation: they refine, clarify, add context, and try different phrasings until they get what they need.

Think of it like editing a document. Your first draft is rarely perfect. You revise, clarify, restructure. The same applies to prompts. Each iteration teaches you something about what the AI needs to understand your request better.

AI is a powerful tool, and although there are genuinely things that AI can't do as well as we humans would like, in general I find that people who write off the use of AI do so prematurely after 1-2 bad results from prompts. AI is a very powerful tool, but you need to iterate over and over to learn how to use it well. This is also a continual process—what works well for prompting GPT-4 might not work well for GPT-5, and what works well for prompting OpenAI models may be less so for Gemini or Qwen or Claude.

**Avoid:**

- Trying a prompt once, getting a mediocre result, and giving up
- Assuming the AI "can't do it" without trying different approaches
- Not documenting what you tried so you repeat the same mistakes

**Do:**

- Treat your first prompt as a starting point, not the final version.
- If the output isn't quite right, ask for specific refinements: "Make it more concise" or "Add more detail about X".
- Keep a simple log of what worked and what didn't, so you can build on successful patterns.
- Try 2-3 different phrasings of the same request to see which approach works best.
- Use a [prompt improver](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompt-improver) to improve the quality of your prompts.
- Ask an LLM how to improve your prompt. Tell the LLM what you tried (paste in your prompt or snippets of your conversation), what the result was, and why you think the result is bad, and see what new prompt the LLM generates. **This is one of my most impactful and least-commonly-known pieces of advice for improving prompt quality.**

## Having super-long chat conversations

An LLM tries to compress everything that you've talked about in a given chat conversation, and it generally does pretty well (for example, it more heavily weighs recent messages over really old ones) but it can forget things that came up in old messages. As conversations get longer, the AI's ability to maintain context degrades. It might forget important details from earlier in the conversation, or it might get confused by conflicting information that appeared at different points.

Additionally, long conversations become harder for you to manage. Finding that one prompt that worked becomes like searching through a 50-page document. Starting fresh conversations for new topics keeps things organized and ensures the AI has full context for each task.

**Avoid:**

- Having one single super-long chat conversation thread with ChatGPT spanning weeks or months.
- Mixing multiple unrelated topics in the same conversation.
- Expecting the AI to remember important details from 50+ messages ago.

**Do:**

- Start a new conversation for each distinct task or topic
- If you need to continue a previous conversation, use ChatGPT's ["Branch in new chat"](https://x.com/OpenAI/status/1963697012014215181?lang=en) feature to start fresh while maintaining context.
- Before starting a new conversation, ask ChatGPT to summarize the current conversation with enough details to pass on to another LLM (then paste that summary into your new chat).
- Organize your chats by topic or project so you can easily find what you need later. For example, ChatGPT has folders that you can use to organize prompts.

## Providing poorly formatted or low-quality information

ChatGPT can only help you based on (1) information that it's learned from being trained on the Internet, (2) whatever prompt you give it, and (3) what documents, files, etc., you attach. Make sure that whatever information you give to ChatGPT is useful for answering your question. In fact, OpenAI and other big companies invest a lot of effort making sure that ChatGPT is built using only *high-quality* training data and tuned with *high-quality prompts*, because they also understand that if you give an LLM too much slop, it crashes.

When you give the AI poorly formatted documents, ambiguous data, or contradictory information, it has to guess what you mean, and it often guesses wrong. A blurry image, a table with unclear column headers, or an Excel file with merged cells and inconsistent formatting will lead to poor results. The AI is only as good as the information you feed it.

**Avoid:**

- Poorly formatted documents (e.g., tables aren't clear, images are blurry, Excel files have terrible formatting or ambiguous column names).
- Including contradictory information (e.g., 2 files with different takeaways) without clarifying which source is authoritative.
- Attaching files with incomplete or missing data and expecting the AI to "figure it out".
- Providing context that's irrelevant to the question you're asking.

**Do:**

- Give, whenever possible, well-formatted documents with clear structure and labels.
- Clean your data before attaching: remove duplicates, fix formatting issues, ensure column headers are clear.
- If you must include conflicting information, explicitly state which source is authoritative: "Use the Q4 report as the primary source; the Q3 report has outdated numbers".
- Provide context about why you're including each document: "This document contains the customer requirements; use it to draft the proposal".

## Ignoring output format specifications

Not specifying how you want the output formatted is like asking someone to write a report but not telling them whether you want paragraphs, bullet points, or a table. The AI might give you something technically correct but in a format that doesn't fit your workflow, forcing you to reformat it manually or re-prompt.

Specifying format upfront saves time and ensures the output is immediately usable. It also helps the AI understand your expectations better—knowing you want a table versus paragraphs changes how it structures the information.

**Avoid:**

- Asking for analysis without specifying format (you might get paragraphs when you wanted bullets, or prose when you needed a table)
- Assuming the AI will "figure out" the best format for your use case
- Not specifying length constraints (word count, number of items, etc.)

**Do:**

- Be explicit about format: "Format as a bulleted list" or "Create a markdown table" or "Write in email format".
- Specify structure: "Present as: (1) executive summary in 3 bullet points, (2) detailed analysis in markdown table, (3) key recommendations numbered 1-5".
- Include length constraints: "Write a 200-word summary" or "List top 5 items" or "Keep each bullet point to one sentence".

## Skipping documentation: Not keeping track of what works

Many people try prompts, get good results, but don't save them anywhere. Then weeks later, they need to do the same task and can't remember what prompt worked. They end up starting from scratch or trying to recreate something they already figured out.

Documentation doesn't need to be elaborate—even a simple text file or Google Doc with your working prompts is valuable. Over time, this becomes your personal prompt library, and you'll find yourself reusing and adapting successful prompts rather than starting over each time.

**Avoid:**

- Trying prompts, getting good results, but not saving them anywhere.
- Not noting what modifications you made to get better results.
- Forgetting which prompts worked for which use cases.

**Do:**

- Keep a simple document (Google Doc, Notion, even a text file) with your working prompts
- For each prompt, document: what it does, what inputs it needs, what outputs to expect, and why it works
- Organize by use case or department so you can quickly find what you need
- Note any variations you tried and which worked best
- Use a version control system (like Google Docs' history feature, or, for the more technically savvy, something like Git) to more systematically track changes.
