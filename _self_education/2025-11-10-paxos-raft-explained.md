---
layout: single
title:  "Paxos and Raft consensus algorithms, explained"
date:   2025-11-10 07:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /self_education/paxos-raft-explained
---

# Paxos and Raft consensus algorithms, explained

This is a brief primer on consensus algorithms, specifically Paxos and Raft, and how they fit into the landscape of transaction-based systems.

## What are consensus algorithms?

The fundamental problem with Two-Phase Commit (2PC) is that it guarantees atomicity but not availability. In 2PC, a single coordinator orchestrates all participants to either commit or abort a transaction. If that coordinator fails after participants have voted “yes” but before sending the final “commit,” every participant is left waiting indefinitely, unable to safely decide whether to commit or roll back. This blocking behavior means that the system cannot make progress until the coordinator recovers, even if all other nodes are healthy and reachable. In distributed systems, where partial failures are common, this makes 2PC brittle and unsuitable for high-availability environments.

Consensus algorithms like Paxos and Raft were designed to overcome this limitation by eliminating any single point of failure (like a single coordinator in the 2PC case) and allowing progress as long as a majority quorum of nodes remains alive. Instead of a fixed coordinator, these protocols dynamically elect leaders and use overlapping majorities to ensure that decisions are both consistent and durable, even if some nodes crash or become unreachable. The result is a system that achieves the same atomic agreement as 2PC—but with fault-tolerant progress, ensuring that the cluster can keep operating smoothly through failures.

## What is the Paxos algorithm?

