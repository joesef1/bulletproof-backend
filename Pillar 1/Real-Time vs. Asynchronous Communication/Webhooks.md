# 🔔 Webhooks: Push, Don't Pull

> **One-sentence summary:** A webhook is a user-defined HTTP callback — a URL on *your* server that an external service automatically calls (with a `POST` request) whenever an event happens, so new data is **pushed** to you instead of you having to go ask for it.

---

## Why Webhooks Exist (The Problem)

Imagine you order food delivery. Two possible worlds:

- **Polling world:** You call the restaurant every 30 seconds — "Is it ready yet? Now? Now? NOW?" You waste a phone call each time, and you still miss the exact moment the food is ready.
- **Webhook world:** You give the restaurant your phone number. The moment your order is ready, *they call you.* You say nothing, do nothing — the news comes straight to you.

That's the whole idea. Polling = the client keeps *asking*. Webhooks = the server *informs*.

**The problem webhooks solve:** In backend systems, "checking every few seconds for updates" is wasteful:

| Pain point of polling | What it costs you |
|---|---|
| Wasted requests | Millions of HTTP calls that return "nothing changed" |
| Latency | You only learn about an event *up to X seconds after* it happened |
| Load on their servers | Unnecessary traffic on the API you're polling |
| Not truly real-time | Updates are always at least one polling-interval late |

Solutions that *push* (webhooks / SSE / WebSockets) eliminate nearly all of that.

---

## 🎭 The Classic Analogy

- **Polling** 🪝 = fishing. You throw a line in and *wait and check, check, check.*
- **Webhook** 📞 = a phone call. *"Call me when something changes."*

Both get you the fish — but only one lets you take a nap in between.

---

## 🔄 Webhook vs Polling — See It

> This diagram from **ByteByteGo** shows the two approaches side by side: polling keeps *asking* the server ("any news?"), while the webhook lets the server *call you back* when there's news.

