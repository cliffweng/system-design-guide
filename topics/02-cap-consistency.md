---
title: "02. CAP theorem & consistency models"
layout: default
nav_order: 3
---

# CAP theorem & consistency models
{: .no_toc }

*~8 min read*

**🎯 Interview frequent**

## Why it matters

Every distributed data store has to make a choice when the network breaks, and interviewers use CAP as a quick way to check whether you understand *why* that choice exists — not just that "CAP theorem says pick two." Real systems (and real interview follow-ups) live in the consistency-model details underneath CAP: what does your database actually promise a reader will see, and what does that cost you in latency or availability? This topic underpins database choice ([SQL vs NoSQL](../05-sql-vs-nosql/)) and shows up again in [partitioning & sharding](../06-partitioning-sharding/) and every case study.

## Core concepts

- **CAP theorem.** In a distributed system, when a network **P**artition happens, you must choose between **C**onsistency (every read sees the latest write, or an error) and **A**vailability (every request gets a non-error response, possibly with stale data). You can't have all three — partitions are a fact of networked life, so the real choice is C vs. A *during a partition*. Outside of a partition, most systems try to offer both.
- **CP systems.** Prioritize consistency over availability during a partition — nodes that can't confirm they have the latest data refuse to serve (or block) rather than risk returning stale/wrong data. Examples: ZooKeeper, etcd, most traditional RDBMS in synchronous-replication mode. Good fit for coordination/config data and anything where a wrong answer is worse than no answer (leader election, financial ledgers).
- **AP systems.** Prioritize availability — every node keeps responding even if it can't confirm it's in sync with the rest of the cluster, meaning reads can return stale data during a partition. Examples: Cassandra, DynamoDB (in its default mode), Riak. Good fit for data where staleness is tolerable and downtime isn't (shopping carts, social media likes, session data).
- **PACELC — CAP's more honest sequel.** CAP only describes behavior *during* a partition. PACELC adds: even when there's **no** partition, you still trade **L**atency for **C**onsistency — synchronously confirming a write across replicas is slower than acknowledging locally and replicating async. So the real design question is two-part: "P → A or C?" and "Else → L or C?"
- **Consistency models, from strongest to weakest.** *Strong/linearizable*: every read sees the most recent committed write, as if there were only one copy of the data. *Sequential*: all nodes see operations in the same order, just not necessarily "real time." *Causal*: operations that are causally related (a reply to a comment) are seen in order; unrelated ones may not be. *Eventual*: given no new writes, all replicas *will* converge, with no bound on when. Stronger consistency generally costs more latency/availability.
- **Quorum-based tuning (N/W/R).** Many AP-leaning systems (Cassandra, DynamoDB) let you tune per-operation: N = replicas, W = replicas that must ack a write, R = replicas read from. If **W + R > N**, reads are guaranteed to overlap with the latest write (strong consistency); if not, you get eventual consistency but lower latency. This is how one system can serve both use cases depending on the call.
- **Eventual consistency is a real-world default, not a compromise you always notice.** DNS propagation, CDN cache invalidation, and "your comment count updates a few seconds late" are all eventual consistency in production, and users tolerate it fine when the staleness window is short and the data isn't safety-critical.

## Mental model

```
            Consistency
               /\
              /  \
             /    \
            / pick \
           /  two   \
          /----------\
Availability --------- Partition tolerance
                        (not optional — networks partition)

  during a partition: choose C (refuse/block) or A (serve stale)
```

PACELC as a decision path:

```
Is there a partition right now?
  ├── yes → choose Availability or Consistency  (CAP)
  └── no  → choose Latency or Consistency        (ELC)
```

## Interview questions

1. **Explain CAP theorem in your own words — why can't a system have all three?**
   Answer: When a network partition splits nodes into groups that can't talk to each other, each group must decide whether to keep serving requests (possibly with data that might be stale relative to the other group) or refuse/block until it can confirm consistency. You can't do both at once for the same request, and since partitions are unavoidable in real networks, C-vs-A is the choice that actually matters; "P" isn't something you opt out of.

2. **Give a real example of a CP system and an AP system, and explain why each made that choice.**
   Answer: etcd/ZooKeeper are CP — they're used for leader election and config, where two nodes disagreeing about who's the leader is far worse than a brief unavailability. Cassandra/DynamoDB (default settings) are AP — they back things like shopping carts and user activity feeds, where staying available under partition matters more than every replica agreeing instantly, and conflicts can be resolved later (e.g., last-write-wins or application-level merge).

3. **What is eventual consistency, and can you give a real-world example most people don't think of as "eventual consistency"?**
   Answer: Eventual consistency guarantees that if no new writes occur, all replicas will *eventually* converge to the same value, with no guaranteed bound on when. DNS is a classic example: after you update a DNS record, different resolvers around the world serve the old value for a while (per their TTL) until it propagates — nobody calls this "eventual consistency" colloquially, but it's exactly the model.

4. **What does PACELC add on top of CAP, and why does it matter even when there's no partition?**
   Answer: CAP is silent about system behavior in the common case where there's no partition — but even then, you trade Latency for Consistency: a synchronous multi-replica write acknowledgment is slower than an async one. PACELC makes explicit that consistency has a latency cost baseline, so "we're not in a partition" doesn't mean the tradeoff disappears.

5. **What do N, W, and R mean in a quorum-based system, and what does W + R > N guarantee?**
   Answer: N is the number of replicas a piece of data is stored on; W is how many replicas must acknowledge a write before it's considered successful; R is how many replicas a read queries before returning a result. If W + R > N, any read set and any write set must overlap on at least one replica, guaranteeing the read sees the latest write — i.e., you get strong consistency tuned per-operation, at the cost of higher latency (more replicas to contact) than a lower W or R would need.

6. **You're designing a shopping cart service and a bank account balance service. Would you make the same C/A choice for both? Why or why not?**
   Answer: No. The shopping cart favors availability — if it briefly shows a stale cart during a partition, that's a minor UX issue, easily reconciled (e.g., merge carts, worst case a customer re-adds an item), so an AP store fits. The account balance favors consistency — showing a stale or wrong balance (or letting two concurrent withdrawals both succeed against the same funds) is a correctness/financial risk, so a CP store (or strong consistency mode) fits even at the cost of occasional unavailability.

## Watch

- [CAP Theorem Simplified](https://www.youtube.com/watch?v=BHqjEjzAicA) — ByteByteGo. Clear, fast explanation of the theorem with concrete CP/AP examples.
- [What is CAP Theorem?](https://www.youtube.com/watch?v=eWMgsk7mpFc) — IBM Technology. Covers the theorem plus how it plays out in real distributed database choices.

## Further reading

- [Eventually Consistent](https://www.allthingsdistributed.com/2007/12/eventually_consistent.html) — Werner Vogels (Amazon CTO), the original widely-cited explanation of eventual consistency.
- [Problems with CAP, and Yahoo's Little Known NoSQL System](https://dbmsmusings.blogspot.com/2010/04/problems-with-cap-and-yahoos-little.html) — Daniel Abadi's post introducing PACELC.
