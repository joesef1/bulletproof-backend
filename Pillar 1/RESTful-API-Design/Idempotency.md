# Idempotency: Why Repeating a Request Is (Usually) Not Scary

## The One-Sentence Summary

**An operation is *idempotent* when calling it 1, 10, or 10,000 times with identical input produces the *exact same end state* as calling it once — and understanding which requests are idempotent is what lets you safely retry a failed API call without creating duplicate users, duplicate orders, or double-charging a credit card.**

The whole internet quietly depends on this idea. If your code isn't idempotent where it should be, production will find out — at the worst possible moment.

---

## The Definition, in One Analogy

> **The elevator button.** Pressing the "floor 5" button once lights it up. Pressing it 47 more times? Still just floor 5. The *end result* is the same no matter how many times you press. **That's idempotent.**
>
> **The vending machine.** Pressing the "buy coke" button once gives you one coke. Pressing it twice gives you *two* cokes. The *end result changes with each press*. **That's NOT idempotent.**

Idempotency is about the **final server state** — not about what the *response* looks like. A duplicate request can still fail, or return a "gone / already deleted" message — the important part is that *it doesn't do the work twice.*

---

## The Money Question: Retrying a Failed Request

Here's the real reason idempotency matters. Your client sends a request. The network hiccups. Your client doesn't get a response. **What should it do?**

Look at this figure — it shows, at a glance, which methods you can safely retry:

