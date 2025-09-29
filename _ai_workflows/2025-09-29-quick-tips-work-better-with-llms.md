---
layout: single
title: "Quick tips that help me work better with LLMs"
date: 2025-09-29 05:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-09-29-quick-tips-work-better-with-llms
---

# Quick tips that help me work better with LLMs

## 1. Use a Prompt Rewriter
- Claude and OpenAI have great prompt rewriters, such as [Claude's Prompt Improver](https://www.anthropic.com/news/prompt-improver).
- These tools help refine your prompts for better outputs

## 2. Verify with Online Examples
- When you doubt what ChatGPT says, ask it to find examples online and prove itself
- More often than not, it corrects its own analysis if online resources don't back it up
- Even if it doesn't self-correct, you get ground truth examples

## 3. Challenge Your Biases
- Ask ChatGPT (or any chat LLM) how your bias or beliefs could be wrong
- LLMs are biased towards affirming what you already believe
- Asking them to point out how you could be wrong is tremendously helpful

**Side note 1:** Take output from one LLM and pass it to another LLM asking how it could be wrong (Grok is great for this). Then pass both outputs to a third LLM and ask it to be a judge and give a final synopsis.

**Side note 2:** Ask LLMs to give 3 reasons pro/against a particular take.

## 4. Explore Conditional Scenarios
- Besides giving a recommendation, ask ChatGPT under what circumstances or changes would its recommendation change
- This way you get both a recommendation and know what would have to be different for its advice to change

## 5. Start Fresh Conversations
- Start a new chat window if you want ChatGPT to view your problem with fresh eyes
- If you need context from a previous conversation, ask ChatGPT to summarize and compress your conversation so you can copy and paste it as context to the next conversation

## 6. Define Terms Very Specifically
- Define terms very very specifically
- Don't assume the LLM knows what you mean by vague terms
- Be explicit about definitions, especially for technical or domain-specific concepts

## 7. Add Examples
- Add examples instead of using vague descriptors
- Instead of saying "good writing", add specific examples like "in the style of a journalist reporting on tech earnings" or "in the style of a NYTimes opinion piece"
- Concrete examples help the LLM understand exactly what you want

## 8. Embrace Context Engineering
- The more you embrace context engineering, the better chance the LLM can do its job right
- Use [personas](https://github.com/mark-torres10/ai_tools/tree/main/agents/personas) to guide the LLM to view a problem from a very specific point of view
- Include only relevant context
- Start new conversations if the current one has gone for too long