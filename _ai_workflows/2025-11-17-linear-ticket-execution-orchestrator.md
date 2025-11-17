---
layout: single
title: "How I use AI agents to automate shipping complex features"
date: 2025-11-17 11:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-11-17-linear-ticket-execution-orchestrator
---

# How I use AI agents to automate shipping complex features

I built a [comprehensive orchestrator prompt](https://github.com/mark-torres10/ai_tools/blob/main/agents/task_instructions/execution/HOW_TO_EXECUTE_A_TICKET.md) to manage how agents execute tickets end‑to‑end, and I use it regularly to deliver and ship features. After shipping a lot of features with AI assistance, I’ve converged on an opinionated workflow that works for both me and the agents that follow the same scaffolding.

## Why I built this

Unsupervised agents drift. They start coding before a plan is approved, forget context, and submit PRs that are hard to review. Humans fix this with experience and process; agents need that process made explicit and checkable.

Cursor's "Plan Mode" is a good place to start, but I wanted something more systematic and also more tuned to my own opinionated workflows.

The orchestrator solves a few recurring pain points:

- Ambiguity at the start of a ticket: this forces a lightweight plan that gets explicit approval before any coding.
- Context loss over long threads: this writes state to durable artifacts (`spec.md`, `plan_<feature>.md`, `tickets/`, `logs.md`).
- PR sprawl and weak testing: enforces TDD-first, success criteria validation, and linkable evidence.
- Review friction: standardizes PR hygiene and review notes so reviewers get the right context fast.

## What this enables

- Safe delegation to agents without babysitting, because progress must pass gates with evidence.
- Faster reviews, because PRs link the spec, ticket doc, and test report in a consistent format.
- Repeatable execution across tickets and teams; onboarding becomes “read the artifacts,” not “DM me.”
- CI-friendly compliance checks; the Adherence Block can be parsed to fail a run when gates aren’t met.
- Better institutional memory; decisions and outcomes live next to the work, not lost in chat.

Developers of coding agents likely have similar workflows internally (e.g., I assume that Cursor has their own complex memory and context engineering stack), but this one is lightweight and explainable enough to be generally useful for my cases.

## What this does well

- **1-page checklist**: A concise checklist drives execution; the long doc explains rationale and edge cases.
- **Built-in state and gates**: Each turn includes a tiny “Adherence Block” JSON (current step, artifacts, blockers). Hard gates (Plan Approved, Spec Compliance Passed, Report Linked) stop drift.
- **Project artifacts as source of truth**: The agent reads `spec.md`/`braindump.md` and continuously updates `tickets/ticket-<id>.md`, `plan_<feature>.md`, `todo.md`, `logs.md`, `lessons_learned.md`, `metrics.md`, and `testing_reports/` so evidence lives with the work.
- **TDD-first with automation hooks**: Encourages tests-first development, coverage thresholds, and scripted report generation. It’s set up to plug into CI from day one.
- **PR and completion hygiene**: Opinionated templates and link requirements (Linear, ticket doc, spec, test report) make reviews fast and auditable.
- **Operational guardrails**: Clear “Red Flags” and “Common Pitfalls” reduce failure modes and enable lightweight automated checks.

### Example Adherence Block

```json
{"step_id":"6_test_and_validate","checklist_completed":["1","2","3","4","5"],"artifacts_written":["projects/.../testing_reports/report.md"],"blockers":[],"awaiting_approval":false}
{"step_id":"6_test_and_validate","checklist_completed":["1","2","3","4","5"],"artifacts_written":["projects/.../testing_reports/report.md"],"blockers":[],"awaiting_approval":false}
```

## What does the workflow look like?

Each step in the process is clear and well-defined and follows a repeatable process of (1) know the task goal, (2) do the task, (3) record the artifacts of the work, and (4) have a gate to check the work progress.

1) **Analyze the ticket and gather context**  
   - **Goal**: Align on scope and constraints.  
   - **Do**: Pull Linear details, scan related PRs, read `spec.md` and `braindump.md`, check dependencies and environment.  
   - **Artifacts**: `logs.md` notes; discrepancies documented.  
   - **Gate**: Plan must be drafted and approved before coding.

