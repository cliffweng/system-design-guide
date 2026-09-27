# System Design Guide

A practical system design study guide for software engineers — interview prep, and building real-world intuition.

**Live site:** https://cliffweng.github.io/system-design-guide/

## Roadmap

14 topics, one file each under [`topics/`](topics/), building blocks first, case studies last:

1. Scalability & availability
2. CAP theorem & consistency models
3. Load balancing
4. Caching
5. SQL vs NoSQL
6. Partitioning & sharding
7. Queues & async processing
8. Rate limiting
9. CDN & edge
10. Observability
11. Case study: URL shortener
12. Case study: news feed
13. Case study: chat / messaging
14. Case study: distributed rate limiter

## Interview hotspots

Every topic page carries a badge (🎯 Interview frequent / Interview occasional / Background) so you know where to spend prep time. If you're short on time, prioritize these:

- **Scalability & availability** — the near-universal opening question; interviewers use it to see if you can reason about vertical/horizontal tradeoffs and redundancy before diving into specifics.
- **CAP theorem & consistency models** — every database and case-study choice in this guide eventually gets challenged with "what happens during a network partition," so this is load-bearing for everything downstream.
- **Load balancing & caching** — the two pieces of infrastructure that show up in almost every design, and the ones interviewers probe hardest on mechanism (algorithms, invalidation, stampedes) rather than just naming the box.
- **SQL vs NoSQL & partitioning/sharding** — "how is the data stored and split up" is asked in nearly every case study; a shallow answer here undermines an otherwise good design.
- **Queues & async processing** — the standard follow-up to "this operation is slow" or "this needs to scale independently," and a common source of delivery-guarantee gotcha questions.
- **Rate limiting** — asked both as a standalone topic and baked into the [distributed rate limiter case study](topics/14-case-distributed-rate-limiter/); interviewers specifically probe the distributed/shared-state version.
- **The four case studies (11–14)** — these are where everything else in the guide gets combined under interview time pressure; practicing the requirements → design → bottlenecks flow matters more than memorizing any one architecture.

**Occasional** (still worth knowing, less likely to anchor a whole interview): CDN & edge, observability — usually surface as a follow-up rather than the main question.

This split is a judgment call based on what shows up in practice today, not a guarantee for any specific interview loop — adjust your prep if a role is unusually infrastructure- or SRE-focused.

## How to use this guide

Each topic page is designed to be read in **~10 minutes** and follows the same structure: why it matters, core concepts, a mental model (diagram), interview questions with brief answer keys, and a short list of verified YouTube videos. Read them in order, or jump straight to what you need. No backend, no auth, no sign-up — just read the pages.

## How to contribute

See [CONTRIBUTING.md](CONTRIBUTING.md). In short: one topic per file, keep it under ~10 minutes to read, and only link to sources (especially YouTube videos) you've personally verified exist. Open an issue before proposing new topics or restructuring the curriculum.

## Decisions

These are the product locks this guide was built against — echoed here so future contributors don't accidentally relitigate them:

- **Audience**: software engineers, learning + interview prep. Not a general "intro to computing" course.
- **Time-boxed**: every topic is readable in 10 minutes or less. Depth is sacrificed for scannability; "further reading" links are where depth lives.
- **Learning + interview prep in one page**: each topic pairs core concepts with interview questions, rather than splitting them into separate tracks.
- **Real links only**: every YouTube link is verified to exist (via the YouTube oEmbed endpoint) before being added. No invented URLs, ever.
- **Static site, GitHub Pages, Just the Docs**: no backend, no auth, no Vercel. Cheap to host, cheap to maintain, easy to contribute to via plain Markdown + front matter.

## Enabling GitHub Pages

Just the Docs is configured via `_config.yml` (remote theme). After this lands on `main`:

1. Repo **Settings → Pages**.
2. **Build and deployment → Source**: **Deploy from a branch**.
3. Branch: `main` / folder: `/ (root)`.
4. Save. Site publishes to https://cliffweng.github.io/system-design-guide/

## Local preview

```bash
bundle install
bundle exec jekyll serve
```

Then open `http://localhost:4000`.

## License

[MIT](LICENSE)
