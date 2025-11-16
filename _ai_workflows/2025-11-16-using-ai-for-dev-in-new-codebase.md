---
layout: single
title: "Onboarding to a New Codebase with Cursor Agents and GPT-5"
date: 2025-11-16 10:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-11-16-using-ai-for-dev-in-new-codebase
---

# Onboarding to a New Codebase with Cursor Agents and GPT-5

I recently picked up working on a new codebase, and I didn't have much context beyond seeing what the final application should look like. I used Cursor and GPT-5 to greatly expedite my onboarding, cutting what was once a task that took many hours into something that took me 30 minutes.

![Query to GPT5](/assets/images/2025-11-16-using-ai-for-dev-in-new-codebase/1.png)

![Resulting doc](/assets/images/2025-11-16-using-ai-for-dev-in-new-codebase/2.png)

I then used this resulting doc as the basis for probing the repo with the help of GPT5:

![Probing the codebase more](/assets/images/2025-11-16-using-ai-for-dev-in-new-codebase/3.png)
![Getting up-to-speed on the database layer](/assets/images/2025-11-16-using-ai-for-dev-in-new-codebase/4.png)

Next, I asked GPT-5 to learn about the coding conventions used in the repo, including the language, frameworks, static type checks enforced, testing policies, etc. I also asked it to see if there were any collaboration conventions also enforced, such as a specific ticketing system or GitHub PR rules as well as particular stakeholders to ask about the code.

![Getting more context](/assets/images/2025-11-16-using-ai-for-dev-in-new-codebase/5.png)
![Repo coding conventions](/assets/images/2025-11-16-using-ai-for-dev-in-new-codebase/6.png)

Given all this, I had the information I needed to start working in a new codebase! What would've taken a couple hours a day over the course of weeks to do when starting in a new codebase, AI greatly expedites. I can now quickly get up-to-speed on a codebase, know how it works, understand what was developed and when, and have enough context to start contributing within an hour of getting started.
