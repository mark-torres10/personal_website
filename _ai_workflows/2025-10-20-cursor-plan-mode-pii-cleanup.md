---
layout: single
title: "10-minute PII cleanup using Cursor's Plan Mode"
date: 2025-10-20 10:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-10-20-cursor-plan-mode-pii-cleanup
---

# 10-minute PII cleanup using Cursor's Plan Mode

I recently needed to go through my entire codebase and remove any files that might contain PII (personally identifiable information). This is the kind of task that's tedious, error-prone, and time-consuming when done manually. You have to review every file, identify potential PII, decide what to remove, and then actually make the changes—all while making sure you don't break anything.

Manually, this would've taken hours. Even if I wrote a script to help, I'd still need to spend time scoping out the problem, writing the script, testing it, and then reviewing the results. I estimated that would take about an hour.

With Cursor's Plan Mode, the entire process took 10 minutes. Two minutes were spent on prompting and finalizing the plan. Eight minutes were spent waiting for Cursor to execute the plan, create a PR, and merge it.

This is a perfect example of how AI-assisted development can handle the kind of repetitive, well-defined tasks that drain your time and energy without actually requiring deep technical thinking.

## What is Cursor's Plan Mode?

For those unfamiliar, Cursor's Plan Mode is a feature that lets you describe a task in natural language, have the AI create a detailed execution plan, and then automatically implement that plan across your codebase. It's particularly good for tasks that involve:

- Multiple files or directories
- Systematic changes that follow a pattern
- Well-defined objectives with clear success criteria
- Tasks that would be tedious to do manually but don't require creative problem-solving

The key advantage over just using Chat or Composer is that Plan Mode explicitly creates a structured plan before execution. You can review the plan, ask for adjustments, and then let it run. This gives you confidence that the AI understands the task correctly before it starts making changes.

## My workflow: Removing PII from the codebase

Here's exactly how I used Plan Mode to clean up PII from my codebase:

### Step 1: Describe the task to Cursor

I started by describing what I needed in plain language. I didn't need to write a detailed specification or pseudocode—just a clear description of the goal and any constraints.

![My initial prompt to Cursor](/assets/images/2025-10-20-cursor-plan-mode-pii-cleanup/1.png)

The prompt was straightforward: "I need to remove any files in my codebase that might contain PII. This includes things like user data, email addresses, phone numbers, or any personally identifiable information. Please identify these files and remove them."

This is the kind of task description that would be way too vague for traditional programming. You'd need to define exactly what constitutes PII, specify which directories to search, determine file patterns, and so on. But with AI, you can start with a high-level goal and iterate.

### Step 2: Respond to clarifying questions

Cursor's Plan Mode didn't just blindly accept my initial prompt. Instead, it asked clarifying questions to make sure it understood the task correctly.

![Clarifications from Cursor and my responses](/assets/images/2025-10-20-cursor-plan-mode-pii-cleanup/2.png)

The AI asked things like:
- "Should I look in all directories or exclude certain folders like `node_modules` or `venv`?"
- "Do you want me to delete entire files or just remove PII content from files?"
- "Should I create a backup before making changes?"
- "What file types should I focus on?"

This back-and-forth is crucial. It's the AI's way of understanding edge cases and making sure the plan will actually solve your problem. I appreciate that it doesn't just make assumptions—it asks when it's uncertain.

My responses helped narrow the scope: yes, exclude dependency folders; yes, delete entire files if they're primarily PII; yes, create a backup; focus on data files, logs, and configuration files.

### Step 3: Review the generated plan

After the clarifications, Cursor generated a detailed execution plan. This is where Plan Mode really shines—you get to see exactly what it's going to do before it does anything.

![The plan that Cursor generated](/assets/images/2025-10-20-cursor-plan-mode-pii-cleanup/3.png)

The plan included:

1. **Scan the codebase** - Search through specified directories for files that might contain PII
2. **Identify PII patterns** - Look for email addresses, phone numbers, names in data files, etc.
3. **Create a backup branch** - Make sure we can roll back if needed
4. **Remove identified files** - Delete files that primarily contain PII
5. **Update references** - Fix any import statements or references to deleted files
6. **Run tests** - Verify that the codebase still works after deletions
7. **Create PR** - Open a pull request with all changes for review

This level of detail gave me confidence that Cursor understood the task. I could see each step, verify the logic made sense, and approve the plan before execution.

If the plan had issues, I could have asked for modifications at this stage. But it looked good, so I told it to proceed.

### Step 4: Watch Cursor execute the plan

Once I approved the plan, Cursor got to work. It went through each step systematically, showing its progress as it went.

![Cursor implementing the plan step by step](/assets/images/2025-10-20-cursor-plan-mode-pii-cleanup/4.png)

What I appreciated here is that Plan Mode doesn't just run blindly. It showed me:
- Which files it was examining
- What PII patterns it found
- Which files it decided to delete and why
- What changes it was making to references

This transparency is important. Even though I'm delegating the task to AI, I can still audit what it's doing and intervene if something looks wrong.

The execution phase took about 8 minutes. During this time, I could have context-switched to other work, but I chose to watch the progress to make sure everything looked reasonable.

### Step 5: Review the completion summary

After Cursor finished executing the plan, it provided a summary of what it accomplished.

![Cursor completing the plan and giving its summary](/assets/images/2025-10-20-cursor-plan-mode-pii-cleanup/5.png)

The summary included:
- **Files deleted**: 23 files containing PII
- **References updated**: 8 files with import statements or config references
- **Tests run**: All passing
- **Backup created**: Git branch with original state
- **PR created**: Ready for review and merge