2) **Implementation planning**  
   - **Goal**: Make the path explicit and testable.  
   - **Do**: Break work into steps in `plan_<feature>.md`; create/sync `tickets/ticket-<id>.md` with acceptance criteria.  
   - **Artifacts**: `plan_<feature>.md`, `tickets/ticket-<id>.md`.  
   - **Gate**: APPROVED PLAN (explicit user sign-off).

3) **Development setup**  
   - **Goal**: Reproducible, isolated work.  
   - **Do**: Create feature branch, activate env, run baseline tests.  
   - **Artifacts**: Branch name follows convention; notes in `logs.md`.

4) **TDD (write tests first)**  
   - **Goal**: Lock in behavior and prevent regressions.  
   - **Do**: Unit + integration tests with coverage targets; name tests for intent.  
   - **Artifacts**: Test files; coverage reports.

5) **Implementation**  
   - **Goal**: Ship minimal, clean code. Avoid verbosity and try to touch only the bare minimum necessary code that needs to be touched. 
   - **Do**: Follow quality rules (types, SRP, small functions), logging and error handling.  
   - **Artifacts**: Code changes; inline docs.

6) **Testing & validation**  
   - **Goal**: Prove it works against the spec. Test cases and user journeys should be well-defined in the spec, so this would involve (1) creating unit/integration tests, as well as, if relevant, (2) using relevant APIs/test runners/Playwright to simulate end-to-end tests.
   - **Do**: Run full test suite; validate success criteria from `spec.md`; do manual checks for UX/perf/edge cases.  
   - **Artifacts**: Test results; `logs.md` validation notes.  
   - **Gate**: SPEC COMPLIANCE PASSED.

7) **Comprehensive testing (recommended)**  
   - **Goal**: Production-level confidence. Make sure testing is done and matches the testing specs, and make it as comprehensive as appropriate for the scope.
   - **Do**: Use the experiments/testing guide; generate a report in `testing_reports/`; link scripts and results.  
   - **Artifacts**: `testing_reports/<report>.md`; scripts.  
   - **Gate**: Testing report linked.

8) **Incremental commits**  
   - **Goal**: Reviewable history tied to Linear.  
   - **Do**: Small commits with clear messages and issue links.

9) **Code review preparation**  
   - **Goal**: Accelerate reviewer time-to-understanding.  
   - **Do**: Prep architecture explanation + review context (critical files, order, risks).  
   - **Artifacts**: Review notes; link in PR.

10) **PR creation**  
    - **Goal**: Clear, complete submission.  
    - **Do**: Use PR template; link Linear issue, `tickets/ticket-<id>.md`, `spec.md`, and testing report.  
    - **Artifacts**: PR with required links.

11) **PR management**  
    - **Goal**: Iterate quickly and transparently.  
    - **Do**: Address feedback with focused commits; re-request review; keep PR updated.

12) **Completion & cleanup**  
    - **Goal**: Close the loop with evidence.  
    - **Do**: Update `todo.md`, `logs.md` (summary + PR link), `lessons_learned.md` (3–5 bullets), `metrics.md` (lead/cycle times), and ensure the ticket doc reflects outcomes.  
    - **Artifacts**: Updated project files; Linear marked complete.

13) **Post‑completion tasks**  
    - **Goal**: Institutional memory.
    - **Do**: Record estimates vs actuals; share patterns; propose standards updates.

14) **Retrospective**  
    - **Goal**: Improve the system, not just the code. Add any lessons learned, hard tasks that took time and iterations to improve, etc. as I want my codebase to learn how to improve itself over time.
    - **Do**: Create `retrospective/{ticket}.md`, index it in `retrospective/README.md`, and add it to the PR + Linear thread.  
    - **Artifacts**: Retrospective doc; follow-up actions.

## Why this matters

I have some consistent workflows that I use when I ship a feature, and I also use AI agents a lot in my work as well. Having this rigorously defined, opinionated work orchestrator helps me be consistent in execution, track what the AI agents are doing, and allow me to focus on managing agents rather than writing the code myself. A lot of the upfront work is now done in the planning/spec development as well as in testing and verification, and one should take caution in trying to automate away these critical components. But for coding, I'm leaning more and more towards spec-driven development.

## Example: Shipping an ML pipeline feature end‑to‑end

Scenario: We need to add a new feature from the feature store, retrain the model, and deploy the new model to production.

### Ticket breakdown

