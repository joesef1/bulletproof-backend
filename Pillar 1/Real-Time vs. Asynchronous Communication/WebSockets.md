# 🔌 WebSockets: One Open Line, Talking Both Ways

> **One-sentence summary:** A WebSocket is a single, long-lived TCP connection — opened with one HTTP handshake — after which client and server can send messages to each other **anytime, in both directions**, with almost zero overhead per message.

---

## Why WebSockets Exist (The Problem)

Normal HTTP is like **sending letters**: you write (request), wait for a reply (response), and the conversation ends. Every new sentence = a whole new letter with envelope, stamp, address (headers, TLS, TCP setup…).

Now imagine a **chat app, a live game, a stock ticker, a collaborative editor**. Letters won't cut it:

| HTTP pain for real-time apps | What it costs you |
|---|---|
| One request → one response, then done | Can't receive anything unless you asked first |
| Headers on every single message (~1 KB) | Typing "hi" costs 1000× more bytes than the message |
| Polling for freshness | Latency + wasted requests (see `Polling (Short & Long).md`) |
| Server can never speak first | No true "push" |

**WebSockets fix all of this** with one idea: *stop hanging up the phone.* Open the connection once, then just… talk.

---

## 🎭 The Classic Analogy

- **HTTP** ✉️ = sending letters back and forth. Formal, slow, overhead every time.
- **Polling** 🪝 = calling every 10 seconds asking "anything new?"
- **WebSocket** 📞 = a **phone call**: you dial once (handshake), then *both sides just talk whenever they want* until someone hangs up.

---

## 🖼️ Where WebSocket Fits — See It

> This diagram from **ByteByteGo** puts all four real-time techniques side by side: short polling keeps re-asking, long polling holds one request open, SSE streams one-way from server → client, and **WebSocket is the only full-duplex one — both sides talk freely over a single persistent connection.**

