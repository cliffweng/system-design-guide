---
title: "01. Scalability & availability"
layout: default
nav_order: 2
---

# Scalability & availability
{: .no_toc }

*~8 min read*

**🎯 Interview frequent**

## Why it matters

Almost every system design interview opens with some version of "how would you scale this?" — it's the question that separates candidates who can name technologies from candidates who understand tradeoffs. Scalability (handling more load) and availability (staying up when things fail) are related but distinct goals, and conflating them is a common mistake: a system can be trivially scalable and still go down constantly, or rock-solid available while falling over the moment traffic doubles. Getting the vocabulary and the tradeoffs right here sets up every other topic in this guide.

## Core concepts

- **Vertical vs. horizontal scaling.** Vertical scaling (scale up) means a bigger machine — more CPU/RAM on the same box. It's simple but hits a hard ceiling and creates a single point of failure. Horizontal scaling (scale out) means more machines behind a [load balancer](../03-load-balancing/) — it has no theoretical ceiling but requires the application to be stateless or to externalize state (sessions, cache) so any instance can serve any request.
- **Availability vs. reliability vs. durability.** Availability is "is the system responding right now" (measured in uptime %, the "nines"). Reliability is "does it keep working correctly over time without failing." Durability is specifically about data — "once written, will it survive" (disk failure, region loss). A system can be highly available while serving stale or wrong data, so these are independent axes.
- **The nines and error budgets.** 99.9% ("three nines") allows ~8.7 hours of downtime/year; 99.99% allows ~52 minutes; 99.999% allows ~5 minutes. Each additional nine is exponentially more expensive to engineer for, which is why teams define an explicit availability target (SLO) and treat the remaining allowed downtime as an "error budget" to spend on risk (deploys, experiments) rather than trying to hit 100%.
- **Redundancy and failover.** Eliminating single points of failure means running N+1 (or more) of every critical component. Active-active failover runs multiple instances all serving traffic simultaneously (better resource use, but requires handling split-brain/consistency); active-passive keeps a standby that takes over on failure (simpler, but the standby is idle capacity and failover takes non-zero time).
- **Statelessness is what makes horizontal scaling easy.** If any server can handle any request because no server holds request-specific state in memory, you can add/remove servers freely behind a load balancer. Session data, in-progress uploads, and WebSocket connections are the usual culprits that sneak state into a "stateless" service — see [caching](../04-caching/) for where that state actually goes.
- **Throughput vs. latency.** Scaling for throughput (requests/sec) and scaling for latency (response time per request) are different problems that sometimes trade off against each other — e.g., batching increases throughput but adds latency per item. Be explicit in an interview about which one you're optimizing for.
- **Scaling can *reduce* availability if done carelessly.** Adding more servers without fixing a shared bottleneck (one DB, one cache) just moves the failure point. Cascading failures — one slow dependency exhausts thread pools/connections in callers, which then become slow themselves — are a common way "we added capacity" makes an outage worse, not better.

## Mental model

```
Vertical scaling:              Horizontal scaling:

   [ bigger box ]                [ LB ]
        ^                       /  |  \
        |                 [ srv ][ srv ][ srv ]
   (hard ceiling,                 |
    single point                (stateless — any
    of failure)                  server, any request)
```

Availability budget at a glance:

| Uptime target | Downtime / year | Downtime / month |
|---|---|---|
| 99% | ~3.65 days | ~7.3 hours |
| 99.9% | ~8.7 hours | ~43 min |
| 99.99% | ~52 min | ~4.3 min |
| 99.999% | ~5 min | ~26 sec |

## Interview questions

1. **When would you scale vertically instead of horizontally?**
   Answer: When the workload isn't easily parallelizable (a single large in-memory computation, a legacy monolith with shared in-process state), when you need a quick fix and horizontal scaling requires re-architecture, or at low scale where the operational overhead of a fleet isn't worth it yet. It's a legitimate short-term lever, not just "the wrong answer."

2. **What does "five nines" mean and why don't most systems target it?**
   Answer: 99.999% uptime, roughly 5 minutes of downtime per year. Most systems don't target it because the cost/complexity to go from four nines to five nines (redundant everything, sub-second failover, extremely disciplined change management) is far higher than the business value of those extra minutes — teams instead set an SLO appropriate to what users actually need and spend the error budget on shipping velocity.
   
3. **What's the difference between scalability and availability, concretely?**
   Answer: Scalability is about handling increased load (more users, more data, more requests/sec) without degrading. Availability is about staying reachable and functioning despite failures (hardware death, network partition, bad deploy). A single powerful server can be scalable-enough for its load but has zero redundancy, so it's not highly available; conversely, a small fleet with failover can be highly available while still falling over under 10x load.

4. **How do you eliminate a single point of failure in a typical web architecture?**
   Answer: Run redundant instances of every tier — multiple app servers behind a load balancer, a database with replicas (and automated failover), redundant load balancers themselves (via DNS or a floating IP/VRRP), and multi-AZ or multi-region deployment for the data center itself. The general pattern: identify each component, ask "what happens if this one instance dies," and add redundancy until the answer is "nothing user-visible."

5. **Explain active-active vs. active-passive failover and when you'd choose each.**
   Answer: Active-active runs all replicas serving live traffic, so failover is instant (traffic just stops routing to the dead node) and you get full use of capacity, but it requires the system to tolerate concurrent writes/reads across nodes (consistency handling). Active-passive keeps a standby that isn't serving traffic until promoted, which is simpler to reason about (no concurrent-write conflicts) but wastes standby capacity and has a failover gap (detection + promotion time). Choose active-active when you can afford the consistency complexity and want zero-downtime failover; active-passive for simpler systems or when strong consistency makes active-active impractical.

6. **How can adding more servers make an outage worse instead of better?**
   Answer: If the new servers all point at one shared bottleneck (a single database, a single downstream API) that wasn't scaled proportionally, you've just increased the load hitting that bottleneck faster, potentially triggering a cascading failure — timeouts pile up, retries amplify load further, and thread/connection pools exhaust across the whole fleet. Scaling has to be end-to-end: identify the actual bottleneck before adding capacity upstream of it.

## Watch

- [Scalability Simply Explained in 10 Minutes](https://www.youtube.com/watch?v=EWS_CIxttVw) — ByteByteGo. Concise walkthrough of vertical vs. horizontal scaling and the core techniques for handling more load.
- [System Design BASICS: Horizontal vs. Vertical Scaling](https://www.youtube.com/watch?v=xpDnVSmNFX0) — Gaurav Sen. Focused comparison of the two scaling strategies with concrete tradeoffs.

## Further reading

- [Google SRE Book — Embracing Risk (error budgets)](https://sre.google/sre-book/embracing-risk/) — the canonical explanation of SLOs and error budgets.
- [AWS Well-Architected Framework — Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html) — practical patterns for redundancy and failure recovery.
