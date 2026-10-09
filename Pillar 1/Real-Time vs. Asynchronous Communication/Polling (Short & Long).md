# ⏳ Polling (Short & Long): Asking Until It's Ready

> **One-sentence summary:** Polling is the client repeatedly asking the server "got anything new?" — **short polling** re-asks every X seconds (fresh HTTP request each round), while **long polling** keeps one request open and holds the server's response until new data actually exists.

---

## Why Polling Exists (The Problem)

An HTTP server can never just *call* you out of the blue — the client always initiates the conversation. So if you want live updates on a normal HTTP connection, the browser/app **has to be the one who keeps asking**.

Polling is the "dumb but simple" solution: **keep asking until the answer changes.**

---

## 🎭 The Classic Analogy

- **Short polling** 🪝 = fishing: you cast the line, check if something bit, reel in, cast again… every few seconds. Even if the lake is empty, you keep checking.
- **Long polling** 📞 = "hold the line, I'll get back to you": you call the restaurant and *don't hang up* until the order is ready. When it's ready, that same (single) call delivers the news.

Short polling = many short calls. Long polling = one long call that stays open.

---

## 🖼️ The Two In Action — See It

> This diagram from **ByteByteGo** compares short polling, long polling, SSE, and WebSockets side-by-side.

![Short/Long polling, SSE, WebSocket — ByteByteGo](https://assets.bytebytego.com/diagrams/0337-short-long-polling-sse-websocket.jpeg)

📎 Source: [`Short/long polling, SSE, WebSocket` guide — ByteByteGo](https://github.com/ByteByteGoHq/system-design-101/blob/main/data/guides/shortlong-polling-sse-websocket.md)

---

## ⚙️ How Short Polling Works (step by step)

1. Client sends a normal HTTP `GET` request → server responds immediately (often "nothing new").
2. Client waits a fixed interval (e.g. 1–2 seconds).
3. Client sends **another** fresh HTTP request to the same endpoint.
4. Repeat forever — every round is a brand-new connection, request, and response.

**Real example — mailbox check:**
```js
setInterval(async () => {
  const res = await fetch("/api/job-status/123");   // brand new request each time
  const { status } = await res.json();
  if (status === "done") console.log("Job finished!");
}, 2000); // ask again every 2 seconds
```

## ⚙️ How Long Polling Works (step by step)

1. Client sends an HTTP `GET` request… and **the server holds the connection open** — no immediate response.
2. The server's code *waits* until either:
   - new data arrives → respond **with the data**, **or**
   - a timeout hits (e.g. 30–60s) → respond with `200` + `empty`/`504`, whichever the design prefers.
3. If the client got empty/timeout, it **immediately re-issues a new long request** (so there's *always* one pending request held open).
4. Data arrives → response returned on the already-open connection → client processes → opens a new long request.

**Real example — chat mailbox with available messages:**
```js
app.get("/messages", async (req, res) => {
  const message = await waitForNewMessage(req.query.lastId, 30_000); // holds ~30s
  res.json(message ?? {});   // data, or empty after timeout
});
```

---

## ⚖️ Short vs Long Polling — Battle Card

| | Short polling | Long polling |
|---|---|---|
| HTTP requests | New request every X seconds | One request held open, tiny gaps between |
| Server response timing | Immediate (even if nothing new) | Delayed until data exists (or ~timeout) |
| Latency of updates | Up to **interval length** (1s, 5s…) | **Near-instant** once data appears |
| Server + network load | High — constant fresh requests | Low — most of the time requests are idle-enough |
| Use case | Job status checks, dashboards, "safe to update" | Chat, notifications, events that are rare |
| Complexity | Trivial | Slightly harder (server holds connections, timeouts) |

**The swap:** short polling saves the server work (instant empty replies) at the cost of *many* requests. Long polling fewer requests but the server must hold connections open — which can pile up.

---

## 🧠 Backend Developer Depth — What Nobody Tells Beginners

### 1. Short polling is a config knob, not a feature
Real apps tune the **interval** to balance freshness vs cost:
- Every **1s** → chatty but fresh (bad for thousands of clients).
- Every **30–60s** → cheap but stale (fine for job progress).

Many providers (Stripe, GitHub) expose polling intervals in their docs & SDKs — work *with* their recommended values, don't hammer.

### 2. Long polling is hard to mis-implement — common traps
- **Timeout management:** server must enforce a max hold (set the request `timeout`, respond empty on expiry), or dead sockets and proxy timeouts kill your connections.
- **Hold connections = hold memory.** Each held request occupies a socket + a thread/worker on some stacks. Thousands of pollers = thousands of held resources (pick an async stack like Node/Go, rarely a thread-per-connection one).
- **Proxies/load balancers** also have idle timeouts — keep the hold below your LB's timeout, or you'll get `504`s.

### 3. Long polling ≈ "the poor man's WebSocket"
It delivers **push-like freshness without WebSockets** — useful where real-time protocols are blocked or overkill (some corporate firewalls, serverless lambdas with websocket limits). But it's **one-way** only and still costs a request per event.

### 4. Always pass a cursor / `since` marker
Both styles should tell the server *what the client already has* (`?afterId=123` or `?since=<clientTime>`), so the server can immediately say "nothing new." Without it every poll re-sends the whole world.

### 5. Memcached/Redis is your friend
When many clients poll the same state, let the server handler check a cheap cache key (Redis `GET job:123`) instead of hitting the DB every poll — that's what makes short polling survivable at scale.

### 6. Beware the thundering herd 🐘
If 10,000 clients short-poll every 5s, your DB hits *all at the same second*. **Jitter** the clients' intervals (`interval + Math.random()*2000`) so requests spread out.

### 7. Know when polling is wrong
If you need **millisecond** updates and **bidirectional** messages, reach for **WebSockets** (our next topic). If you only need **one-way server→client** updates, **SSE** is often cleaner than polling. Polling wins when: simple infrastructure, no open connections allowed, or provider doesn't support push.

---

## 🛠️ Real Tools & Commands

- **Polling a job-status API with curl** (short polling, manual):
  ```bash
  while true; do
    curl -s https://api.example.com/jobs/123 | jq '.status'
    sleep 2   # the "interval"
  done
  ```
- **Node `setTimeout` loop** (long polling client loop):
  ```js
  async function pollLong() {
    const res = await fetch("/messages?lastId=88", { signal: AbortSignal.timeout(35000) });
    const data = await res.json();
    if (data.msg) handle(data.msg);
    await pollLong(); // re-issue instantly => the "long" chain
  }
  pollLong();
  ```
- **Serverless note:** many serverless hosts (AWS Lambda behind API Gateway) cap request durations (~30s), so long-polling often doesn't fit – short polling is the serverless-friendly default.
- **Alternatives to watch for:** Redis-based pub/sub or `EventSource` (SSE) — when you design the backend, prefer them over polling where possible.

---

## 🔥 Cheat Sheet — When to Use Which

| Need | Pick |
|---|---|
| Job status, downloads, build progress (few seconds freshness OK) | **Short polling** |
| Notifications, rare events, want push feel on HTTP only | **Long polling** |
| Fast one-way updates from server | **SSE** (next Pillar topic) |
| Full-duplex chat/games/editors | **WebSockets** (next topic) |

---

## 🧲 How to Memorize

- **Short = Short has more rotations** — many quick HTTP round-trips.
- **Long = the connection Laaaaasts** — one request held open, waiting.
- **Polling P-lays the same question:** "P-lease, anything new?" 🪝 Penguin asking, every round.
- One-liner: **Short polling wears out the network; long polling wears out the connections.**

---

## ✅ Quick Self-Check

<details>
<summary>1. What makes long polling "long"?</summary>

The server deliberately **holds the response** — not returning until new data appears (or a timeout), instead of instantly saying "nothing new." The client then immediately re-opens a new request.
</details>

<details>
<summary>2. Which has higher latency and why? Short or long polling?</summary>

Short polling: updates arrive only at the next polling tick (up to the full interval late). Long polling delivers the moment the data exists — so it's fresher.
</details>

<details>
<summary>3. Name the biggest operational cost of long polling.</summary>

Held-open connections: every open request occupies a socket + memory, and must respect proxy/LB idle timeouts or you get 504s.
</details>

<details>
<summary>4. Why add jitter to client poll intervals?</summary>

To avoid a **thundering herd** — thousands of clients hitting the DB in the same second. Spreading intervals smooths the load.
</details>

<details>
<summary>5. When is polling a poor choice, and what should replace it?</summary>

When you need near-real-time **bidirectional** comms → WebSockets; one-way fast server→client → SSE. Polling fits simple, connection-free infrastructure.
</details>