The [Paxos algorithm](https://martinfowler.com/articles/patterns-of-distributed-systems/paxos.html) is one approach for leaderless consensus across multiple nodes.

### How does the Paxos algorithm work?

*The following is based from [the description of Paxos from Wikipedia](https://en.wikipedia.org/wiki/Paxos_(computer_science)), which is pretty comprehensive!*

The Paxos algorithm has 3 components:

- **Proposers**: Suggest values for agreement.
- **Acceptors**: Vote on these proposals based on protocol constraints.
- **Learners**: Learn the outcome (the agreed-upon value).

The Paxos algorithm consists of three steps:

#### Paxos Phase 1: Prepare

The proposer tells the other nodes "I'm going to send you a new proposal. I want you to *prepare* for it". The proposer also sends a proposal number *n* (this helps to keep track of the order that proposals come in).

- **Phase 1a**: The proposer sends the proposal.
- **Phase 1b**: Other nodes respond by either (1) agreeing to not accept proposals < *n*, or (2) sending a "not acknowledged" or not responding at all.

#### Paxos Phase 2: Accept

Once it gets majority promises, proposer sends `Accept(n, v)`. Acceptors accept if they haven’t promised a higher proposal number.

- **Phase 2a**: The proposer has to figure out what value to include in the proposal. If any Acceptor, in its Promise, reported a previously-accepted value, use the value from the highest-numbered such proposal. Else if none, the proposer sends its own value. The Proposer then sends an `Accept(n, v)` message to a quorum.
- **Phase 2b**: Each Acceptor, upon `Accept(n, v)`, checks if it has already promised to only consider proposals with numbers greater than `n`.
  - If not, it accepts the proposal, records `v` and notifies Proposers and Learners with an `Accepted(n, v)` message.
  - Otherwise, it ignores the message.

When a majority (quorum) of Acceptors accept the same proposal number, consensus is achieved on the value for that identifier.

#### Paxos Phase 3: Commit/Learn

Once a majority accepts, the value is chosen. Let all the replicas know that a value has been chosen.

#### How does Paxos handle rounds and failures?

If multiple Proposers compete (e.g., network partitions or leader failure), rounds may conflict and fail. Resolution is by retrying with higher proposal numbers.

Paxos guarantees that once a value is chosen (by a quorum), it cannot be replaced, regardless of new proposals.

#### Where does Paxos do well?

- **Safety**: Only a value that was actually proposed can be chosen (not some phantom or corrupt value). No two different values are chosen for the same run (consensus instance), so all honest nodes agree on the outcome. This means if a value is chosen, it was (1) proposed at some point, and (2) was the only value chosen in that round of voting.

- **Liveness**: The algorithm doesn't get stuck in an infinite indecision loop, trying to decide a value. Eventually, it will pick a value. The system doesn't get stalled in a deadlock, which would be bad for persisting values across nodes.

- **Durability**: because of the safety and liveness guarantees, Paxos works well for replicated databases, filesystems, or leader election where safety is critical.

- **Fault Tolerance**: since it requires a majority quorom, it can survive up to *F* faults given *2F+1* nodes, which makes it resilient to network partitions or process crashes.

#### Drawbacks of Paxos

- **Liveness in theory, not guaranteed in practice**: although liveness is guaranteed at the limit, it is possible for nodes to get caught in contention (e.g., lots of simultaneous proposers), but there are mitigations for these problems (e.g., electing a single definite proposer).

- **Performance**: Each consensus operation (per-value in Basic Paxos) requires several round trips.

- **No Byzantine Fault Tolerance out of the box**: Paxos is great for multi-node consensus, but doesn't have countermeasures for adversarial environments.

### Paxos in practice: multi-Paxos

**Multi-Paxos** is an extension of basic Paxos designed to efficiently reach consensus on a sequence of values (such as commands or log entries), rather than repeating the entire Paxos protocol for each value.

#### How does multi-Paxos work?

##### 1. **Leader Election**

In Multi-Paxos, the system chooses a leader (sometimes called master or coordinator) through the basic Paxos protocol. The leader is responsible for proposing values for each slot in the distributed log. The first round uses full Paxos: a Prepare phase to claim leadership.

##### 2. **Efficient Repeated Consensus**

Once a leader is established and remains stable, future log entries can skip the Prepare phase:

- Clients send commands to the leader.
- For each command, the leader assigns a log index (slot) and immediately sends an Accept message (phase 2 of Paxos) to a quorum of Acceptors, proposing its value for that slot.
- Acceptors reply with Accepted messages to the leader and learners.

This optimization saves messages and time; phase 1 (Prepare/Promise) is only required when a new leader is elected due to failure or contention.

##### 3. **Handling Leader Failure**

If the leader fails or there is contention, a new round begins:

A new proposer initiates phase 1 (Prepare) to claim leadership for future slots, ensuring safety (no conflicting choices).

##### 4. **Sequence of Consensus Instances**

Each log slot (command number) represents an instance of consensus. Multi-Paxos links many Paxos consensus rounds together efficiently. Each value/log entry is agreed upon through the protocol, but steady state requires only two message delays from client request to value learned (instead of four in basic Paxos).

#### Why Multi-Paxos?

1. Reduces message overhead: fewer round trips per value when leader is stable.
2. Designed for long-lived, repeat consensus: ideal for databases, reliable distributed logs, replicated state machines.

### Systems that use Paxos

Paxos is a fundamental consensus algorithm used throughout industry. For example, [Google's distributed lock service, Chubby](https://static.googleusercontent.com/media/research.google.com/en//archive/chubby-osdi06.pdf) and [distributed database, Spanner](https://static.googleusercontent.com/media/research.google.com/en//archive/spanner-osdi2012.pdf), use implementations of multi-Paxos. Elsewhere, [Cassandra](https://docs.datastax.com/en/cassandra-oss/3.0/cassandra/dml/dmlLtwtTransactions.html), [Apache Zookeeper](https://zookeeper.apache.org/doc/r3.4.13/zookeeperInternals.html), and many others use variations of Paxos.

## What is the Raft algorithm?

If Paxos is so great, why do we need another consensus algorithm?

Conceptually, Raft = Multi-Paxos with a structured leader protocol and log repair mechanism.

### Why would we want an alternative to Paxos?

### Systems that use Raft

Many...

Databases like [ScyllaDB](https://www.scylladb.com/glossary/paxos-consensus-algorithm/)...

## Python implementation

[This is a Github link](link) to a worked-out implementation of Paxos, multi-Paxos, and Raft.


Here are some results from running this script:

```python
(base) ➜  database_from_scratch git:(main) ✗ python consensus_demo.py paxos       
------------------------------------------------------------------------
Alive acceptors: [0, 2, 4]
[PAXOS] target value = 'x=42', proposal number = 1
------------------------------------------------------------------------
Acceptor 0: PROMISE for n=1; previously accepted=(None,None)
Acceptor 4: PROMISE for n=1; previously accepted=(None,None)
Not enough promises: got 2, need 3. Liveness may stall.
------------------------------------------------------------------------
RESULT: chosen=False, value=None, msgs=2
(base) ➜  database_from_scratch git:(main) ✗ python consensus_demo.py multi-paxos
------------------------------------------------------------------------
Alive acceptors: [0, 2, 4]
[PAXOS] target value = 'x=1', proposal number = 1
------------------------------------------------------------------------
Acceptor 0: PROMISE for n=1; previously accepted=(None,None)
Acceptor 4: PROMISE for n=1; previously accepted=(None,None)
Not enough promises: got 2, need 3. Liveness may stall.
Leader could not be established; sequence may stall.
------------------------------------------------------------------------
RESULT: chosen_sequence=[None], msgs=2
(base) ➜  database_from_scratch git:(main) ✗ python consensus_demo.py raft       
------------------------------------------------------------------------
Alive nodes: [0, 2, 4]
[RAFT] Starting election
Node 0: votes for Candidate 2 (term 1)
Node 4: votes for Candidate 2 (term 1)
[RAFT] Leader elected: Node 2 with 3/5 votes (term 1)
[RAFT] Leader 2 proposes log[0]='x=1'
  Follower 0: appended 'x=1' (term 1)
  -> NOT committed 'x=1' (acks=2, need 3)
[RAFT] Leader 2 proposes log[1]='x=2'
  Follower 0: appended 'x=2' (term 1)
  Follower 4: appended 'x=2' (term 1)
  -> Committed 'x=2' with 3/5 acks
[RAFT] Leader 2 proposes log[2]='x=3'
  Follower 0: appended 'x=3' (term 1)
  -> NOT committed 'x=3' (acks=2, need 3)
------------------------------------------------------------------------
RESULT: leader=2, chosen_sequence=[None, 'x=2', None], msgs=6
```
