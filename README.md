<div align="center">

# Bulletproof Backend

### From _"I kinda get backend"_ to _"I can reason about any backend problem."_

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](https://makeapullrequest.com)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-orange.svg?style=flat-square)](#-contributing)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/status-roadmap%20rebuild-brightgreen.svg?style=flat-square)](#-repo-status)

</div>

---

## The Promise

Most backend engineers write code that _works_. Very few write code that **holds up**.

That gap — between shipping and _shipping that survives production_ — is where imposter syndrome lives. It is not that you do not know enough. It is that you cannot yet tell **which** things matter.

**This repo closes that gap.** Not with 400 links. With the concepts that carry almost all the weight — the ones that let you reason about problems you have not seen before.

> 🛡️ **Learn what matters. Build things that don't break.**

---

## The Problem

Backend engineering is **enormous.** Networking, APIs, databases, security, testing, deployment, scaling, distributed systems — it spreads in every direction, and most learning material makes it _worse_, not better.

So most people collect tutorials. They open twelve tabs. They half-read each one. They never build the mental model. And months later they still cannot answer a simple interview question without fear.

**That is the trap.** Not that the topic is too big — but that nobody tells you _which parts matter, and in which order._

---

## The Idea

There is a small set of backend concepts that explain **almost everything** you will ever build:

| What goes wrong                            | The concept that explains it                      |
| ------------------------------------------ | ------------------------------------------------- |
| 💥 Your API is inexplicably slow           | HTTP lifecycle + indexing                         |
| 🧨 A query that's fine in dev dies in prod | Connection pooling + transactions                 |
| 🔓 Your login is insecure                  | Hashing vs. encryption vs. encoding               |
| 📉 Your server falls over at 1,000 users   | Stateless architecture + caching + load balancing |
| 😬 Data silently corrupted                 | ACID + transactions + normalization               |

Master those deeply and the rest of backend stops feeling like a wall and starts feeling like trivia.

---

## Who Is This For?

**This repo is for you if any of these sound familiar:**

- 😰 _"I write backend code, but I don't feel like I actually understand backend."_
- 🎭 _"I have junior-level skills and senior-level anxiety."_
- 🎤 _"I'm interviewing for backend roles and panicking on fundamentals."_
- 🧩 _"I know the **what** (Django, Express, Postgres) but not the **why** behind it."_
- 🌫️ _"I can't explain the big picture — where does a request even start?"_
- 🧠 _"I learn best by seeing the whole system, not scattered fragments."_
- 🛠️ _"I want to level up from 'I follow tutorials' to 'I make decisions.'"_

You do **not** need 5 years of experience. You need a map.

---

## What You'll Walk Away With

|                                     |                                                                                                                                       |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| 🗺️ **A complete mental model**      | From `google.com` typed in a browser → TCP → TLS → HTTP → your framework → your database → back out to the client. Every hop visible. |
| 🎯 **Interview-ready fundamentals** | The questions interviewers actually ask, answered properly. Not hand-waved.                                                           |
| 🧠 **Senior-level judgment**        | Knowing _why_ one design beats another — ACID vs. eventual consistency, SQL vs. NoSQL, sync vs. async.                                |
| 🐛 **The ability to debug**         | When something breaks at 2am, you'll know _where in the stack_ to look, and why.                                                      |
| 🛡️ **Production instincts**         | The failure modes nobody teaches you until they bite you — N+1 queries, connection exhaustion, cache invalidation.                    |
| 🚫 **Zero impostor syndrome**       | Not because you know everything — but because you know what actually matters, and where the edges are.                                |

---

## 📚 Content Catalog — What This Repo Covers

`content.md` is the raw topic list. It groups topics by **category**, not by learning order. Categories help you find a topic. They do **not** tell you where to start.

The ten sections:

- **00 — Backend Foundations and Networking** — architecture, request lifecycle, DNS, TCP/UDP, HTTP/HTTPS, TLS, concurrency, I/O vs. CPU, CORS
- **01 — API Engineering and Application Architecture** — request handling, layers, REST, serialization, OpenAPI, versioning, errors, validation
- **02 — Database Engineering and Data Integrity** — modeling, constraints, transactions, isolation, indexing, ORMs, migrations, pooling, diagnosis
- **03 — Authentication, Security, and Access Control** — AuthN/AuthZ, sessions/tokens, OAuth/OIDC, crypto, OWASP-style bugs, rate limiting, firewalls, threat modeling
- **04 — Production Readiness and Reliability** — config/secrets, logging, alerting, errors, timeouts/retries, shutdown, health checks, incidents, twelve-factor
- **05 — Testing and Quality Assurance** — strategies, doubles, test data, coverage/TDD, API/contract tests, database tests, auth tests
- **06 — Deployment, Infrastructure, and DevOps** — servers/proxies, Docker, CI/CD, hosting, backups, prod migrations, release strategies
- **07 — Performance, Caching, and Asynchronous Processing** — latency/throughput, profiling, query tuning, caching, queues, reliable async, storage, pagination, scaling
- **08 — Advanced Backend and Distributed Systems** — distribution basics, consistency/availability, CAP, fault tolerance, load balancing, capacity, search
- **09 — System Design in Practice** — capstone: design full systems from the pieces above (sub-topics open)

> ⚠️ **Do not study top to bottom.** Some topics depend on others. A database transaction means nothing before you know what a database is. OAuth means nothing before you know HTTP and auth basics.

---

## 🗺️ Where to Start — The Learning Order

The catalog above is **not** the curriculum. The curriculum lives in `main.md`.

`main.md` puts the same topics in **dependency order** — eight stages, each one building on the last:

| Stage | Name | In short |
| ----- | ---- | -------- |
| 1 | How the Web Works | Networking, HTTP, TLS, servers, CORS |
| 2 | Build an API | Layers, REST, validation, errors, OpenAPI |
| 3 | Store Data Correctly | Modeling, transactions, indexes, ORMs, pooling |
| 4 | Secure It | Auth, crypto, OAuth, attacks, rate limiting |
| 5 | Test What You Built | Unit, integration, API, database, auth tests |
| 6 | Ship It to Production | Config, Docker, CI/CD, releases, logging, incidents |
| 7 | Scale It | Profiling, caching, queues, pagination, scaling |
| 8 | Distribute It and Design Systems | Load balancing, CAP, fault tolerance, system design |

👉 **Start here: [`main.md`](main.md).** Follow the stages in order. Each stage lists its topics, why that order, and a checkpoint to pass before moving on.

**Shortcuts for common cases:**

| Your situation                                | Path through `main.md`                          |
| --------------------------------------------- | ----------------------------------------------- |
| 🔴 **Starting out**                           | Stage 1 in order. Don't skip the handshakes.    |
| 🟡 **Framework-literate**, fundamentals fuzzy | Stages 1 → 2 → 3, then pass each checkpoint     |
| 🟢 **Strong fundamentals**, want depth        | Skim 1–2, prove it at checkpoints, focus 3 → 8  |
| 🎤 **Interview in 2 weeks**                   | Stage 1 (HTTP/TLS) → Stage 3 (ACID/indexing) → Stage 4 (auth/crypto) |
| 😵 **Lost, need the big picture**             | Stage 1 topics 1–2 first, then follow the map  |

> 💡 **Pro tip:** After reading a topic, close the file and explain it out loud to an empty room. If you can't, you didn't learn it — you just recognized it.

> 🛡️ **Bulletproof habit:** For every topic, ask _"what breaks if I get this wrong?"_ That question is the whole difference between working code and production code.

---

## 🤝 Contributing

This repo is a **work in progress**, and it gets better the more smart people add to it.

**Ways to help:**

- 🐛 **Report issues** — broken links, unclear explanations, wrong info
- 📝 **Improve a topic** — better examples, clearer analogies, more depth
- ➕ **Add a missing topic** — especially in sections 04–09
- 🌐 **Add translations** — this material should be accessible to everyone
- 💡 **Suggest ordering fixes** — "this topic needs an earlier prerequisite"

**Contribution guidelines:**

- Keep it **practical** — real examples, real failure modes
- Keep it **explainable** — if a junior can't follow it, rewrite it
- **No fluff.** Every line should teach something.
- Check `content.md` for the catalog and `main.md` for where the topic belongs in learning order.

---

## 📚 Topic Template

Consistency makes a repo learnable. When you write a topic, follow this structure:

```markdown
# Topic Name

## What It Is

One-sentence definition. Plain language.

## Why It Matters

The problem it solves. When you'll actually need it.

## How It Works

The mechanism. Diagrams encouraged.

## Failure Modes

What breaks when it's implemented wrong. (This is the bulletproof part.)

## Example

Real code. Real request/response.

## Key Points

- Bullet summary for fast review
```

**Tone:** Confident, clear, jargon-defined-on-first-use. Talk _to_ the reader — they're smart, just uncertain.

---

## 📊 Repo Status

The old 5-pillar structure is deleted. The repo now follows `content.md` (catalog) + `main.md` (learning order).

| Section | Topics | Status |
| ------- | ------ | ------ |
| 00 — Foundations and Networking | 9 | 🟡 Roadmap mapped, tutorials open |
| 01 — API Engineering | 9 | 🟡 Roadmap mapped, tutorials open |
| 02 — Database Engineering | 10 | 🟡 Roadmap mapped, tutorials open |
| 03 — Auth and Security | 8 | 🟡 Roadmap mapped, tutorials open |
| 04 — Production Readiness | 9 | 🟡 Roadmap mapped, tutorials open |
| 05 — Testing and QA | 7 | 🟡 Roadmap mapped, tutorials open |
| 06 — Deployment and DevOps | 8 | 🟡 Roadmap mapped, tutorials open |
| 07 — Performance and Async | 9 | 🟡 Roadmap mapped, tutorials open |
| 08 — Distributed Systems | 7 | 🟡 Roadmap mapped, tutorials open |
| 09 — System Design in Practice | open | ⚪ Sub-topics to be defined |

_Actively built. Tutorials land in `main.md` stage order — Stage 1 first._

---

## ⭐ Star This Repo

If this is the backend roadmap you wish you'd had when you started — **star it** so you can find it again, and share it with someone who's stuck.

That feeling of _"I don't really understand backend"_? It's not a talent problem. **It's an information problem.** And information problems are solvable.

---

## 📄 License

Released under the [MIT License](LICENSE). Free to use, share, and teach from.

---

<div align="center">

**Built in public, for people learning in public.**

⭐ Star it · 🍴 Fork it · 🤝 Improve it

🛡️ _"You don't have to learn all of backend. You just have to learn what holds it up."_

</div>
