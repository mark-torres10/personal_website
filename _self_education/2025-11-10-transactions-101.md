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

A transaction is a bundle of operations grouped together to be treated as a single, atomic unit of work within a system, meaning it either completes entirely, or has no effect at all. Transactions ensure consistency and correctness in environments where multiple operations must act together, especially under concurrent access, partial failures, or retries. They provide guarantees that shared data reflects only successful, complete changes and prevent systems from ending up in partial, invalid, or inconsistent states.

### What does a transaction look like?

#### 1. Technical Example

Consider a classic banking scenario: transferring $100 from Account A to Account B. In SQL, a transaction might look like this:

```sql
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';
  UPDATE accounts SET balance = balance + 100 WHERE account_id = 'B';
COMMIT;
```

If anything goes wrong between the `BEGIN TRANSACTION` and the `COMMIT`, the database rolls back all changes so neither account is altered. Only if every step succeeds does the database "commit" and make the updates visible.

#### 2. Real-life Analogy

Imagine you're writing an important letter by hand, and you want to make sure it contains no errors before you send it out. Your transaction is:

- drafting the letter
- proofreading it word by word, and
- finally sealing it in an envelope.

If you notice a typo or spill ink halfway through, you tear up the letter and start again from scratch, never sending a half-correct or messy version. Only when every step is double-checked and perfect do you actually mail the letter. The whole process is all-or-nothing, just like a transaction.

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

Write skew happens when two concurrent operations each read a shared predicate/condition, both see it as true, and then each make updates in a way that jointly invalidates the condition. Because each actor touches a different row, row-level locking or "last-writer-wins" semantics don't catch an error like this.

For example, imagine that you always need to have 1 person scheduled for on-call at a hospital. Doctor A reads the roster, sees Doctor B on call, and sets their own flag to off-call. Concurrently, Doctor B reads the roster, sees Doctor A on call, and also sets their own flag to off-call. Based on what each of them saw, each operation was valid, but together this means that there are no doctors on-call.

This can happen when there's no "global" check on the validity of concurrent writes, so we need a way to be able to have a way to "check" if the predicate/condition is still valid if we attempt to do a particular action.

#### Problem 4: Double effects (missing idempotence)

Let's say that you're at the checkout page on Amazon. Sometimes when you check out, you get a pop-up saying "do not go back and do not submit the form again". But what if you do? What I'd like to happen is that I don't get double-charged even if I submit a form twice. Let's say I have slow WiFi and the submission is taking a long time and I can't tell if it went through. If I refresh the page and try again, the page would ideally invalidate my attempted purchase since I just bought it.

If I refresh the page and try to buy again and Amazon recognizes that I did that and so it doesn't double-charge my card, that means that purchasing an item on Amazon has **idempotency**. An operation is idempotent if running it multiple times (e.g., pressing the "buy now" button multiple times, refreshing the page and pressing "buy now") produces the same observable result as pressing it once.

It would be a problem if Amazon purchases weren't idempotent and I could get charged 2x, 10x, or even 100x for the same item in my checkout cart, but this is a real problem in systems where we can't guarantee that queued operations will get the most updated state of the database. For a database to have idempotency guarantees, we need to make sure that entries are handled in order and that once an entry with the unique stable identifier is committed, that any other entries are invalidated.

#### Why are naive fixes (retries, “best effort” cleanup, ad-hoc locks) insufficient at scale?

##### "We'll just retry"

Retries mask transient faults but amply any side effects when operations lack idempotence or coordination. Imagine that Amazon can't finalize your purchase on their end so they keep charging your card.

##### "We'll add a few ad-hoc locks"

Ad-hoc locks (cron-held mutexes, database flags, etc.) are very clunky, hard to maintain, are prone to bugs (e.g., locking a specific row but not an index), and don't scale well to multi-node deployments.  These eventually become bottlenecks and temporary patches.

##### "We'll write a few markers and clean it up later"

A lot of databases suppose eventual consistency, meaning that the read and write states across nodes might not be 100% synced at all times. But it's one thing to have this guaranteed by a well-maintained database system as opposed to an ad-hoc patch or cleanup. At best, consumers read slightly stale data, but the data will always be out-of-date until whoever is supposed to "clean up" the data can get to it.

#### Which invariants do we actually care about preserving, and how do crashes break them?

An invariant is the promise your system makes the way a restaurant promises health code standards. You can swap chefs, handle rush hour, or suffer a power blip, but the “don’t poison customers” rule stands. Because enforcing every possible promise is impossible, we intentionally select the ones where a breach is catastrophic.

The invariants that we care about supporting differ based on the use case, but some common ones include:

- Move invariance: money moved from account A must exist in account B. Every debit must pair with a credit.
- Uniqueness: each seat number on a plane can only be assigned to 1 person. Each item in an auction can only be bought by 1 person.
- Monotonic ordering (events never travel backward): If I update my paper in Google Docs and then my roommate updates the same document 5 minutes later, my update should show up before their update. If I write a comment on Facebook and someone comments on my comment, their comment should appear after mine (otherwise, it'll look like they're commenting on a ghost thread).

We can enforce these rules before finalizing any proposed updates to our data and then reverse any updates that don't match these rules.

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