![Polling vs Webhook diagram — ByteByteGo](https://assets.bytebytego.com/diagrams/0057-pooling-vs-webhook.png)

📎 Source: [`Polling vs Webhooks` guide — ByteByteGo](https://github.com/ByteByteGoHq/system-design-101/blob/main/data/guides/polling-vs-webhooks.md)

---

## ⚙️ How a Webhook Actually Works (step by step)

Think of the classic real-world example: **a payment provider (e.g., Stripe).**

1. **You register an endpoint** — you tell Stripe: *"Here is my URL: `https://yourshop.com/api/payments/webhook`."* This registration is called **subscribing to the event** (e.g., `payment.succeeded`).
2. **A customer pays** on a card form that lives on *Stripe's* servers (not yours).
3. **Stripe sees the event** → it builds a JSON payload describing it:
   ```json
   {
     "id": "evt_1P9zKx...",
     "type": "payment.succeeded",
     "data": { "amount": 4999, "currency": "usd", "customer": "cus_123" }
   }
   ```
4. **Stripe sends an HTTP `POST`** to your registered URL, with that JSON body.
5. **Your server receives it**, verifies it's really from Stripe (signature check 🔐), processes it (update DB, send email, unlock the feature).
6. **You reply `200 OK`** quickly, as fast as possible.
7. **If you reply `5xx` or don't reply in time**, Stripe **retries** the delivery (with growing delays) until you acknowledge — or it gives up (after ~3 days for Stripe).

---

## 🧠 Backend Developer Depth — What Nobody Tells Beginners

### 1. Webhooks are REVERSED request/response 📨
Everything you learned in `Request-Response-Cycle.md` is flipped: **you** are now the *server*. Your route only **receives** `POST` bodies. That's why webhook handlers are usually the simplest code you'll write — but the security around them is not.

### 2. Anyone can hit your webhook URL — VERIFY THE SIGNATURE 🔐
Your webhook is a **public URL**. Attackers can and will POST fake events to it (e.g., "payment succeeded" when nobody paid). The fix: providers sign every payload (Stripe uses HMAC-SHA256 with a shared secret).

```python
# Stripe-style signature check (pseudo-code, real libs exist)
signature = request.headers["Stripe-Signature"]
expected = hmac_sha256(secret, timestamp + "." + raw_body)
if not hmac.compare_digest(expected, signature):
    raise Unauthorized("not from Stripe!")
```

Rules:
- **Never** use the parsed JSON to verify — hash the **raw body as-bytes**, or the signature breaks.
- **Replay protection** — providers often include a timestamp; reject signatures older than a few minutes.

### 3. Webhook handlers MUST be idempotent 🔁
Remember Idempotency? Here's where it bites. Providers **retry on network failures / timeouts**, so the *same event* can arrive more than once. Your handler must be safe to run twice:
- Store processed event IDs in a table (e.g., `processed_events`) and return 200 if you've seen `evt_1P9zKx` before.
- Or make your DB writes naturally idempotent (e.g., `INSERT ... ON CONFLICT DO NOTHING`).

### 4. Reply FAST, work LATER ⚡
The provider only waits a few seconds. **Never** do slow work (sending emails, calling other APIs, heavy DB) inside the webhook handler's response path.
- Acknowledge the webhook immediately with `200 OK`.
- Push the event onto a **message queue** (SQS, RabbitMQ, Redis) and process it asynchronously in a worker.

### 5. Always respond — even to errors ⚠️
- Return **200** to stop the retries once you've *accepted* the event.
- Return **4xx** (e.g., `400`) only if the event is permanently invalid (won't retry).
- Return **5xx** if you want the provider to **retry**. In other words, the HTTP status you return *is* your retry control.

### 6. You may want to expose a way to RE-send events 🔄
Many providers let you **replay events from their dashboard**. Use it to backfill data when you had an outage.

### 7. Timeouts and throttling
Your endpoint should respond within ~seconds. If the provider sends bursts, your server needs to handle concurrent `POST`s gracefully (async handler, quick ack).

---

## 🛠️ Real Tools & Commands

- **Testing locally:** [**ngrok**](https://ngrok.com) gives your localhost a public HTTPS URL — superb for dev:
  ```bash
  ngrok http 3000
  # Forwards https://abc123.ngrok.app -> http://localhost:3000
  ```
  Paste your server's webhook URL, run it on `localhost:3000`, and you get a real public callback.
- **Receive webhooks in dev:** Postman's "webhook request" feature, or `webhook.site` — gives you a throwaway URL and logs every payload it receives.
- **Accepting the POST — minimal Express example:**
  ```js
  app.post("/api/payments/webhook", express.raw({ type: "application/json" }), (req, res) => {
    const sig = req.headers["stripe-signature"];
    // 1. verify signature, 2. check event id is not processed, 3. enqueue work
    res.sendStatus(200); // ack fast, work later
  });
  ```
- **Simulating a webhook yourself** on the command line:
  ```bash
  curl -X POST "https://yourshop.com/api/payments/webhook" \
       -H "Content-Type: application/json" \
       -d '{"type":"payment.succeeded","data":{"amount":4999}}'
  ```
- **Sending your OWN webhooks to customers** (you become the provider): libraries like `svix`, `hookdeck`, or `stripe`'s pattern apply.

---

## 🔥 When to use Webhooks vs Polling (cheat sheet)

| Situation | Use | Why |
|---|---|---|
| Payment status, order updates, CI build finished | **Webhook** | Instant, low overhead |
| Your provider *only supports polling* | Polling | No choice |
| Consumers without a public URL / firewalled | Polling | You can't receive pushes |
| You need full control over request flow | Polling | Client pulls when it wants |
| High-throughput event delivery | **Webhook** | Push scales better than millions of polls |

---

## 🧲 How to Memorize

- **"W" = Wait** for the call. Polling starts with **P** = **"Please?"** ... (every few seconds, begging for data). Webhooks let the server do the talking.
- **Pun:** Polling = *you* do the elbow work. Webhooks = *they* do the calling — "the server **pings your web-hook**."
- One-liner: **Polling asks. Webhook tells.**
- **Callback = webhook.** A callback takes time to sink in, but they're the same thing: "a function (URL) the other side calls back."

---

## ✅ Quick Self-Check

<details>
<summary>1. What's the single core difference between polling and webhooks?</summary>

Polling: the **client** keeps making requests to *check* for changes. Webhooks: the **server** sends a `POST` to the client's registered URL when a change happens (push vs pull).
</details>

<details>
<summary>2. Why must you verify a webhook's signature?</summary>

Because your webhook URL is public — anyone could POST a fake event. The signature proves the request genuinely came from the provider (shared-secret HMAC), not an attacker.
</details>

<details>
<summary>3. You return HTTP 500 from your webhook handler. What happens?</summary>

The provider sees the delivery as failed and **retries** later (with backoff). Return **200** to stop retries once you've accepted the event.
</details>

<details>
<summary>4. Why must webhook handlers be idempotent?</summary>

Providers retry failed deliveries, so the same event can arrive more than once. If your handler isn't idempotent, you'll double-process (double-charge, double-emails). Track processed event IDs.
</details>

<details>
<summary>5. Should you do heavy work (send emails) directly inside the webhook handler? Why/why not?</summary>

No. Providers wait only a few seconds before timing out and retrying. Acknowledge with `200 OK` fast, push the event to a queue, and process it in a background worker.
</details>