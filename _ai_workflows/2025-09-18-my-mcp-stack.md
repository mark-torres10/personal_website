---
layout: single
title:  "My MCP Stack: The Tools That Currently Power My AI Workflow"
date:   2025-09-18 08:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-09-18-my-mcp-stack
---

# My current MCP Stack: The Tools That Power My AI Workflow

I use Cursor as my primary development environment, and I quickly discovered that the power of Cursor is really unlocked when you connect it with external tools using MCPs. Here's a list of some that I use.

## My Current MCP Stack

### 1. Linear MCP
- **Purpose:** Project management and issue tracking
- **What it does:**
  - Create, update, and manage Linear issues
  - Track project progress and cycles
  - Manage team workflows and assignments
  - Search and filter issues across projects

This is actually pretty critical to my work. All of my work now is defined as Linear projects and tickets. I used to track projects and tickets with my own personal Notion workflow, but the Linear interface is so neat, they have great integrations with their MCP as well as with Github, and it's quick to set up. My [AI workflows](https://github.com/mark-torres10/ai_tools/blob/main/agents/task_instructions/project_management/PROJECT_PLANNING_EXECUTION_OUTLINE.md) all require AI agents to interface with Linear and update the projects and tickets with work progress, so Linear serves as the source of ground truth across each of my workflows. I even have specific instructions on what [writing a good Linear project](https://github.com/mark-torres10/ai_tools/blob/main/agents/task_instructions/project_management/HOW_TO_WRITE_LINEAR_PROJECT.md) as well as a [good Linear ticket](https://github.com/mark-torres10/ai_tools/blob/main/agents/task_instructions/project_management/HOW_TO_WRITE_LINEAR_TICKET.md).

### 2. Exa MCP

- **Purpose:** Web search and research
- **What it does:**
  - Perform real-time web searches
  - Crawl specific URLs for content

I know that Cursor uses their own web search tool (I think it's powered by Exa anyways, plus maybe another tool). But surprisingly I find much better results when I manually trigger the web search myself with Exa. I find that the queries that are run when I trigger the Exa MCP myself are higher quality than the ones that Cursor triggers with its own native web search tool and it also is lower latency.

### 3. Playwright MCP

- **Purpose:** Browser automation and testing
- **What it does:**
  - Automate browser interactions
  - Take screenshots and snapshots
  - Fill forms and navigate websites
  - Test web applications
  - Handle file uploads and downloads

Whenever I am building anything with a web interface, I have Playwright connect with my AI agent during development so that it can test and run its work. I also define testing scenarios in markdown files (e.g,. "open the page, then scroll, then click, then verify that you see XYZ") and this now becomes an "integration test" that I can run automatically, via an AI agent using the Playwright MCP, on any code change.

## Other MCPs I've considered but am not using

Here are a few other MCPs that I've tried but ended up not continuing to use.

### MCPs that I ended up not using enough

- Slack
- Notion
- Figma

### MCPs that have CLI equivalents anyways

For these ones, as it turns out the CLIs are pretty good already (LLM agents are great at using CLI commands, they can use `--help` to get the most up-to-date commands, and if needed, I can use the Exa MCP to get the most recent docs)

- Github
- Vercel
- Supabase

### Alternative Search MCPs

- Brave Browser
- Tavily

Exa was just far superior in terms of quality of results and the latency. I'm just amazed by how fast Exa is (I think they built their own search index from scratch).

## Summary

Here are some of the MCPs that I use in my workflow, why I use them, and how they help me. As time goes on, they'll surely evolve, especially as more and more companies release MCP servers (side note: I think that SDKs are going to go away as companies can more directly expose MCPs, which also fits into the notion of making your applications "AI-agent-friendly"). For now this stack serves me pretty well.