![Short/Long polling, SSE, WebSocket — ByteByteGo](https://assets.bytebytego.com/diagrams/0337-short-long-polling-sse-websocket.jpeg)

📎 Source: [`Short/long polling, SSE, WebSocket` guide — ByteByteGo](https://github.com/ByteByteGoHq/system-design-101/blob/main/data/guides/shortlong-polling-sse-websocket.md)

---

## ⚙️ How a WebSocket Actually Works (step by step)

It happens in **two phases**: a polite HTTP introduction, then the real conversation.

### Phase 1 — The Handshake (speaks HTTP, once)

1. Client opens a TCP connection (plus TLS first if `wss://`) and sends a normal-looking HTTP `GET` with special headers:
   ```http
   GET /chat HTTP/1.1
   Host: example.com
   Upgrade: websocket
   Connection: Upgrade
   Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
   Sec-WebSocket-Version: 13
   ```
   Translation: *"Hey server, can we stop doing request/response and switch to WebSocket?"*
2. Server agrees → replies:
   ```http
   HTTP/1.1 101 Switching Protocols
   Upgrade: websocket
   Connection: Upgrade
   Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
   ```
   (`101` = "yes, switching". The `Accept` value is the client's key hashed with a magic GUID — proof the server *understood* WebSocket, not a confused proxy.)
3. Handshake done. **HTTP is never spoken on this connection again.**

### Phase 2 — Frames (tiny, fast, both directions)

4. From now on, both sides exchange **frames** — tiny packets with ~2–14 bytes of overhead (vs ~1 KB of HTTP headers). Types: **text**, **binary**, **ping**, **pong**, **close**.
5. Either side sends anytime — no asking, no waiting. Server pushes a chat message the millisecond it arrives; client sends keystrokes just as freely.
6. Someone sends a **close frame** (or the TCP drops) → connection ends.

**Real example — browser client:**
```js
const socket = new WebSocket("wss://example.com/chat"); // wss = websocket over TLS
socket.onopen = () => socket.send("hello server 👋");   // client → server
socket.onmessage = (e) => console.log("server says:", e.data); // server → client, anytime
```

---

## 🧠 Backend Developer Depth — What Nobody Tells Beginners

### 1. `ws://` vs `wss://` — same rule as HTTP vs HTTPS 🔐
`ws://` is plaintext, `wss://` runs over TLS. **Always use `wss://` in production** — browsers even block insecure `ws://` from HTTPS pages (mixed content). Recall `TLS-SSL-Handshake.md`: TLS happens *before* the HTTP Upgrade on the same connection.

### 2. WebSocket servers are STATEFUL — and that changes everything 🧠
Normal HTTP handlers are stateless: any server can answer any request. A WebSocket connection **lives on one specific server process** for minutes/hours. Consequences:
- **You can't just add servers behind a round-robin load balancer** and call it a day: if User A is connected to Server 1 and User B to Server 2, a message from A never reaches B — unless the servers talk to each other.
- The standard fix: a **pub/sub backbone** (Redis Pub/Sub is the classic). Every server *publishes* incoming messages to Redis and *subscribes* to broadcast them to its own connected clients.
- Load balancers need **sticky sessions** (or IP-hash) so reconnects land sensibly, plus special WebSocket-aware proxy config (Nginx `Upgrade`/`Connection` header passthrough).

### 3. Auth is awkward — plan for it 🔑
The browser's `new WebSocket()` **can't set custom headers** (no `Authorization:` header!). Common workarounds:
- Pass the token in the URL query (`wss://host/chat?token=...`) and validate it *during the handshake*, before accepting.
- Or accept, then require an auth message as the **first frame**, and drop the connection if it never comes / is invalid.
- Cookies *are* sent with the handshake (same-origin), so cookie-session auth works naturally.

### 4. Heartbeats: connections silently die 💓
Proxies, NATs, and mobile networks kill idle connections without telling you. Both sides should exchange **ping/pong frames** (e.g., every 25–30s); if a pong doesn't come back, close and reconnect. Most libraries do this for you — **don't turn it off**.
- Client-side: add **reconnect with backoff** (1s → 2s → 4s… + jitter). Networks *will* drop; your UX shouldn't.

### 5. One connection ≠ one user action — design a mini-protocol 📦
A raw socket is just a pipe. Real apps layer a tiny message protocol on top, e.g. JSON envelopes:
```json
{ "type": "chat.message", "room": "general", "text": "hello!" }
{ "type": "typing", "user": "ana" }
```
Libraries like **Socket.IO** (rooms, auto-reconnect, polling fallback) or Django's **Channels** (groups, channel layers) give you this out of the box. Raw `ws` = maximum control, maximum DIY.

### 6. Backpressure & limits ⚠️
A fast producer + slow consumer = exploding memory. Set **max message size**, **rate limits per connection**, and cap connections per IP. A malicious client sending 100 MB frames or 10k connections is a trivial DoS if you don't.

### 7. Don't use WebSockets for everything
If data flows **only server → client** (news feed, stock ticker, notifications), **SSE** is simpler (plain HTTP, auto-reconnect, works through more proxies). If updates are rare/slow, **polling or webhooks** win. WebSocket earns its complexity when you need **low-latency + bidirectional**.

---

## 🛠️ Real Tools & Commands

- **Test a socket from the terminal with `wscat`:**
  ```bash
  npm install -g wscat
  wscat -c wss://echo.websocket.org   # public echo server
  > hello                             # type this, server echoes it back
  ```
- **Minimal Node server (the `ws` library):**
  ```js
  import { WebSocketServer } from "ws";
  const wss = new WebSocketServer({ port: 8080 });
  wss.on("connection", (socket) => {
    socket.on("message", (msg) => {
      for (const client of wss.clients) client.send(`echo: ${msg}`); // broadcast
    });
    socket.send("welcome 👋");
  });
  ```
- **Python/Django path:** **Django Channels** (ASGI + channel layers + Redis) turns consumers into `connect / receive / disconnect` handlers. **FastAPI** supports `websocket` routes natively via Starlette.
- **Inspect frames in DevTools:** Chrome → Network tab → filter `WS` → click the connection → *Messages* sub-tab shows every frame live. Best debugging view you'll ever get.
- **Nginx snippet** (don't forget this in prod — plain proxying breaks the Upgrade):
  ```nginx
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection "upgrade";
  ```

---

## 🔥 Cheat Sheet — Picking the Right Tool

| Need | Pick |
|---|---|
| Chat, games, live collaboration, trading (two-way + instant) | **WebSocket** ✅ |
| One-way server → browser stream (feeds, tickers, progress) | SSE |
| Server → your backend on rare events | Webhooks |
| Slow/rare updates, simplest infra | Polling |

---

## 🧲 How to Memorize

- **"101 = one-on-one"** — status `101 Switching Protocols` means the line is now a private 1-to-1 conversation.
- **Upgrade = "up-grade the relationship"** 💍 — from formal letters (HTTP) to an open phone line.
- **Full-duplex = "full duo"** 🎤🎤 — two microphones, both live at once. (SSE = one mic, server only.)
- One-liner: **HTTP asks and answers. WebSocket just talks.**

---

## ✅ Quick Self-Check

<details>
<summary>1. What are the two phases of a WebSocket connection?</summary>

(1) **Handshake** — one HTTP `GET` with `Upgrade: websocket` → server replies `101 Switching Protocols`. (2) **Frames** — both sides exchange tiny text/binary/ping/pong/close frames over the same TCP connection, no HTTP anymore.
</details>

<details>
<summary>2. Why can't you naively scale WebSocket servers behind a load balancer?</summary>

Connections are **stateful** — each lives on one specific server. A message arriving at Server 1 can't reach a client connected to Server 2 unless servers share via a pub/sub backbone (e.g., Redis), typically with sticky sessions on the LB.
</details>

<details>
<summary>3. `new WebSocket()` can't set custom headers. How do you authenticate?</summary>

Pass the token in the query string and validate during the handshake, or require an auth JSON message as the first frame (drop the connection if invalid). Cookies work too since the handshake is HTTP.
</details>

<details>
<summary>4. What are ping/pong frames for?</summary>

**Heartbeats.** Middleboxes silently kill idle connections; regular pings detect dead sockets so both sides can close and reconnect instead of talking into the void.
</details>

<details>
<summary>5. When should you choose SSE or polling instead of WebSockets?</summary>

One-way server→client streams → **SSE** (simpler, plain HTTP). Rare/slow updates or no persistent connections allowed → **polling/webhooks**. WebSocket earns it for low-latency **bidirectional** traffic.
</details>