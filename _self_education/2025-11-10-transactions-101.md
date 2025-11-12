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

When you have multiple systems that are trying to coordinate, why can't you just wait until everyone confirms that they're on the same page? Turns out, this is a well-studied dilemma.

### The Two Generals Problem

One problem that arises is that agreement is impossible with unreliable communication if you demand certainty. This is shown through the Two Generals problem (check [this article](https://markptorres.com/self_education/2025-11-10-modern-day-two-generals-analogy) for a deeper dive about the Two Generals problem). In practice, all distributed systems live with this limitation. They can achieve eventual agreement, probabilistic agreement, or durable commit decisions, but never guaranteed synchronized action in the face of message loss or crashes, and so systems are built knowing that 100% certainty is impossible (but you can get as close as possible).

Here's a toy Python demo of the Two Generals problem:

```python
import random

def unreliable_send(msg, loss_prob=0.3):
    """Simulate unreliable message passing."""
    return random.random() > loss_prob  # message delivered?

def two_generals(loss_prob=0.3, max_messages=5):
    messages = 0
    ack_from_b = False
    while messages < max_messages:
        messages += 1
        sent = unreliable_send("attack?", loss_prob)
        if not sent:
            print(f"Message {messages} lost.")
            continue
        print(f"Message {messages} delivered.")
        ack = unreliable_send("ack", loss_prob)
        if ack:
            ack_from_b = True
            print("B sent ack back successfully.")
        else:
            print("Ack lost.")
        if ack_from_b:
            break
    return ack_from_b

result = two_generals()
print("\nDecision:", "Attack together" if result else "Abort - uncertain")
```

Here's what I got from running it:

```python
Message 1 delivered.
Ack lost.
Message 2 lost.
Message 3 delivered.
Ack lost.
Message 4 delivered.
Ack lost.
Decision: Abort - uncertain
```

If the system requires acknowledgement between different parties, then the Two Generals problem shows us that this is impossible to have with 100% certainty when you have lossy networks. This shows why you need a global source of truth, like a commit log, instead of relying on acknowledgements.

### The FLP Impossibility theorem

The FLP (Fischer-Lynch-Paterson) theorem proves than in an asynchronous distributed system where at least 1 node has a nonzero probability of crashing, you can't both guarantee liveness and safety of outcomes.

What does this actually mean? Let's walk through a non-technical example. Let's say that you're in a group chat with 5 friends and you're making plans for a fancy restaurant reservation (the kind where you can't walk in and you have to reserve the exact number of seats) on Saturday. You need everyone to say “yes” or “no” to lock plans. You message the group; four friends reply quickly, but Sam goes silent. Do you go ahead without Sam? If you assume “no reply means yes,” you risk Sam showing up outraged that plans changed. If you wait forever, you never book the restaurant. FLP says that in an asynchronous world with even one flaky participant, a protocol can’t guarantee both:

- Safety (no conflicting decisions) and
- Liveness (eventually making a decision).

Your group chat mirrors the theorem’s “bivalent” state: until Sam responds or is declared officially out, both “go” and “don’t go” remain possible. In practice you set a timeout or appoint a leader to decide, but those are extra assumptions that break pure asynchrony, which is exactly what real consensus algorithms (e.g., Paxos, Raft) do. They impose extra "tiebreaker" rules so that in cases like this where at least one node can hold up the rest of the group, that the group has a default course of action in case they never hear back from that node.

### How do transactions manage this impossibility?

Transactions don't solve the problems raised by the Two Generals and FLP dilemmas. Instead, they make assumptions and practical tradeoffs. Below, we'll discuss using two types of protocols: (1) two-phased commits (2PC) and (2) Paxos/Raft.

#### Two-phased commits

We describe in detail [in this link](https://markptorres.com/self_education/2025-11-10-two-phased-commit) what two-phased commits (2PC) are. We'll focus here on the tradeoffs that the 2PC protocol makes.

##### Allowing blocking

In the classic 2PC model, the coordinator must write to an external log and all nodes must reference this log as the source of truth. If the coordinator dies, all other nodes are blocked and delayed until the coordinator can come back and complete the write to the log. This is FLP in action, where the 2PC protocol refuses to guess and would rather block a set of actions than make the wrong action. The coordinator is the single point of progress; things move as fast or as slow as the coordinator can update the commit log, and this is by design.

##### Sacrificing liveness

Allowing blocking in the 2PC protocol also means that in practice, systems sacrifice the liveness requirement. Transactions can take time to be finalized or committed if there is a delay in the coordinator writing the ground truth to the log.

##### Using durable logs and deterministic replay

The use of logs is yet another sign of the protocol erring on the side of safety over liveness. By having durable logs, every vote (prepare/commit/abort) is persisted to disk, and after a crash the log can be replayed, meaning that we always have a ground-truth state.

#### Paxos/Raft

We describe in more detail [in this link](https://markptorres.com/self_education/2025-11-10-modern-day-two-generals-analogy) about what the Paxos and Raft protocols are.

## ACID semantics (and useful subsets)

TODO: have as a separate blogpost.

## Programming model vs implementation

## A Python demo showing the tradeoffs

(generic notes that I'll divvy up into separate articles later on)
Notes:

- Transactions:
  - Why do we need a new paradigm? What problem do transactions solve?
  - What are transactions?
