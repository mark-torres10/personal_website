---
layout: single
title:  "Two-phased commits (2PC), explained"
date:   2025-11-10 06:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /self_education/2025-11-10-two-phased-commit
---

# Two-phased commits (2PC), explained

## What is a two-phase commit (2PC) protocol?

*The below explanation is grounded in the [Wikipedia article](https://en.wikipedia.org/wiki/Two-phase_commit_protocol) about the two-phase commit protocol, which is quite comprehensive!*

## What problem does it solve?

Suppose you need to transfer $100 from a checking account in Bank A to a savings account in Bank B. Both banks must reflect the change atomically, so if money leaves Bank A, it must appear at Bank B.

The obvious thing to do would be:

- Step 1: Deduct $100 from Bank A.
- Step 2: Add $100 to Bank B.

But what happens if this goes wrong? Imagine Bank A successfully deducts, but a network failure occurs before Bank B can add the money. The result is that $100 "vanishes" from the system and the accounts are inconsistent.

2PC aims to fix this. It is designed so that each of the steps in the transaction must all succeed, otherwise it doesn't go through. Importantly, 2PC aims for **atomic commitment** (either everyone commits or everyone aborts). This isn't the only way to go about it; [protocols like Paxos/Raft aim for majority quorom](https://markptorres.com/self_education/paxos-raft-explained), for example, but there are tradeoffs to each approach.

## How does the 2PC protocol work?

### 2PC protocol components

1. **Coordinator**: Orchestrates the protocol, requests votes, and makes the final commit or abort decision.

2. **Participants (Cohorts)**: Each holds a part of the distributed transaction, votes yes/no based on local success.

Some assumptions that are made at this point are:

- Stable storage (write-ahead logs) at every node.
- Nodes do not crash permanently.
- Write-ahead log survives crashes.
- Nodes can communicate with each other.

### Algorithm, step-by-step

#### Phase 1: Voting (Prepare/Commit-Request Phase)

The coordinator sends a “prepare/commit-request” message to all participants.

Each participant executes the transaction locally up to the commit point, logs the necessary undo/redo info, and responds:

- Yes (Agreement): Ready to commit.
- No (Abort): Unable to commit (local failure).

#### Phase 2: Commit (Completion Phase)

If all vote Yes:

- Coordinator sends a “commit” message to all.
- Participants commit locally, release locks/resources, and acknowledge.
- Coordinator completes the transaction on receiving all acknowledgments.

If any votes No or a timeout occurs:

- Coordinator sends a “rollback” message.
- Participants undo locally, release resources, and acknowledge.
- Coordinator finalizes on receiving all acknowledgments.

#### A simple worked-out scenario

Suppose we have three databases (DB1, DB2, DB3):

- DB1 (coordinator) starts a transaction affecting DB2 and DB3.
- Phase 1: DB1 sends “prepare” to DB2/DB3. DB2 replies “Yes”, DB3 replies “Yes”.
- Phase 2: DB1 sends “commit”. All databases commit, acknowledge completion.

If DB3 had replied “No”, DB1 would send “rollback” and all databases would undo their changes.

## When does 2PC do well?

2PC does well in the following scenarios:

- Distributed databases requiring strict ACID transactions.
- Good when failures are rare and temporary.
- Provides strong and easy atomicity guarantees between systems (since we know that all nodes have to agree and be successful before a transaction is completed).

## Drawbacks of 2PC and failure modes

- **High latency** due to message round-trips, logging, and coordination.
- **Blocking**: If the coordinator fails after participants have voted Yes, those participants can be blocked, waiting indefinitely for commit/rollback instructions.
- **Not resilient to all failures**: If both coordinator and a participant crash, the outcome may stall indefinitely (since 2PC requires that (a) a coordinator communicates to all nodes and (b) that all participant nodes agree).
- **Slow recovery**: recovering frmo failure requires log-based recovery and then reconciling past log states with the current state to find ground truth.

## Where is 2PC used in industry?

- Classical SQL relational databases support 2PC for distributed transactions.
- Banking systems and federated DBs needing global atomic updates.
- Payment process, when requiring strong atomicity.

However, the industry by and large is shifting away from 2PC. There's been a push towards high availability, non-blocking operations, and partition tolerance, none of which 2PC can provide (because it needs constant performance from a single coordinator and all the participant nodes, else it can't ever commit a transaction). For these reasons, modern cloud databases (e.g., Spanner) and NoSQL systems (e.g., MongoDB, Cassandra), as well as modern financial firms and tech companies, often use consensus-based approaches, which offer strong performance (perhaps not having the atomicity guarantee as 2PC) while meeting availability and resilience requirements.

## Applying 2PC on our banking example

Let's take the previous banking case:

```markdown
Suppose you need to transfer $100 from a checking account in Bank A to a savings account in Bank B. Both banks must reflect the change atomically, so if money leaves Bank A, it must appear at Bank B.
```

How would the two-phase commit handle this?

### 1. Prepare Phase (Voting/Commit-Request)

A coordinator asks Bank A and Bank B: "Are you ready to commit?"

- Each bank performs their local portion, writes to logs, and responds:
  - "Yes" (if all is ready)
  - "No" (if any local error occurs)

### 2. Commit Phase (Completion)

- If all say Yes:
  - Coordinator instructs both: "Commit!"
  - Both finalize the transaction and respond "Done."
- If any say No:
  - Coordinator instructs both: "Rollback."
  - Both undo any local changes and respond "Done."

If a failure occurs between prepare and commit, the protocol ensures recovery and consistent outcome: either both banks process the transaction or neither does.