![Top 9 HTTP Request Methods — whether each is idempotent — diagram by ByteByteGo](https://assets.bytebytego.com/diagrams/0371-top-9-http-request-methods.png)

*Diagram source: [ByteByteGo System Design 101 — Top 9 HTTP Request Methods](https://github.com/ByteByteGoHq/system-design-101)*

Now the critical distinction:

| Situation | Is it idempotent? | Safe to blindly retry? |
|-----------|-------------------|------------------------|
| You called `GET /users/123` and got no response | ✅ Yes | **Yes** — retrying just reads again |
| You called `DELETE /users/123` and got no response | ✅ Yes | **Yes** — deleting a already-deleted user is harmless (the "already deleted" case is exactly the idempotent outcome) |
| You called `PUT /users/123` and got no response | ✅ Yes | **Yes** — replacing the whole object twice = same result |
| You called `POST /payments` and got no response | ❌ **No** | **DANGER** — was it processed? You don't know. Blindly retrying could double-charge. |

> **The core insight:** a network failure means *you don't know if the server received the request*. Idempotency is the trick that makes "send it again" safe in that exact moment of doubt.

---

## Which Methods Are Idempotent (The Table You'll Reuse Forever)

| Method | Safe (no side effects)? | Idempotent (repeat = same state)? | Cacheable? |
|--------|:---:|:---:|:---:|
| `GET` | ✅ | ✅ | ✅ |
| `HEAD` | ✅ | ✅ | ✅ |
| `OPTIONS` | ✅ | ✅ | ❌ |
| `TRACE` | ✅ | ✅ | ❌ |
| `PUT` | ❌ | ✅ | ❌ |
| `DELETE` | ❌ | ✅ | ❌ |
| `POST` | ❌ | ❌ | conditional |
| `PATCH` | ❌ | **depends on design** | conditional |

Rules to internalize:

1. **Every safe method is idempotent** (see `GET`, `HEAD`, `OPTIONS`).
2. **`PUT` and `DELETE` are idempotent but NOT safe** — they change state, but repeating them doesn't change it *further*. Deleting a deleted resource, or setting a value to itself, is the same as doing it once.
3. **`POST` is the odd one out** — it creates something *new* each time. This is the method that needs the most engineering care.
4. **`PATCH` is special** — it's only idempotent if designed to be. Read on.

---

## PATCH and the "Increment Trap"

The RFC rule for PATCH: a method is idempotent if the *intended effect* of N identical requests equals the effect of one request. Whether PATCH qualifies *depends entirely on the body you send*.

Compare these two PATCH payloads:

```
PATCH /accounts/123   {"balance": 500}      ← setting a VALUE
```
Call it twice → balance is 500 both times. **Idempotent.** ✅

```
PATCH /accounts/123   {"balance": "+=100"}  ← doing a RELATIVE change
```
Call it twice → balance grew by 200 instead of 100. **NOT idempotent.** ❌

So:
- **Setting** a field to a value = idempotent.
- **Adding/incrementing/appending** = not idempotent (each call adds more).

> **The rule of thumb:** if your operation means *"make the state BE X"* → idempotent. If it means *"add to the state"* → not idempotent. This binary applies to everything in system design — not just HTTP.

---

## How Real Systems Handle Non-Idempotent POSTs: Idempotency Keys

The HTTP protocol can't make `POST` idempotent. So *you* do — with an **idempotency key**.

The pattern is famous (Stripe's API made it mainstream):

```
1. CLIENT generates a unique key (UUID) for each logical attempt:

   POST /payments
   Idempotency-Key: 9f4c1a2e-...            ← unique per logical operation

   {"amount": 1000, "currency": "USD"}

2. SERVER stores { key → result } before/while processing:

   - First request with this key  → process the payment, STORE the result under the key.
   - Retry with the SAME key      → don't process again; just RETURN the stored result.

3. Any number of retries with the same key return the SAME response.
   A DIFFERENT key = a NEW operation (a new payment!).
```

**Why it works:** the key makes the *client's logical operation* (not the request) the unit of idempotency. The server remembers "this payment already happened" even though `POST` itself isn't idempotent.

**Backend rules for idempotency keys:**
- The server must **deduplicate on the key** (usually with a unique DB constraint — so two racing duplicate requests can't both slip through).
- The **response is cached** under the key so a retry returns the original result (status + body), not a different one.
- The key has an **expiry** (e.g., 24h) so you don't store them forever.
- The key must be scoped to the **user + endpoint** (User A's `9f4c...` isn't User B's).

---

## The "Idempotency Key in 4 Steps" Flow

```
 CLIENT                                     SERVER
   │                                           │
   │  1. POST /payments                       │
   │     Idempotency-Key: key-123 ──────────►  │
   │                                           │  2. Check DB for key-123
   │                                           │     ├─ FOUND → return STORED response
   │                                           │     └─ NOT FOUND →
   │                                           │        process + STORE response
   │  3. ◄──────────────────────────────────────── (network dies here!)
   │                                           │
   │  4. Client retries with the SAME key      │
   │     POST /payments                       │
   │     Idempotency-Key: key-123 ──────────►  │  → key FOUND →
   │                                           │     return STORED response
   │  5. ◄──────── same response, NO 2nd charge ┘
```

**The memory hook:** the idempotency key is like a **receipt number**. The cashier asks "do you have your receipt?" — if you show the same receipt, you don't pay again. Show a *new* receipt, new payment.

---

## Beyond HTTP: Idempotency Everywhere

This concept isn't just about methods — it's a **core systems principle**. You'll meet it constantly:

| Context | Idempotent version |
|---------|--------------------|
| **Database** | `INSERT ... ON CONFLICT DO NOTHING` or `UPDATE SET x = 5` (set a value) |
| **Message queues** | At-least-once delivery + **consumer deduplication** (process a message once even if delivered twice) |
| **Retries in microservices** | Retry with `Retry-After` + idempotency keys, not blind re-execution |
| **File uploads** | Same file with same checksum → don't re-upload |
| **Cron jobs** | Running twice shouldn't send two emails — use dedup flags |

The pattern to remember across all of them: **either make the operation naturally idempotent, or make it idempotent by recording what you already did.**

---

## How to Memorize Idempotency

### The one-liner definition
> **"The result after N tries is the same as after 1."**

### The 3 buckets
> - **Safe + Idempotent:** `GET`, `HEAD`, `OPTIONS` *(just reading)*
> - **Idempotent but moody:** `PUT`, `DELETE` *(changes state, but once is enough)*
> - **Feels dangerous:** `POST` *(creates something new every time)*

### The "B.E." test
> **B**e = idempotent. **B**ecome = not.
> "Make it **B**e X" — repeat it all day, it's still X. ✅
> "Make it **B**eco-me X" — each retry changes it. ❌

### The interview phrase to drop
> "POST isn't idempotent, so we make it idempotent with a unique idempotency key and a database-level unique constraint — so retries auto-deduplicate."

That sentence demonstrates you understand *both* the protocol and the systems-level fix. Say it, and the interviewer nods.

---

## Quick Self-Check (Interview-Style Questions)

1. Define idempotency in one sentence.
2. `DELETE /orders/456` returned no response due to a network failure. Should the client retry? Why?
3. Which methods are idempotent but NOT safe, and what does that mean?
4. Why isn't `PATCH` considered idempotent by default? Give a non-idempotent example.
5. How would you make a `POST /payments` endpoint safely retryable?

<details>
<summary>Answers (try first, then peek)</summary>

1. An operation is idempotent when N identical requests produce the same end state as one request.
2. **Yes** — `DELETE` is idempotent. Retrying it has no additional side effect (worst case: the resource is already gone, which is the intended end state).
3. `PUT` and `DELETE`. They change server state, but repeating them doesn't change it *further* — so the state after 1 call equals the state after N calls.
4. Because a PATCH body can describe a *relative* change: `{"balance": "+=100"}`. Each retry increments again. `{"balance": 500}` would be idempotent because it *sets* rather than *adds*.
5. Require the client to send a unique **Idempotency-Key** header per logical operation. The server stores `key → result` (with a unique constraint), and any retry with the same key returns the original response without processing again.
</details>