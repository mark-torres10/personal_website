---
layout: single
title:  "A modern day 'Two Generals' and 'Byzantine Generals' analogy in distributed computing"
date:   2025-11-08 04:00:00 +0800
classes: wide
toc: true
categories:
- self_education
- all_posts
permalink: /self_education/2025-11-10-modern-day-two-generals-analogy
---

# A modern-day analogy of the Two Generals and Byzantine Generals dilemmas in distributed computing

I was having a hard time understanding the core dilemma from the [Two Generals](https://en.wikipedia.org/wiki/Two_Generals'_Problem) in distributed computing (and by extension the generalized [Byzantine Generals](https://en.wikipedia.org/wiki/Byzantine_fault) problem). It wasn't until I tried to connect them to modern-day scenarios that the problem made more sense.

## The original problem

The Two Generals Problem is a foundational demonstration in distributed systems that shows the impossibility of reaching guaranteed agreement over unreliable communication channels, such as messages that can get lost in transit.​

### Two Generals Scenario Breakdown

- Two generals (processes or nodes) want to coordinate a simultaneous attack to succeed; if only one attacks, they both lose.​
- Their only means of communication is sending messengers through enemy territory, but these messengers are sometimes lost or captured and never arrive.​

### Attempt at Coordination

- General A sends a message: “Attack at dawn.”
- General B receives this and replies: “Got it - attack confirmed.”
- General A wonders: “Did my messenger get through to B? Does B know that I got the reply?”
  - They only will know if their messenger got to B with the message "Attack at dawn" if they received the confirmation from B's messenger saying "Got it". If they don't get a messenger back from B saying "Got it", they can't know if B either (a) didn't get the message, and so won't attack, or (b) did get the message, but their acknowledgement ("Got it - attack confirmed") didn't get back to A.
- General B wonders: “Will General A attack, or did my reply not reach them?”
  - General B first needs to get the message from A. If they don't, they won't know to attack.
  - Even if General B gets the "Attack at dawn" message, they need to send a messenger back to A saying "Got it - attack confirmed". But after this messenger is sent, B can't know if A knows that B got it, unless A sends a messenger back to B saying "I got your acknowledgement". But then, B sends to send back another messenger to A saying "I got your acknowledgement".

This was particularly a problem in the battlefield, where messengers could be intercepted and killed and therefore ruin information flow back and forth. There wasn't a way to guarantee that the two generals were 100% on the same page, as confirmations and information exchange required intermediate messengers that had a nonzero `p` of drop-off. This creates a problem *ad infinitum* where neither is 100% sure that the other one got their messages.

## A modern-day version: the office

Imagine two coworkers, Alice and Bob, need to coordinate for an important 9am meeting. Both must attend, but neither wants to show up alone,but Slack is sometimes unreliable or goes down unexpectedly.

### Scenario Walkthrough

#### 1. Alice Sends the Initial Message:
- Alice sends Bob: "I'll bring the slides to the 9am meeting."
- She isn't sure Slack will deliver it (maybe Bob never sees it if Slack fails or bugs out).

#### 2. Bob Receives the Message (maybe):
- If Bob gets Alice's message, he now knows Alice intends to bring the slides.
- But from Alice's perspective: Did the message get through? She's unsure.

#### 3. Bob Sends an Acknowledgment:
- Bob replies over Slack: "Got your message, see you at 9am!"
- Now Bob knows he received Alice's plan, and he replied. But…

#### 4. Did Alice Get Bob's Reply?
- If Slack fails before the acknowledgment reaches Alice, Alice never sees Bob's confirmation.
- Alice doesn't know whether Bob actually got her message, so she isn't sure he'll show up.

#### 5. Desire for Further Confirmation:
- To be extra sure, Alice could send, "Did you get my 'got it'?"—and Bob could reply again.
- But now, it's back to Step 4: what if that message or acknowledgment is lost? The chain never ends; either side could always be waiting for another confirmation.

### Key Points of the Analogy

- **Unreliable Channel**: Slack could drop messages at any point—messages might not be delivered, or acknowledgments might not arrive.
- **Mutual Uncertainty**: Both Alice and Bob want to be sure the other will show up. But if any message is lost, each wonders: "Did they receive my last response? Should I really go to the meeting?"
- **No Guaranteed Common Knowledge**: No matter how many back-and-forth confirmations are sent, each new message could fail, so perfect certainty is impossible without a perfectly reliable communication channel.

### What Actually Gets Lost?

- The message (“I’ll bring the slides”)
- The reply (“Got it”)
- The acknowledgment of the reply
- And any further acknowledgments

### FAQ

#### Why isn't this a problem in real life?

This doesn't end up being a problem in real life because:

- We trust Slack to be highly available (tech companies promise availability in the high 9s and they often have a website that says if the site is experiencing an outage).
- Tech companies have such resilient and battle-hardened architecture that it's a big deal whenever they do have an outage, it's (1) a big deal, but (2) we trust that mission-critical data is not lost - in the Slack case, we may lose a few messages, though presumably there's a local transaction log that can be reconciled with Slack when the outage is resolved; in cases like finance, we trust that even in outages, the companies err on the side of not allowing people to lose their money.
- We have other means of communication, both remote. (e.g,. email, text) and in-person (going up to someone's desk), and it's highly unlikely that all those modes of communication go down simultaneously.

#### Why can't you just send more messages?

Recall that this is only a problem when the medium is unreliable (e.g., if Slack drops messages 30% of the time). You can only increase your confidence, but never achieve 100% certainty, because you cannot guarantee that the very last needed acknowledgment wasn't lost.

This is not just about one message. It is about guaranteeing that both parties know, and both know that the other knows, and so on, in a loop that never ends unless the communication is 100% reliable.

## Why this matters

Distributed computing protocols (like in databases or networks) face this same challenge with unreliable channels.​

In practice, we accept a little risk and use retries/timeouts—just like at work, we trust Slack will usually get the job done, but sometimes, we check in person—or things go wrong.

## Generalization to the Byzantine Generals problem

The [Byzantine Generals](https://en.wikipedia.org/wiki/Byzantine_fault) problem is a generalization of the Two Generals problem in distributed computing. It requires consensus in n-body environments in which the system's actors must agree on a strategy but there is a non-zero chance of (a) information loss (like in the Two Generals case) and, in addition, (b) malicious intent.

Now the problem becomes one of (1) unreliable message delivery (is everyone on the same page) AND (2) possible malicious intent (even if everyone's supposed to be on the same page, someone might be giving bad information to others, either with malicious intent or not).

### The Two Generals Slack Analogy, Revisited

Two coworkers need to agree: "I'll bring slides to the 9am meeting."

- Problem: Slack outages - maybe their messages/acknowledgments are lost, so neither can be 100% sure the other is committed.

### Enter the Byzantine Scenario

Now imagine three or more coworkers need to agree on a plan (say, to organize a big client presentation). Communication is still vulnerable to Slack outages and, crucially, one or more team members might actively mislead others (intentionally or due to buggy software, miscommunication, or even sabotage).

#### What does this look like at work?

- Coworker A posts: "Let's all meet at 9am with slides and handouts."
- Coworker B replies: "Got it! 9am, slides and handouts, I'll bring coffee."
- Coworker C, however, is unreliable (maybe disgruntled, maybe just confused, maybe malicious!). They message Coworker A: "Sure, see you at 10am!" but tell Coworker B: "A said the meeting is canceled."

Now, A and B get conflicting info from C, and can't tell if someone is lying, buggy, or if a message was lost along the network.

#### What's the Core Tension, Now?

- Unreliable messaging: As in the Two Generals, you can't always trust your Slack delivered everything.
- Unreliable people: Now, even if Slack is perfect, someone might intentionally send bad info.

Honest coworkers can't always tell who's being honest or who is sabotaging communications.

Coordination is even harder! Now the problem isn't just about eventually getting the message, it's whether the message can be trusted at all.

#### How Can This Fail?

- Lies and Conflicts: One participant says, "We're meeting at 9am," but sends everyone else a different time—or gives a different version of the plan to everyone.
- Confusion: Honest people receive conflicting stories. Who do you believe? Who's right? How do you reach consensus?
- Not enough reliable people: If there are too many traitors, honest parties simply cannot be sure they're agreeing on the right plan. It's mathematically impossible to guarantee agreement if a third or more are malicious, even if messages always get through

## Workarounds for the Two Generals and Byzantine Generals Problems in Real Distributed Computing Systems

There are practical workarounds for each of these problems. Though there's never a 100% perfect solution, there are implementations that work well enough in practice to be widely adopted in distributed computing.

### Workarounds for the 'Two Generals' Problem

#### Solution 1: Retry and Acknowledge (TCP, Email, Slack "Seen" Flags)

- **What it does**: Systems send messages repeatedly until they get an acknowledgment. You can set a time-out or a maximum retry count.
- **Why it's not perfect**: If your last acknowledgment is lost, one side is still left in doubt. No amount of retries can reach absolute certainty—the last message could always disappear.
- **Office analogy**: On Slack, you keep sending "Did you get it?" or use read receipts, but if Slack crashes after your last message, you're left guessing.

#### Solution 2: Two-Phase Commit (2PC)

- **What it does**: In databases or transactions, a coordinator asks all parties if they're ready to commit (prepare), then sends a final "commit" or "abort" after everyone agrees.
- **Why it's not perfect**: If the network fails during the commit, some participants might commit while others abort, causing split decisions.​
- **Office analogy**: Before a big meeting, you ping everyone for a confirmed "yes"; then send a final "Go". If Slack goes down after sending "Go," some folks might not get the memo and show up late.

#### Solution 3: Go for High Probability, Not Certainty (Eventual Consistency, Practical Protocols)

- **What it does**: Accept that not all actions will be perfectly synchronized; design for "eventual" agreement. Handle errors by retry, reversing actions, or reconciling later (like refunding someone if payment/later confirmation fails).​
- **Why it's practical**: You give up on strict coordination, but mostly life goes on. You might occasionally fix mistakes after-the-fact.
- **Office analogy**: If you don't get a reply from a coworker, you might assume they're coming, and update them later if needed, accepting occasional confusion.

### Workarounds for the 'Byzantine Generals' Problem

#### Solution 1: Majority Agreement (Quorum/Consensus)

- **What it does**: Require decisions to get a supermajority (e.g., at least two-thirds) so that traitors can't mislead honest parties. This is used in algorithms like [PBFT (Practical Byzantine Fault Tolerance)](http://pmg.csail.mit.edu/papers/osdi99.pdf).
- **Office analogy**: For an office poll, everyone votes; at least 2/3 need to agree for the plan to go ahead. Even if one person lies about the plan, the majority wins.

#### Solution 2: Raft/Paxos Consensus Algorithms

- **What it does**: These protocols are designed to reach agreement among nodes even with faulty members. They use multiple rounds of voting and strong leader election so that, even if some messages/nodes fail, honest nodes still reach a decision.​
- **Why they help**: You can't eliminate all failure, but you can withstand up to a set number of traitors or faults.
- **Office analogy**: If your team needs agreement, you select a trusted person to coordinate decision. Others "vote" and if enough honest coworkers agree, you move forward.

#### Raft Consensus Algorithm

##### How does Raft work? (Leader-based coordination)

- Elects a leader who organizes all requests and proposals. The leader receives meeting requests and sends them out to everyone.
- Whenever a proposal is made (such as "let’s meet at 9am"), the leader logs it and distributes the proposal to the group (followers).
- Followers reply back—when a majority have accepted the proposal, the leader considers it committed, informs everyone, and the plan goes into action.
- If the leader fails (crashes, loses connection), a new leader election takes place, and the process starts anew.

##### Raft in the Office

- Your Slack group picks a temporary organizer (say, Alice is the meeting leader).
- Anyone wanting to schedule a meeting tells Alice.
- Alice confirms the time with everyone—she needs thumbs-up from a majority before making it official.
- If Alice becomes unresponsive (e.g., off sick, Slack crash), the group votes to pick a new leader, who resumes leadership from where Alice left off.
- This ensures that, as long as a majority of people respond, meetings are scheduled reliably—even if some coworkers or messages are lost.

#### Paxos Consensus Algorithm

##### How does Paxos work? (Any-to-any coordination)

- Proposers suggest a value ("Let’s hold the meeting at 9am"), and send it to a set of acceptors (the rest of your coworkers).
- Acceptors promise to only accept one value for a given round, to prevent split-brain/conflicting agreements.
- If a proposal gains enough promises (a majority), the value is chosen—everyone agrees to act.
- It can handle failures: if proposers or acceptors crash or messages are delayed, the protocol can keep retrying until consensus is eventually reached (so long as a majority is available).

##### Paxos in the Office

- Imagine you have a group Slack, but some people don’t always get messages, or some are slow to respond.
- You assign one person as proposer, who proposes: “Let’s all show up at 9am.”
- Each person who receives that proposal responds, “I promise to go with this time if most others do.”
- The proposer waits until a majority reply with promises before announcing the official meeting time.
- Even if some acknowledgments or responses get dropped, as long as the proposer gets a majority, the meeting time is set and agreed to by the group.

#### Solution 3: Blockchain/Distributed Ledger (Proof-of-Work, etc.)

- **What it does**: Use cryptographic techniques and global logs to ensure that even if some members cheat, the honest majority can verify what happened.​
- **Why it helps**: Tampering and lying are detectable; reconciliation is public.
- **Office analogy**: Every decision is documented in a shared Google Doc, where anyone can check the history—preventing sneaky changes.
