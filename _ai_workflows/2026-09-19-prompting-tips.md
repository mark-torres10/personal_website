---
layout: single
title: "Basic tips for prompting well"
date: 2026-09-19 17:30:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2026-09-19-prompting-tips
---

# Basic tips for prompting well

Given how present LLMs are in our work, it's good to have the ability to prompt LLMs well. Here are some tips on how to get better at prompting (from [a talk I gave on prompting best practices](https://docs.google.com/presentation/d/189VYscba8AbR5PkQg7jU-cuCQItOkuBlAdaOvpBV5sU/edit?slide=id.g3e6f0b6583c_0_1579#slide=id.g3e6f0b6583c_0_1579))

1. Take a [“Prompting 101”](https://www.coursera.org/specializations/prompting-essentials-google) class.
2. Be specific.
3. Less is better.
  - Don’t attach every document or write every single thing.
  - Less + higher quality context > more but lower quality context
4. Add examples.
5. Use [prompt improvers](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-tools).
6. Don’t have super long conversations. Pro tip: Ask an LLM “give me the prompt to give to another LLM to continue our conversation”
7.  Ask questions in ways that the LLMs are trained on:
  - Look up SFT datasets to see what data LLMs are fine-tuned on. The more your queries look like this, the better.
  - Good: markdown, JSON, bulleted list.
  - Bad: free-form text, counting numbers, domain-specific file formats (e.g., .RData), new or less-used software (e.g., LLMs do better on Python than Stata).
8. Track what works!

Once you have the requisite skills for prompting, here are some practical pieces of advice that I have for prompting well, based off my own experience experimenting with how to prompt.

1. **Make sure that your task is as unambiguous as possible**. If you want the LLM to generate a label, make sure that that label has a well-defined set of distinct classes. If you want the LLM to generate free text, be specific about how long the free text should be, what sort of information should be in the free text, as well as categories for what belongs in the free text response.
2. **Add a few short examples at the end of your prompt**. Ideally, add a few examples of what you actually want. For example, if you want the LLM to generate labels presented as pairs of stimulus and a response, if you wanted to generate some kind of text, give examples of what good and bad text look like.
3. **Try to keep prompts shorter where possible**. Everything added to a prompt should earn its place. Additional information in a prompt, at the very least, just increases the number of tokens used, which increases the cost of your prompt. In the worst case, additional information might actually distract the LLM and therefore make it perform more poorly.
4. **When making prompts, think about how you would instruct a person to do this task**. Be as specific as possible while still maintaining clarity. Imagine if you had to pay a human annotator to do the thing that you want them to do. How specific would you need to make the instructions? What are the ways you can imagine this being interpreted or someone doing it incorrectly? Run through that exercise yourself to try to pin down what you want to put in the prompt.
5. **Start long, then cut. It's very likely that it'll be easier to make a long prompt than it is to make a short prompt**. It's okay to start long, but see what you can cut in order to still maintain performance. I suggest having a representative sample of what good and bad results look like. For example, you may have 10 posts that you want to label with an LLM. Make the prompts as long as possible first, and then start cutting things down until the performance begins to degrade. You can do a similar thing with generating example text, where you can actually have an LLM as a judge, grade the quality of the text based on some criteria you set. You can pick and choose parts of the prompt as needed until the quality goes down. Note that if you're going to use an LLM as a judge, you need to define in very strict terms what you're looking for.
6. **Use very specific binary criteria to figure out what looks good and what doesn't look good**. LLMs do best when they can be weighed against right or wrong criteria. They do more poorly on Likert scales or things that are more open-ended. See if you can define what good looks like when you ask as a series of binary questions, and use that to evaluate how good the responses are.
7. **Ask an LLM to rewrite your prompt, BUT always edit it yourself**: LLMs are quite good at being able to write their own prompts. If you want to improve your prompt, you can take your prompts and pass them to the LLM and ask it to improve them, or you can describe to the LLM your task and then have it write the prompt. Some caveats here include making sure that you still edit it yourself and you can own the prompt that the LLM designed. Sometimes the LLM's idea of the task doesn't match what you would have wanted, and that often comes from providing poor specification to the LLM itself. LLMs are great at writing their own prompts, but it doesn't absolve you of the responsibility of defining your intended task well in the first place.

We prompt LLMs so often that knowing how to prompt well turns out to actually be a high-leverage skill. Upskilling over even a weekend or two over the course of a month can have meaningfully high ROI on your ability to work well with AI applications.
