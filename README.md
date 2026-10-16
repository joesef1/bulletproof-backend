<div align="center">

# Bulletproof Backend

### From _"I kinda get backend"_ to _"I can reason about any backend problem."_

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](https://makeapullrequest.com)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-orange.svg?style=flat-square)](#-contributing)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)
[![Status](https://img.shields.io/badge/status-actively%20built-brightgreen.svg?style=flat-square)](#-repo-status)

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

## Learning Backend in the AI Era

AI changed how we learn. Any syntax, any boilerplate, any standard snippet is one prompt away. Memorizing those things used to be valuable. It is not anymore.

What AI cannot do for you: know what to ask, judge if the answer is right, and choose between options. When AI writes your database query, you still must know if it is slow. When it designs your login flow, you still must know if it is safe. A wrong answer that looks right is worse than no answer at all.

So this repo focuses on the timeless parts: how requests, data, and failures really work. These do not change when a new tool comes out. Learn them, and AI becomes a strong helper — you guide it, check it, and make the calls. Not the opposite.

---

## Who Is This For?

**This repo is for you if any of these sound familiar:**

- 😰 _"I write backend code, but I don't feel like I actually understand backend."_
- 🎭 _"I have junior-level skills and senior-level anxiety."_
- 🎤 _"I'm interviewing for backend roles and panicking on fundamentals."_
- 🧩 _"I know the **what** (Django, Express, .Net,...) but not the **why** behind it."_
- 🌫️ _"I can't explain the big picture — where does a request even start?"_
- 🧠 _"I learn best by seeing the whole system, not scattered fragments."_
- 🛠️ _"I want to level up from 'I follow tutorials' to 'I make decisions.'"_

---

## What You'll Walk Away With

|                                |                                                                                                                                       |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| 🗺️ **A complete mental model** | From `google.com` typed in a browser → TCP → TLS → HTTP → your framework → your database → back out to the client. Every hop visible. |
| 🎯 **Interview-ready basics**  | The questions interviewers actually ask, answered properly. Not hand-waved.                                                           |
| 🧠 **Senior-level judgment**   | Knowing _why_ one design beats another — ACID vs. eventual consistency, SQL vs. NoSQL, sync vs. async.                                |
| 🐛 **The ability to debug**    | When something breaks at 2am, you'll know _where in the stack_ to look, and why.                                                      |
| 🛡️ **Production instincts**    | The failure modes nobody teaches you until they bite you — N+1 queries, connection exhaustion, cache invalidation.                    |
| 🚫 **No more guessing**        | Not because you know everything — but because you know what actually matters, and where the edges are.                                |

---

## 📚 Content Catalog — What This Repo Covers

`content.md` is the raw topic list. It groups topics by **category**, not by learning order. Categories help you find a topic. They do **not** tell you where to start.

The ten sections:

- **00 — Backend Foundations and Networking** — architecture, request lifecycle, DNS, TCP/UDP, HTTP/HTTPS, TLS, concurrency, I/O vs. CPU, CORS
- **01 — API Engineering and Application Architecture** — request handling, layers, REST, serialization, OpenAPI, versioning, errors, validation
- **02 — Database Engineering and Data Integrity** — modeling, constraints, transactions, isolation, indexing, ORMs, migrations, pooling, diagnosis
- **03 — Authentication, Security, and Access Control** — AuthN/AuthZ, sessions/tokens, OAuth/OIDC, crypto, common web bugs, rate limiting, firewalls, threat modeling
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

```bash
git clone https://github.com/joesef1/bulletproof-backend.git
cd bulletproof-backend
```

Then read [`main.md`](main.md). Follow the stages in order. Each stage lists its topics, why that order, and a checkpoint to pass before moving on.

> 💡 **Pro tip:** After reading a topic, close the file and explain it out loud. If you can't, you didn't learn it — you just recognized it. Then apply it in a project you work on.

---

## 🤝 Contributing

Everyone is welcome here. Report a mistake, improve an explanation, or add a missing topic.

**Ways to help:**

- 🐛 **Report issues** — broken links, unclear explanations, wrong info
- 📝 **Improve a topic** — better examples, clearer analogies, more depth
- ➕ **Add a missing topic** — especially in sections 04–09
- 💡 **Suggest ordering fixes** — "this topic needs an earlier prerequisite"

**Contribution guidelines:**

- Keep it **practical**
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

The repo follows `content.md` (catalog) + `main.md` (learning order).

| Section                         | Topics | Status                            |
| ------------------------------- | ------ | --------------------------------- |
| 00 — Foundations and Networking | 9      | 🟡 Roadmap mapped, tutorials open |
| 01 — API Engineering            | 9      | 🟡 Roadmap mapped, tutorials open |
| 02 — Database Engineering       | 10     | 🟡 Roadmap mapped, tutorials open |
| 03 — Auth and Security          | 8      | 🟡 Roadmap mapped, tutorials open |
| 04 — Production Readiness       | 9      | 🟡 Roadmap mapped, tutorials open |
| 05 — Testing and QA             | 7      | 🟡 Roadmap mapped, tutorials open |
| 06 — Deployment and DevOps      | 8      | 🟡 Roadmap mapped, tutorials open |
| 07 — Performance and Async      | 9      | 🟡 Roadmap mapped, tutorials open |
| 08 — Distributed Systems        | 7      | 🟡 Roadmap mapped, tutorials open |
| 09 — System Design in Practice  | open   | ⚪ Sub-topics to be defined       |

_Actively built. Tutorials land in `main.md` stage order — Stage 1 first._

---

## 📄 License

Released under the [MIT License](LICENSE). Free to use, share, and teach from.

---

<div align="center">

⭐ Star it · 🍴 Fork it · 🤝 Improve it

🛡️ _"You don't have to learn all of backend. You just have to learn what holds it up."_

</div>
