---
title: "05. SQL vs NoSQL"
layout: default
nav_order: 6
---

# SQL vs NoSQL
{: .no_toc }

*~8 min read*

**🎯 Interview frequent**

## Why it matters

"Would you use SQL or NoSQL here?" is one of the most common system design questions precisely because there's no universally right answer — it's a proxy for whether you can reason about access patterns, consistency needs, and scale before reaching for a technology. A weak answer names a database; a strong answer explains what about the workload led there.

## Core concepts

- **The relational model and ACID.** SQL databases (Postgres, MySQL) store data in tables with fixed schemas and relationships enforced via foreign keys, queried with joins. ACID transactions guarantee **A**tomicity (all-or-nothing), **C**onsistency (constraints always hold), **I**solation (concurrent transactions don't see each other's half-done work), and **D**urability (committed data survives a crash) — this is what makes relational databases the default choice whenever correctness of multi-row operations matters (e.g., transferring money between two accounts).
- **NoSQL is a family, not one thing.** *Key-value* (Redis, DynamoDB): simplest model, extremely fast lookups by key, no query flexibility beyond that. *Document* (MongoDB): stores semi-structured JSON-like documents, good when your data is naturally nested and doesn't need to be normalized. *Wide-column* (Cassandra, Bigtable): optimized for very high write throughput and queries over huge datasets partitioned by a row key, common for time-series and logging. *Graph* (Neo4j): optimized for traversing relationships (friend-of-friend, recommendation graphs) that would require expensive joins in a relational model.
- **BASE as NoSQL's answer to ACID.** Many NoSQL systems trade strict consistency for availability and scale, described as **B**asically **A**vailable, **S**oft state, **E**ventually consistent (see [CAP theorem & consistency models](../02-cap-consistency/)). This isn't a weaker version of correctness — it's a different contract that's a better fit when the workload can tolerate brief staleness in exchange for lower latency and horizontal scalability.
- **Why NoSQL scales horizontally more easily.** Removing cross-row transactions and joins means a NoSQL store can shard data across many machines with minimal coordination between them — each shard can process reads/writes independently (see [partitioning & sharding](../06-partitioning-sharding/)). Relational databases *can* be sharded too, but joins and multi-row transactions across shards are expensive or impossible, which is the actual reason "SQL doesn't scale" became conventional wisdom (it's a workload-pattern limitation, not a law of physics).
- **Schema flexibility vs. query power.** NoSQL documents can have different fields per record and evolve without a migration, which is convenient for rapidly changing or heterogeneous data — but you give up the query optimizer's ability to do arbitrary joins/aggregations efficiently, pushing more logic into the application. SQL's fixed schema is more upfront cost but buys you flexible, declarative querying over relationships forever.
- **Denormalization for scale.** At high read scale, relational systems often deliberately denormalize (duplicate data across tables to avoid joins) or NoSQL documents embed related data directly (a blog post document embedding its comments) — trading storage and write complexity (must update every copy) for read speed. This is a recurring theme across [caching](../04-caching/) and the [case studies](../11-case-url-shortener/) later in this guide.

## Mental model

| | SQL (relational) | NoSQL |
|---|---|---|
| Schema | Fixed, enforced | Flexible / schema-less |
| Transactions | Multi-row ACID | Usually single-key atomic only |
| Scaling | Vertical easiest; horizontal is hard (joins) | Horizontal by design |
| Query power | Rich (joins, aggregations) | Limited to access patterns you designed for |
| Best fit | Correctness-critical, relational data | High scale, simple access patterns, flexible schema |

## Interview questions

1. **What does ACID actually guarantee, and why does it matter for choosing a database?**
   Answer: Atomicity (a transaction fully happens or not at all), Consistency (data always satisfies defined constraints), Isolation (concurrent transactions don't interfere), Durability (once committed, survives a crash). It matters because any workload involving multi-step operations that must all succeed or all fail together (debit one account, credit another) needs these guarantees — without them you risk partial updates that leave data in an invalid state.

2. **When would you pick a document database over a relational one?**
   Answer: When your data is naturally hierarchical/nested and usually read/written as a whole unit (a user profile with embedded preferences, a product with embedded reviews), when the schema will evolve frequently and you don't want migrations gating every change, or when you need to scale writes horizontally more easily than a relational system's joins allow.

3. **Why do NoSQL databases generally scale horizontally more easily than relational ones?**
   Answer: By giving up cross-row transactions and joins, each piece of data can live independently on its own shard with no need to coordinate with other shards for a given operation. Relational databases *can* be sharded, but a join or transaction spanning shards requires expensive cross-node coordination, so most relational deployments scale vertically or shard only when the workload can tolerate the loss of cross-shard joins.

4. **What's the tradeoff of denormalizing data for scale?**
   Answer: Reads get faster and simpler (no joins, or a single document read instead of multiple table lookups), but writes get more expensive and error-prone — the same piece of data may need to be updated in multiple places, and any missed update location introduces inconsistency. It's worth it when reads vastly outnumber writes, which is the common case for content-serving systems.

5. **You're designing a system where a single logical operation must update three related rows atomically, or none at all. What database family fits, and why?**
   Answer: A relational database, because that's exactly what multi-row ACID transactions are for — the database guarantees all three updates commit together or roll back together. Most NoSQL stores only guarantee atomicity at the single-key/single-document level, so achieving the same guarantee there would require building compensating logic (sagas, two-phase commit) in the application, which is more complexity than just using a database that already provides it.

6. **What's a wide-column store good for, and what's a concrete example use case?**
   Answer: Wide-column stores (Cassandra, Bigtable) are optimized for very high write throughput and for queries that scan a large, well-partitioned range of rows by a known key — a canonical example is time-series/event data, like storing sensor readings or user activity logs partitioned by device ID or user ID and ordered by timestamp, where you write constantly and query by "give me this key's data in this time range."

## Watch

- [SQL vs. NoSQL: What's the difference?](https://www.youtube.com/watch?v=Q5aTUc7c4jg) — IBM Technology. Clear breakdown of the core differences and when each is appropriate.
- [SQL vs NoSQL - Tradeoffs](https://www.youtube.com/watch?v=QzLhb1WBFjQ) — Gaurav Sen. System-design-interview-focused framing of the decision.

## Further reading

- [PostgreSQL Documentation — Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html) — official explanation of ACID transactions in a relational database.
- [MongoDB — When to Use MongoDB](https://www.mongodb.com/resources/products/fundamentals/when-to-use-mongodb) — vendor's own framing of document-model fit, useful for understanding the intended use cases.
