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

### Why would we want an alternative to Paxos?

If Paxos is so great, why do we need another consensus algorithm?

Paxos is great in theory, but in practice it proves to be hard to implement, prone to bugs, and difficult to maintain. For production cases that require correct implementation and long-term maintainability, engineers prefer to use Raft instead.

### How does the Raft algorithm work?

The Raft algorithm has the following components:

- Leader Election
- Log Replication
- Safety Rules
- Cluster Membership Changes & Log Compaction

#### Leader Election

Raft times partitions into **terms**. Each term can have a leader.

Election happens when followers do not receive a heartbeat (a periodic "I'm here" message) from a laeder within a random election timeout period (e.g., 150-300 ms), they become candidates and start an election(a randomized timeout reduces the likelihood of multiple nodes timing out concurrently and trying to become leaders).

During a proposed election, a candidate increments its term, votes for itself, then requests votes from other nodes. The first candidate to receive votes from a majority of nodes becomes its new leader. If a candidate sees a term higher than its own, it steps down to follower status.

#### Log replication

The leader node processes the client requests, adding each new request as a new entry in its own log.

The leader then tries to replicate the log entry to all the followers via `AppendEntries` [RPCs](https://www.knowledgenile.com/blogs/understanding-remote-procedure-call-rpc-in-distributed-computing).

The leader waits for confirmation from a majority of servers before considering the entry committed. Once committed, the entries are applied to the state machine (e.g,. if the request was to add a row to a server, then once the leader has committed the entry to the log, it applies the change to the database) and the reply goes back to the client. Followers commit the entry once informed that it's committed.

If the leader fails, the new leader fixes inconsistencies by comparing logs with followers and bringing them up to date, deleting or overwriting conflicting entries.

#### A worked-out example of Raft

Let's walk through a short scenario:

1. All 5 servers are followers. Server 1 times out and starts an election (term 1), becoming a candidate.

2. Server 1 asks for votes. Servers 2, 3, 4 vote for it; Server 5 times out or votes for itself.

3. Server 1 wins majority (3 out of 5), becomes leader, begins sending heartbeats to all.

4. A client sends a command to leader (Server 1). Leader appends it to its log as entry 1.

5. Replication: Leader sends AppendEntries (including entry 1) to 2, 3, 4, and 5. 2, 3, and 4 reply with success, but 5 is down.

6. Once leader gets responses from a majority (including itself, plus 2 more), entry 1 is considered committed. Leader applies the entry.

7. Leader informs followers entry 1 is committed; they also apply it. Cluster converges on state.

8. Server 5 returns: On heartbeat, leader sends all missing log entries; Server 5's log is corrected.

9. If the leader fails, another server times out, wins a vote, takes over, and log consistency is ensured before committing new entries.

#### Where does Raft do well?

Raft does well in the same safety, liveness, durability, and fault tolerance considerations that Paxos does well in. In addition, it has the following benefits:

- **Understandability**: because Raft works in distinct steps (leader election, commit replication) and the operations occur within defined time intervals, it's easier to implement and debug compared to Paxos.

- **Strong consistency**: committed entries are durable even through crashes.

#### Drawbacks of Raft

- **Leader bottleneck**: although Raft doesn't have the liveness stalls that are possible with Paxos, it has the opposite problem: all client communication and log replication goes through the leader, which can limit write throughput and create a single point of failure for performance.

- **No Byzantine Fault Tolerance out of the box**: Like Paxos, Raft is great for multi-node consensus, but doesn't have countermeasures for adversarial environments.

### Systems that use Raft

Raft is widely used in industry as the consensus algorithm underpinning many distributed systems that require strong consistency, high availability, and durability. It has been adopted both directly and via libraries in several high-profile products and cloud infrastructures.

Raft is used within [etcd](https://github.com/etcd-io/raft), a widely used, highly available key-value store that forms the backbone of Kubernetes cluster state management and coordination.

Docker uses Raft as part of [Swarmkit](https://github.com/moby/swarmkit), which powers Docker Swarm, to maintain cluster state and orchestrate containers reliably.

Distributed message queues like [RabbitMQ](https://www.rabbitmq.com/docs/quorum-queues) and [NATS Jetstream](https://docs.nats.io/running-a-nats-service/configuration/clustering/jetstream_clustering) use their own implementations of Raft for managing distributed queues and determining quorom.

Many modern databases like [ScyllaDB](https://www.scylladb.com/glossary/paxos-consensus-algorithm/), [CockroachDB](https://www.cockroachlabs.com/docs/stable/architecture/replication-layer), [Neo4j](https://neo4j.com/docs/operations-manual/current/clustering/setup/routing/) also use Raft.

Apache introduced  [Kafka KRaft](https://developer.confluent.io/learn/kraft/), their own implementation of Raft, to start decoupling Kafka from ZooKeper. This allows  Kafka to manage cluster metadata internally and removes the dependency on ZooKeeper that historically introduced complexity and scaling bottlenecks in practice.

Basically, Raft is used as the go-to consensus algorithm for distributed applications throughout industry.

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
