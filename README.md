<div align="center">

# 🧠 The 20/80 Backend Course

### The only map you need to go from *"I kinda get backend"* to *"I can reason about any backend problem."*

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](https://makeapullrequest.com)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-orange.svg?style=flat-square)](#-contributing)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)
[![Python](https://img.shields.io/badge/examples-Python-3776AB.svg?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Status](https://img.shields.io/badge/status-actively%20built-brightgreen.svg?style=flat-square)](#-roadmap)

</div>

---

## The Problem

Backend engineering is **enormous.** You could spend ten years and still not "learn all of it." Distributed systems, databases, networking, security, infrastructure, observability — it sprawls in every direction, and most learning material makes it *worse*, not better.

So you do what most people do: you collect tutorials. You open twelve tabs. You half-read each one. You never build the mental model. And six months later you still can't answer a simple interview question without flinching.

**That is the trap.** Not that the topic is too big — but that nobody tells you *which parts matter.*

---

## The Idea (Pareto, applied to your career)

There is a small set of backend concepts that explain **almost everything** you will ever build:

- Why your API is slow → HTTP lifecycle + indexing
- Why your query is fast in dev but dies in prod → connection pooling + transactions
- Why your login is insecure → hashing vs. encryption vs. encoding
- Why your server falls over at 1,000 users → stateless architecture + caching + load balancing

Learn those **20%** deeply and the remaining 80% stops feeling like a wall and starts feeling like trivia.

> **The 20/80 Backend Course** is that 20%. Not a list of 400 links. Five pillars, distilled to the concepts that carry the weight.

---

## Who Is This For?

**This repo is for you if any of these sound familiar:**

- 😰 *"I write backend code, but I don't feel like I actually understand backend."*
- 🎭 *"I have junior-level skills and senior-level anxiety."* (Impostor syndrome has a curriculum — this helps.)
- 🎤 *"I'm interviewing for backend roles and panicking on fundamentals."*
- 🧩 *"I know the **what** (Django, Express, Postgres) but not the **why** behind it."*
- 🌫️ *"I can't explain the big picture — where does a request even start?"*
- 🧠 *"I learn best by seeing the whole system, not scattered fragments."*
- 🛠️ *"I want to level up from 'I follow tutorials' to 'I make decisions.'"*

You do **not** need 5 years of experience. You need a map.

---

## What You'll Walk Away With

| | |
|---|---|
| 🗺️ **A complete mental model** | From `google.com` typed in a browser → TCP → TLS → HTTP → your framework → your database → back out to the client. Every hop visible. |
| 🎯 **Interview-ready fundamentals** | The questions interviewers actually ask, answered properly. Not hand-waved. |
| 🧠 **Senior-level judgment** | Knowing *why* one design beats another — ACID vs. eventual consistency, SQL vs. NoSQL, sync vs. async. |
| 🐛 **The ability to debug** | When something breaks at 2am, you'll know *where in the stack* to look, and why. |
| 🚫 **Zero impostor syndrome** | Not because you know everything — but because you know what the important 20% is, and where the edges are. |

---

## 🏛️ The Five Pillars

### Pillar 1 — The Internet & Communication
*The foundational rules of how data moves across the wire.*

- **HTTP/HTTPS Lifecycle** — DNS Resolution · TCP 3-Way Handshake · TLS/SSL Handshake · Request/Response Cycle
- **RESTful API Design** — HTTP Verbs · Idempotency · Resource-Based URLs · Status Codes
- **Real-Time vs. Async** — Webhooks · Polling (Short & Long) · WebSockets
- **Data Serialization** — JSON Parsing · Data Validation
- **CORS** — Preflight requests, why your frontend is blocked, how to fix it
- **API Versioning** — Evolving an API without breaking clients
- **HTTP/2 & HTTP/3** — Multiplexing, QUIC, and killing head-of-line blocking

`✅ Complete`

---

### Pillar 2 — Databases & Data Management
*How state is stored, retrieved, and not silently corrupted.*

- **Relational Databases (SQL)** — ACID Properties · Normalization · Joins · Transactions
- **NoSQL Databases** — Key-Value (Redis) · Document Stores (MongoDB) · CAP Theorem
- **Indexing** — B-Trees · Primary vs. Foreign Keys · Composite Indexes
- **ORMs** — Abstraction · The N+1 Query Problem
- **Migrations** — Schema vs. data migrations, dependencies
- **Connection Pooling** — Why your app crashes without it

`🟡 In progress` — SQL core shipped · NoSQL, Indexing, ORMs, Migrations & Pooling coming next

---

### Pillar 3 — Security & Identity
*Defending the app and verifying users.*

- **Authentication vs. Authorization** — AuthN vs. AuthZ, sessions vs. JWT
- **Cryptography Basics** — Hashing vs. Encryption vs. Encoding (and why confusing them ruins you)
- **The Big 3 Vulnerabilities** — XSS · CSRF · SQL Injection
- **OAuth 2.0 / OpenID Connect** — "Login with Google," properly
- **Rate Limiting & Throttling** — Abuse, brute force, resource protection

`⚪ Planned`

---

### Pillar 4 — Performance & Scalability
*Handling real traffic without crashing.*

- **Caching Strategies** — Redis/Memcached and the hardest problem in CS: invalidation
- **Async Task Queues** — Message brokers, workers, Celery
- **Scaling Systems** — Vertical vs. horizontal, stateless architecture
- **Load Balancing** — Nginx, AWS ALB
- **Pagination** — Offset vs. cursor, and why one dies at scale

`⚪ Planned`

---

### Pillar 5 — Deployment & Infrastructure
*Getting code from your laptop to the public internet, reliably.*

- **Containerization (Docker)** — Killing "works on my machine"
- **CI/CD** — Automated testing and deployment
- **Environment Variables & Secrets** — Never hardcode. Never. Ever.
- **Web Servers vs. App Servers** — Nginx vs. Uvicorn/Gunicorn, WSGI/ASGI
- **Logging & Monitoring** — Logging config, Sentry, catching errors before users do
- **Testing** — Unit · Integration · Mocking · Fixtures · Coverage

`⚪ Planned`

---

## 🗺️ How to Use This Repo

```bash
git clone https://github.com/joesef1/20-80-backend-course.git
cd 20-80-backend-course
```

Then just read, in order. Each topic is a self-contained markdown file — short, dense, and written to be *understood*, not skimmed.

**Suggested paths:**

| Your situation | Start here |
|---|---|
| 🟢 **Strong fundamentals**, want depth | Pillar 2 → 3 → 4 → 5 |
| 🟡 **Framework-literate, fundamentals fuzzy** | Pillar 1 from the top, then Pillar 2 |
| 🔴 **Starting out** | Pillar 1 in order. Don't skip the handshakes. |
| 🎤 **Interview in 2 weeks** | Pillar 1 REST section → Pillar 2 ACID/Joins/Transactions → Pillar 3 |
| 😵 **Lost, need the big picture** | Read `main.md` first — it's the whole checklist in one page |

> 💡 **Pro tip:** After reading a topic, close the file and explain it out loud to an empty room. If you can't, you didn't learn it — you just recognized it.

---

## 🤝 Contributing

This repo is a **work in progress**, and it gets better the more smart people add to it.

**Ways to help:**
- 🐛 **Report issues** — broken links, unclear explanations, wrong info
- 📝 **Improve a topic** — better examples, clearer analogies, more depth
- ➕ **Add a missing topic** — especially in Pillars 3–5
- 🌐 **Add translations** — this material should be accessible to everyone
- 💡 **Suggest a pillar** — "what's missing from the 20%?"

**Contribution guidelines:**
- Keep it **practical** — real examples, real failure modes
- Keep it **explainable** — if a junior can't follow it, rewrite it
- **No fluff.** Every line should teach something.
- Match the existing markdown style in `Pillar 1/`

---

## 📚 Contributing Guidelines & Style Guide

Consistency makes a repo learnable. When you write a topic, follow this structure:

```markdown
# Topic Name

## What It Is
One-sentence definition. Plain language.

## Why It Matters
The problem it solves. When you'll actually need it.

## How It Works
The mechanism. Diagrams encouraged.

## Example
Real code. Real request/response.

## Key Points
- Bullet summary for fast review
```

**Tone:** Confident, clear, jargon-defined-on-first-use. Talk *to* the reader — they're smart, just uncertain.

---

## 📊 Repo Status

| Pillar | Status | Topics |
|---|---|---|
| 1 — The Internet & Communication | ✅ Complete | 16 |
| 2 — Databases & Data Management | 🟡 In progress | 4 |
| 3 — Security & Identity | ⚪ Planned | — |
| 4 — Performance & Scalability | ⚪ Planned | — |
| 5 — Deployment & Infrastructure | ⚪ Planned | — |

*Actively built. Check back weekly — new topics land regularly.*

---

## ⭐ Star This Repo

If this is the backend roadmap you wish you'd had when you started — **star it** so you can find it again, and share it with someone who's stuck.

That feeling of *"I don't really understand backend"*? It's not a talent problem. **It's an information problem.** And information problems are solvable.

---

## 📄 License

Released under the [MIT License](LICENSE). Free to use, share, and teach from.

---

<div align="center">

**Built in public, for people learning in public.**

⭐ Star it · 🍴 Fork it · 🤝 Improve it

*"You don't have to learn all of backend. You just have to learn the right 20%."*

</div>
