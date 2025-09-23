---
layout: single
title:  "Using AI to create simple demos and to get unstuck quickly"
date:   2025-09-22 08:00:00 +0800
classes: wide
toc: true
categories:
- ai_workflows
- all_posts
permalink: /ai_workflows/2025-09-22-create-simple-demos-with-ai
---

# How I use AI to create simple demos and get unstuck quickly

[problem statement]

**Problems with typical AI development workflows**

**What I'd like to do**

*I realized that I can actually use AI to build a demo very quickly in order to figure the core concepts*

(I create plans)

(But instead of having the AI just go off and do it, I have it first create a simple demo version. That way, I can test the code logic on well-constrained data, verify training logic, check if file exports make sense, and see if code functionality and analysis of results make sense)

(More generally, it's really good for demo-driven development, which IDK if it exists generally so I'll have to look, but I like it as a complement to test-driven development)

(It's definitely a framework that I'll keep in mind for executing further workflows)

(TODO: can I add this to my AI agent tool stack?)

## Where does a normal AI-driven workflow fall short?

## Example use case

I'm currently doing some analysis related to hashtag usage in social media text. This task happens to be pretty LLM-friendly, as the complicated parts (data loading, data exports, and visualizations) have common patterns that I've used elsewhere so I have common utilities as well as example code elsewhere to pass to Cursor, and the only novel part (counting the number of unique hashtags) is a pretty unique algorithm.

However, a common experience for even such a simple task is that there can be small one-off bugs that happen (incorrect indexing, incorrect filenames referenced, etc). There are also cases where the code does compile, but something about the output doesn't look right (for example, I might not like the visualizations).

In this case, a pattern that has worked for me is this:
1. I have a spec defined in a ticket that I've defined.
2. I have the AI agent implement the spec.
3. I have it create unit tests to verify functionality.

### Shortcomings of AI-powered test-driven development (TDD)

Test-driven development (TDD) is one of the most common software development frameworks, a best practice for mature codebases (but, like always, you have to know how to bend the rules). It just so happens that testing is a great application to be able to use AI agents. AI agents are pretty good at generating a diverse set of test cases. However, I've found that there are a few shortcomings when just trying to blindly use testing with agents doing their own work:

1. The tests that are generated are often pretty trivial tests that don't actually test interesting or plausible control flows.
2. The tests themselves can be insufficiently comprehensive.
3. Even if the tests themselves test the individual functionalities, AI agents tend to struggle in creating what I consider to be more comprehensive integration tests. This is especially true since the applications that I generally care about are harder to define with a simple test.

### Improvement 1: For web apps, integrating both the terminal logs and Playwright MCP into the testing framework

One improved approach that I've observed is integrating terminal logs and Playwright MCP into a testing suite. I now have the AI agent create a checklist of expectations for both the front-end and the back-end. This can be a checklist of things to see or things to verify in the UI, as well as things to verify in the logs, things that exist or don't exist. Essentially assertions, but in a natural language flow. This serves two purposes:

1. It significantly improves the consistency of the verification process of the AI agents
2. As a human, it helps me to inspect the results myself and double-check the results

Here's one example of how that looks for frontend expectations.
```markdown
### Phase 2: Basic Functionality Testing

#### 2.1 Test Home Page (Unauthenticated)
- [x] **Navigate to**: `http://localhost:3000`
- [x] **Expected**: Should show landing page with "Get Started" button
- [x] **Verify**: No NEXT_REDIRECT errors in browser console (F12 → Console tab)
- [x] **Check**: Page loads without any JavaScript errors

#### 2.2 Test Home Page (Authenticated)
- [x] **Sign in** to the application
- [x] **Navigate to**: `http://localhost:3000`
- [x] **Expected**: Should show landing page with "Go to Dashboard" button
- [x] **Verify**: No NEXT_REDIRECT errors in browser console
- [x] **Check**: Button should link to role selection if no role assigned

### Phase 3: Sign-out Functionality Testing

#### 3.1 Test Sign-out from Role Selection Page
- [x] **Navigate to**: `http://localhost:3000/role-selection`
- [x] **Look for**: "Sign Out" button in the top-right header area
- [x] **Click**: "Sign Out" button
- [x] **Expected**: Should sign out and redirect to home page (not 404 error)
- [x] **Verify**: You're now on the landing page and can sign in again
```

Here's one for the backend:
```markdown
2. **Test Complete Authentication + User Management Flow**
   - [ ] Sign in with Clerk (Google/Microsoft/Email)
   - [ ] Verify user is redirected to home page
   - [ ] Test user creation through API
   - [ ] Test user retrieval through API
   - [ ] Test user update through API
   - [ ] Test sign out and verify redirect to login

3. **Test API Endpoints with Authentication**
   - [ ] Test `POST /api/users` to create user record
   - [ ] Test `GET /api/users/[clerkUserId]` to retrieve user
   - [ ] Test `PUT /api/users/[clerkUserId]` to update user
   - [ ] Test `GET /api/users` to list all users (admin only)
   - [ ] Test `DELETE /api/users/[clerkUserId]` to delete user
```

### Improvement 2: Creating quick AI-powered demos

However, for cases that are more for ML or data science or data pipelines, this workflow generally proves insufficient because you care about the quality of the data and the outputs that you could in your analysis. There are ways to be able to automate testing with data such as frameworks like Great Expectations, but I also care about the analysis that's generated and being able to pick the correct analysis frameworks, being able to test the correct statistics, being able to create the correct graphs, make them presentable and understandable and the like. In addition, I also often want to quickly QA any generated code from the AI agent with an integrated test.

(put example of demo here and how that looks).

## Creating a demo for our example use case

For the hashtag analysis, I created a simple demo to help me with quick development.

## What works well for me?

## Caveats

## Takeaways
