---
layout: single
title: "Is This Worth Automating? Here's How to Tell"
date: 2025-12-17 13:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-12-17-is-this-worth-automating-with-ai
---

# Is This Worth Automating? Here's How to Tell

I've worked with many clients who want to automate everything with AI. They see a task that takes time, assume AI can handle it, and jump straight to building automation. But not every task is worth automating, and trying to automate the wrong things leads to wasted effort, unreliable systems, and sometimes costly mistakes.

The question isn't "Can AI do this?", since modern AI can do a surprising amount. The real question is: "Should we automate this, and if so, how much human oversight do we need?"

After working with clients across different industries and use cases, I've developed a simple framework to answer this question. It evaluates tasks on two dimensions: **standardizability** and **verifiability**. Understanding where your task falls on these axes tells you whether to fully automate, use human-in-the-loop augmentation, or avoid automation for now entirely.

## The Framework: Two Dimensions

I evaluate AI automation on two dimensions:

1. **Standardizability**: Can we define clear rules and patterns? (vs. requiring judgment)
2. **Verifiability**: Can we quickly verify outputs, and what's the cost of errors?

Tasks in the high/high quadrant are good automation targets. Others benefit from human-in-the-loop augmentation.

| | High Verifiability | Low Verifiability |
|---|---|---|
| **High Standardizability** | ✅ **Best candidates for full automation** | ⚠️ **Good with human-in-the-loop** |
| **Low Standardizability** | ⚠️ **Good for augmentation** | ❌ **Avoid full automation** |

## Understanding Standardizability

**High standardizability** means the task has:

- Clear rules that can be written down
- Consistent patterns that repeat
- A teachable process that doesn't require judgment
- Predictable inputs and outputs

**Low standardizability** means the task:

- Requires judgment calls
- Has variable inputs that change significantly
- Is context-dependent
- Needs interpretation or creative problem-solving

Think of it this way: if you could teach a smart intern to do this task by writing clear instructions, it's highly standardizable. If you'd need to sit with them, explain context, and make judgment calls together, it's less standardizable. A task should be doable by someone who doesn't have domain-specific expertise or intuition for it to be standardizable.

### Standardizability Examples

**High standardizability:**

- Converting PDFs to Excel format (same structure every time)
- Extracting data from financial statements (consistent format)
- Scheduling and formatting social media posts from a content calendar (repetitive pattern)

**Low standardizability:**

- Making investment decisions (requires judgment, market knowledge, risk assessment)
- Complex due diligence analysis (context-dependent, needs interpretation)
- Writing suggested messaging (creative, audience-dependent)

## Understanding Verifiability

**High verifiability** means:

- Easy to spot-check outputs
- Low cost of errors (mistakes are cheap to fix)
- Clear success criteria
- You can quickly tell if something is wrong

**Low verifiability** means:

