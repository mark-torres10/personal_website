---
layout: single
title:  "Transactions in concurrent systems, explained"
date:   2025-11-10 08:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /self_education/2025-11-10-transactions-101
---

# Transactions in concurrent systems, explained

## What are transactions?

### What problems do transactions solve?

Doing multiple concurrent updates to shared state is fragile and prone to errors. Some problems that can come up include:

#### Problem 1: Crash mid-sequence

Imagine that you have to move $100 from Account A to Account B. In steps, that would look like:

1. Subtract $100 from Account A.
2. Add $100 from Account B.

But if you do these one at a time, if your app crashes after Step 1, you will have a very unhappy customer, A, who just lost $100, while B still hasn't gotten paid their money.

#### Problem 2: Lost updates (interleaving race condition)

Let's say you're booking airplane seats and you press "submit" at the same time that someone else does. One possible thing that can happen is that they process yours first, and then process the other person's submission, but you both get tickets saying you got seat 31A. When you both inevitably show up to claim seat 31A, their internal system will say that the other person got seat 31A because that person's submission overrode yours.

In parallel systems, concurrent writes to a shared resource are typically solved using something like [read-modify-write](https://en.wikipedia.org/wiki/Read-modify-write) operations. These generally work under some form of "check what value is currently set, and if that's what you expect, then override, else try again later" and is designed to resolve the problem of interleaving race conditions. But for updates to shared systems like databases, we need a higher level abstraction of the read-modify-write operations in low-level concurrent systems.

#### Problem 3: Write skew



#### Problem 4: Double effects

#### Problem 5: Read-write visibility leaks

#### What specific failure/interleaving patterns corrupt data when multiple operations must succeed together?

#### Why are naive fixes (retries, “best effort” cleanup, ad-hoc locks) insufficient at scale?

#### Which invariants do we actually care about preserving, and how do crashes break them?

#### How does this show up in real systems you build (filesystems, object stores, databases, stream processors, feature stores)?

## Impossibility results (Two Generals/FLP) and the scope of what transactions solve

(Check [this article](https://markptorres.com/self_education/2025-11-10-modern-day-two-generals-analogy) about the Two Generals problem)

## ACID semantics (and useful subsets)

TODO: have as a separate blogpost.

## Programming model vs implementation

## A Python demo showing the tradeoffs

(generic notes that I'll divvy up into separate articles later on)
Notes:

- Transactions:
  - Why do we need a new paradigm? What problem do transactions solve?
  - What are transactions?