This summary gave me everything I needed to verify the work was done correctly. I could see the scope of changes, confirm tests passed, and know that I had a safety net (the backup branch) if anything went wrong.

### Step 6: Review and merge the PR

Finally, Cursor created a pull request with all the changes. This follows standard development workflow—even though an AI did the work, it still goes through proper review before merging.

![The completed PR that Cursor generated](/assets/images/2025-10-20-cursor-plan-mode-pii-cleanup/6.png)

The PR was well-formatted with:
- A clear description of what changed
- A list of deleted files
- Explanation of why each file was removed
- Test results showing everything still works
- A note about the backup branch in case rollback is needed

I reviewed the PR, confirmed everything looked correct, and merged it. Total time from start to finish: 10 minutes.

## Why this approach works

This workflow demonstrates several key principles of effective AI-assisted development:

### It's efficient

A task that would've taken hours manually (or about an hour even with custom scripting) took 10 minutes. The time savings compound when you're dealing with multiple similar tasks across different projects.

The efficiency doesn't just come from speed—it also comes from reduced context switching. I didn't need to write code, debug edge cases, or manually verify each file. I described what I wanted, reviewed the plan, and let it run.

### It's systematic

Plan Mode forced a structured approach: clarify requirements, create a plan, execute, verify, document. This is better than ad-hoc manual work where it's easy to miss files or make inconsistent decisions.

The systematic nature also makes it reproducible. If I need to do this again on a different codebase, I can use a similar workflow and get consistent results.

### It's transparent

At every step, I could see what was happening and why. This builds trust in the AI's work and makes it easy to catch errors if they occur.

Transparency also makes debugging easier. If something breaks after the changes, I have a detailed log of exactly what changed and why, which makes it much faster to identify and fix issues.

### It's auditable

The PR, backup branch, and execution logs create a complete audit trail. Anyone reviewing the work can see exactly what changed and why.

This is particularly important for tasks like PII removal where you might need to demonstrate compliance or explain your process to stakeholders.

### It reduces cognitive load

Instead of tracking 23 files in my head, remembering which imports to update, and manually running tests, I could focus on the high-level task and let the AI handle the details.

This freed up my mental energy for the parts that actually matter: defining the problem correctly and verifying the solution works.

## When to use Plan Mode vs. other Cursor features

I've been using Cursor for a while now, and I've developed a sense of when to use different features:

### Use Plan Mode for:
- Multi-file refactoring tasks
- Systematic changes across the codebase
- Well-defined objectives that can be broken into steps
- Tasks where you want explicit planning before execution
- Situations where you need an audit trail

### Use Chat for:
- Quick questions about code
- Understanding existing code
- Debugging specific issues
- Exploring different approaches to a problem

### Use Composer for:
- Creating new features from scratch
- Making changes to a single file or small set of files
- Iterative development where requirements are evolving

For this PII cleanup task, Plan Mode was perfect because the objective was clear (remove PII), the scope was large (entire codebase), and I wanted explicit planning (to make sure nothing important was deleted).

## Comparison to my custom planning system

I've written before about my [custom planning system](https://github.com/mark-torres10/ai_tools/blob/main/agents/task_instructions/project_management/PROJECT_PLANNING_EXECUTION_OUTLINE.md) for complex projects. That system is designed for deep planning where requirements are ambiguous and you need extensive iteration.

Plan Mode is excellent for:
- Smaller, well-defined tasks
- Generally simpler workflows  
- Projects that need planning but don't require deep architectural thinking
- Quick iteration and execution

I still use my custom planning system for complex research projects or when I need extensive stakeholder input. But for day-to-day development tasks like this PII cleanup, Plan Mode is perfect and much faster to work with.

## The broader lesson: AI-assisted task delegation

This PII cleanup is a specific example of a broader principle: AI tools are incredibly good at handling repetitive, well-defined tasks that don't require creative problem-solving.

The key is knowing how to delegate effectively:

1. **Start with a clear objective** - "Remove PII from the codebase" is clear and measurable
2. **Be willing to iterate** - Answer clarifying questions to refine the plan
3. **Review before execution** - Check the plan makes sense before letting it run
4. **Verify the results** - AI can make mistakes, so always review the output
5. **Document the process** - The PR and commit history serve as documentation

This approach works for all kinds of tasks: code migrations, dependency updates, test creation, documentation generation, and more. The pattern is the same—describe what you want, let AI plan the execution, review the plan, and let it run.

## Conclusion

Cursor's Plan Mode turned a multi-hour task into a 10-minute automated process. The key was having a well-defined objective (remove PII), being willing to clarify requirements (answering the AI's questions), and trusting the structured plan (but still reviewing the results).

This is the kind of AI-assisted development that actually works in practice. You're not replacing human judgment—you're still defining the problem, reviewing the plan, and verifying results. But you're delegating the tedious execution work to AI, which frees you up to focus on higher-level thinking.

The time savings are real and significant. But more importantly, this approach reduces the cognitive load of repetitive tasks. Instead of spending mental energy on manual file review and deletion, I could focus on making sure the task was defined correctly and the results met my requirements.

If you're using Cursor and haven't tried Plan Mode yet, I highly recommend it for these kinds of systematic, multi-file tasks. It's particularly valuable for work that's important enough to do carefully but tedious enough that you'd rather not do it manually. The combination of explicit planning, transparency, and automation makes it one of the most useful AI development features I've used.