- Hard to verify outputs (requires expertise or time)
- High cost of errors (mistakes are expensive)
- Subjective outputs (what's "good" is debatable)
- Errors might not be obvious until much later

The cost of errors matters here. A formatting mistake in a file converted from PDFs is easy to spot and fix. A mistake in an investment decision or poor marketing copy to a large social media audience could cost thousands or more, and you might not realize it's wrong until it's too late.

### Verifiability Examples

**High verifiability:**

- Data extraction (you can quickly check if numbers match the source)
- Formatting tasks (you can visually verify the output looks correct)
- Generating social media performance reports (standard templates, easy to cross-check metrics)

**Low verifiability:**

- Investment decisions (hard to know if a decision was "right" until much later)
- Complex analysis (requires expertise to verify, subjective interpretation)
- Strategic recommendations (outcomes depend on many factors, hard to attribute success/failure)

## The Four Quadrants: What to Do in Each

### High Standardizability + High Verifiability: Best Candidates for Full Automation

These are your automation targets. The task has clear rules, and you can easily verify the output is correct.

**Finance examples:**

- PDF → Excel formatting
- Data extraction and normalization from financial documents
- Monthly statement formatting into trackers

**Software engineering examples:**

- Code formatting and linting
- API documentation generation from code comments
- Unit test boilerplate generation (with human review of test logic)
- Dependency version updates in configuration files

**Marketing examples:**

- Social media post scheduling
- Email list segmentation based on clear criteria
- A/B test data collection and basic reporting

**What to do:**

- Build full automation with minimal human oversight.
- Set up spot-checking (maybe 10-20% of outputs).
- Monitor for edge cases, but trust the system for routine cases.

The hardest parts here are defining the task clearly and in defining what is success. Otherwise, for these tasks, AI automation can generally be trusted to do them consistently and correctly. These cases can also be often augmented with automated tests (or LLM-as-a-judge AI agents) to help improve the consistency of evaluating correctness over time.

### High Standardizability + Medium/Low Verifiability: Good with Human-in-the-Loop

The task has clear rules, but errors could be costly or hard to spot. You want automation to do the heavy lifting, but you need human verification.

**Finance examples:**

- Due diligence grunt work (with spot checks)
- Performance tracking data entry (with validation)
- Financial report generation (with human review of calculations)

**Software engineering examples:**

- Code review assistance (AI flags potential issues, human makes final call)
- Bug report triage and categorization
- Automated dependency updates (with human review of breaking changes)

**Marketing examples:**

- Content calendar generation (with human approval of timing and topics)
- Campaign performance reporting (with human validation of metrics)
- Lead scoring automation (with human review of scoring rules)

**What to do:**

- Automate the repetitive parts
- Build in human review checkpoints
- Use AI to flag uncertain cases for human review
- Start with higher oversight, reduce as confidence grows

### Medium/Low Standardizability + High Verifiability: Good for Augmentation

The task requires judgment, but you can quickly verify if the output is good. AI can help, but humans make the final decisions.

**Finance examples:**

- AI notetaking during meetings (human reviews and edits)
- Email drafting for client communications (human edits before sending)
- Financial analysis summaries (human verifies accuracy and context)

**Software engineering examples:**

- Code generation (human reviews and tests thoroughly)
- Architecture suggestions and design patterns (human evaluates fit)
- Debugging assistance (human verifies solutions work)
- Technical documentation writing (human reviews for accuracy)

**Marketing examples:**

- Content creation (blog posts, social media—human edits before publishing)
- Ad copy generation (human reviews and approves)
- Blog post outlines and first drafts (human refines and finalizes)
- Email campaign copy (human edits for brand voice and compliance)

**What to do:**

- Use AI to generate first drafts or suggestions
- Always have human review and editing
- Treat AI as a starting point, not the final output
- Build workflows that make it easy to review and refine

### Low Standardizability + Low Verifiability: Avoid Full Automation

These tasks require judgment, and errors are costly or hard to verify. Full automation is risky. AI can help serve as a resource or as a companion for iterating on ideas, but *avoid automating these and trust human expertise instead*.

**Finance examples:**

- Investment decisions
- Complex due diligence analysis
- High-stakes financial advice
- Strategic portfolio allocation decisions

**Software engineering examples:**

- System architecture decisions
- Technology stack choices for new projects
- Security-critical code implementation
- Performance optimization strategies (context-dependent)

**Marketing examples:**

- Brand strategy and positioning
- Campaign strategy and creative direction
- Messaging strategy and brand voice decisions
- Market positioning and competitive strategy

**What to do:**

- Use AI for research and information gathering only
- Keep humans in full control of decisions
- Use AI to augment human judgment, not replace it
- If you do use AI, build extensive safeguards and review processes

## How to Apply This Framework

When evaluating a task for automation, ask yourself:

1. **Standardizability questions:**

   - Can I write clear instructions for this task?
   - Does the task follow consistent patterns?
   - Are the inputs predictable, or do they vary significantly?
   - Does this require judgment, or can it be rule-based?

2. **Verifiability questions:**

   - How quickly can I tell if the output is correct?
   - What's the cost if the output is wrong?
   - Do I have clear success criteria?
   - Can I spot-check this, or does it require deep expertise to verify?

3. **Then decide:**

   - High/High → Full automation with spot checks
   - High/Low → Automation with human review
   - Low/High → AI augmentation (human in control)
   - Low/Low → Avoid automation, use AI for research only

## Common Mistakes to Avoid

### Mistake 1: Automating everything because you can

- Just because AI *can* do something doesn't mean it *should* do it autonomously. AI is very good at giving you plausible-looking slop (even if it's been trained not to do so). This doesn't mean that AI isn't impressively performant, but more that it doesn't do it the way that *you* know it should be done.
- Low verifiability tasks need human oversight, even if they're standardizable.

### Mistake 2: Treating all tasks the same

- Different tasks need different approaches.
- A task that works well with full automation might be completely different from one that needs human-in-the-loop.
- Even the same task across different teams and companies might be treated differently, depending on team and management dynamics and differences in constraints.

### Mistake 3: Ignoring the cost of errors

- A task might be highly standardizable, but if errors are costly, you still need oversight. A 1% error rate is OK if the cost is a typo in a document, but is unacceptable if the cost is a $1,000,000 fee from the SEC.
- The verifiability dimension matters just as much as standardizability.

### Mistake 4: Starting with full automation

- Even for high/high tasks, start with human oversight and reduce it as you build confidence.
- You'll discover edge cases and failure modes you didn't anticipate.
- Starting with full automation generally leads to people being frustrated that AI, for all the hype it receives in the press, isn't a one-shot solution for their use cases. I've seen this lead some to stop using AI altogether, which is the wrong takeaway.

## Conclusion

Not every task is worth automating, and not every automation should be fully autonomous. The key is matching your automation approach to the task's characteristics.

Tasks that are highly standardizable and highly verifiable are your best bets for full automation. Tasks that are standardizable but harder to verify need human oversight. Tasks that require judgment but are easy to verify work well as AI augmentation. And tasks that require judgment and are hard to verify should avoid full automation.

The framework is simple, but applying it consistently helps you avoid the common mistake of trying to automate everything or being too cautious about automation. It also helps you build the right level of human oversight, not too much (wasting the benefits of automation) and not too little (risking costly errors).

Consider also that defining tasks in this way is a moving target. What might be considered a task low on the standardization matrix might very well be easily standardized as AI intelligence improves (e.g., writing personalized copy). What might be considered an error-prone task now (e.g., content creation, automated debugging) might also improve with AI intelligence gains. It's often worth having [your own prompt library](https://markptorres.com/ai_workflows/2025-12-05-writing-your-own-prompt-library) so you can continuously evaluate new AI models.

Start by evaluating your current tasks on these two dimensions. You might find that some tasks you thought needed full automation actually work better with human-in-the-loop, or that some tasks you were doing manually are perfect candidates for automation. The goal isn't to automate everything. Instead, it's to automate the right things, in the right way, with the right level of human oversight.