- Ticket 001 — Add feature from feature store  
  - Acceptance criteria: Feature `user_7d_active_minutes` is materialized and available in training and online inference; data quality checks pass; schema and contract documented.  
  - Evidence: `tickets/ticket-001.md`, `plan_feature_add.md`, DQ report in `testing_reports/feature_add.md`, updated `spec.md` with schema.

- Ticket 002 — Retrain model with new feature  
  - Acceptance criteria: Training pipeline runs reproducibly; metrics improve or tradeoffs justified; model card updated; artifacts versioned.  
  - Evidence: `tickets/ticket-002.md`, `plan_retrain.md`, training report in `testing_reports/retrain.md`, registered model version with metadata.

- Ticket 003 — Deploy model  
  - Acceptance criteria: Staging canary passes; latency/SLOs meet thresholds; rollback plan defined and tested; production deployment completed.  
  - Evidence: `tickets/ticket-003.md`, `plan_deploy.md`, validation report in `testing_reports/deploy.md`, PR links to dashboards/runbooks.

### Applying the flow (condensed)

1) Analyze and gather context  
   - Read `spec.md` for feature definition, current pipeline constraints, and success criteria (e.g., AUC +0.5pp with no P95 latency regression).  
   - Pull prior PRs for feature ingestion and last deployment. Log assumptions and risks in `logs.md`.

2) Implementation planning  
   - Create `plan_feature_add.md`, `plan_retrain.md`, `plan_deploy.md` with steps, dependencies, and gates.  
   - Draft acceptance criteria in each `tickets/ticket-<id>.md`.  
   - Gate: APPROVED PLAN.

3) Development setup  
   - Branch conventions: `feat/feature-store-user-7d-mins`, `ml/retrain-v34`, `deploy/model-v34`.  
   - Baseline tests pass; capture environment notes in `logs.md`.

4) TDD (tests first)  
   - Feature add: write ingestion unit tests + schema checks; add DQ assertions (nulls, drift, range).  
   - Retrain: add reproducibility test (fixed seed, stable metrics); contract tests for feature availability.  
   - Deploy: write smoke/canary checks and latency budgets as tests or monitors linked from repo.

5) Implementation  
   - Ticket 001: Wire feature store read, transform, and materialization; update schema contracts and docs.  
   - Ticket 002: Update training config, include feature, track experiments, store metrics and artifacts.  
   - Ticket 003: Promote model to staging, validate canary, prepare rollout + rollback scripts.

6) Testing & validation  
   - Run full test suite; generate DQ, training, and deployment reports under `testing_reports/`.  
   - Gate: SPEC COMPLIANCE PASSED for each ticket.

7) Comprehensive testing (recommended)  
   - Backfill sample to detect historical inconsistencies; run load/latency checks on staging; attach evidence to reports.  
   - Gate: Testing report linked.

8–11) Commits, review prep, PR creation and management  
   - PRs must link: Linear ticket, `tickets/ticket-<id>.md`, relevant `plan_*.md`, and `testing_reports/*.md`.  
   - Include reviewer context: critical files, risks (e.g., feature drift), rollback steps.

12–14) Completion, post‑completion, retrospective  
   - Update `todo.md`, summarize in `logs.md`, add 3–5 bullets in `lessons_learned.md`, record cycle/lead time in `metrics.md`.  
   - Create `retrospective/ticket-003.md` covering deployment outcomes and follow‑ups (e.g., monitor thresholds).

### Sample Adherence Block (Ticket 003 — Deploy)

```json
{"step_id":"10_pr_creation","checklist_completed":["context_prepared","links_added","risk_notes_included"],"artifacts_written":["tickets/ticket-003.md","plan_deploy.md","testing_reports/deploy.md"],"blockers":[],"awaiting_approval":true}
```

## Some caveats in practice

- I still have to nudge the LLMs sometimes to track and record their progress to the relevant files.
- This still requires manual oversight over the process. Guardrails are great, but if you, for example, rely on LLMs to generate 100% of your test cases, then it can also generate trivial use cases.
- The hardest work is done in the planning and spec development steps. The more time you spent on that, the better the results are.
- I still haven't cracked how to best handle workflows that don't 100% follow the linear pattern, such as those that are more (1) experimental/exploratory, or (2) those that require lots of revisions/iterations. You can iteratively make tasks more narrowly defined as one way of combatting this, but it does force a very specific workflow that might not be ideal for, say, a data scientist doing EDA.
