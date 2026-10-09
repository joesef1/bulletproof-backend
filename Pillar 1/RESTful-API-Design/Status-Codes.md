# HTTP Status Codes: The Server's One-Number Report Card

## The One-Sentence Summary

**Every HTTP response begins with a 3-digit status code that tells the client instantly what happened — the first digit alone reveals which of the 5 outcome classes it falls into (1xx info, 2xx success, 3xx redirect, 4xx your fault, 5xx our fault) — and using them correctly is what makes an API debuggable, automated, and honest.**

When something breaks, the status code is the first sentence of the conversation. If you get that number right, the client can react automatically without reading a message. If you get it wrong, every client has to special-case your API forever.

---

## The Five Categories (The Whole Universe of Outcomes)

Status codes are designed so **the first digit tells you everything**: no memorizing every number needed — know the 5 classes, then slot codes in.

| Class | Range | Meaning in friendly words | Examples |
|-------|-------|---------------------------|----------|
| **1xx** | 100–199 | "Working on it… hold on" | `100 Continue`, `102 Processing` |
| **2xx** | 200–299 | "Success — it worked" | `200 OK`, `201 Created`, `204 No Content` |
| **3xx** | 300–399 | "Moved — go there instead" | `301`, `302`, `304 Not Modified` |
| **4xx** | 400–499 | "You messed up" (client) | `400`, `401`, `403`, `404`, `429` |
| **5xx** | 500–599 | "We messed up" (server) | `500`, `502`, `503`, `504` |

### The memory story (this one's worth autopilot)

> **10x say "hi", 2xx say "yay", 3xx say "go away — but follow me", 4xx say "YOU got it wrong", 5xx say "WE got it wrong".**
>
> Or the kitchen defense: *"1=information, 2=success, 3=redirection, 4=client, 5=server"* → **"1-2-3-4-5: Info-You-Go-Blame-Them-Blame-Us."**

### The money-question trick: "Who's to blame?"

- **4xx** → The **client** can fix it by changing its request.
- **5xx** → The **server** must fix it. The client can only retry or complain.

That single distinction is the most useful diagnostic in all of web development.

---

## The Diagram

This figure from the ByteByteGo System Design 101 series is the classic cheat sheet for the codes that matter:

![HTTP Status Codes You Should Know — diagram by ByteByteGo](https://assets.bytebytego.com/diagrams/0233-http-status-code.png)

*Diagram source: [ByteByteGo System Design 101 — HTTP Status Code](https://github.com/ByteByteGoHq/system-design-101)*

---

## The Codes You'll Use 95% of the Time (Backend Edition)

### 2xx — SUCCESS

| Code | When to return it | Real example |
|------|-------------------|--------------|
| `200 OK` | The generic "it worked" — a `GET` or `PUT` returning a resource | `GET /users/123` → user JSON |
| `201 Created` | A **new resource** was created (a `POST` succeeded) | `POST /users` → new user + `Location` header |
| `204 No Content` | Request succeeded, but **nothing to send back** | `DELETE /users/123` → done, no body |

> **Backend rule:** `POST` → **201** with a `Location` header. `DELETE` → **204** (or 200 with body). `GET` → **200**. `PATCH`/`PUT` → **200** (or 204).

### 3xx — REDIRECTION

| Code | When to return it | Real example |
|------|-------------------|--------------|
| `301 Moved Permanently` | The URL **permanently** changed — update your bookmarks/clients | `http://` → `https://` |
| `302 Found` | **Temporary** move — keep using the old URL | "Just for this session, go here" |
| `304 Not Modified` | Client has a cached copy — "use that, it's fresh" | Conditional `GET` with `If-None-Match` |

> **Backend rule:** clients (and browsers) **auto-follow** 3xx. Automatically redirecting users after a `POST` (PRG pattern) is a classic pattern that prevents accidental double-submission on refresh.

### 4xx — CLIENT ERROR (You messed up)

| Code | When to return it | Real example |
|------|-------------------|--------------|
| `400 Bad Request` | Malformed/unparseable request | Invalid JSON body, missing required field |
| `401 Unauthorized` | **Not authenticated** — who are you? | Missing/invalid token |
| `403 Forbidden` | **Authenticated but not allowed** — I know you, and no | Logged in as "user", asking for admin data |
| `404 Not Found` | Resource doesn't exist | Typo in URL, deleted item |
| `409 Conflict` | Request clashes with current state | Duplicate email on signup, version mismatch |
| `429 Too Many Requests` | **Rate-limited** — slow down | API abuse throttling |

> **401 vs 403 — the eternal interview question:**
> - `401` = **"I don't know who you are"** → authenticate first (login, re-send token).
> - `403` = **"I know exactly who you are, and you're not allowed"** → no point retrying; your permissions are wrong.

### 5xx — SERVER ERROR (We messed up)

| Code | When to return it | Real example |
|------|-------------------|--------------|
| `500 Internal Server Error` | Unexpected server crash | An unhandled exception, DB outage |
| `502 Bad Gateway` | A **proxy** got a bad response from an upstream server | Nginx can't reach Gunicorn |
| `503 Service Unavailable` | Server is overloaded / in maintenance | Too many requests, deploy window |
| `504 Gateway Timeout` | An upstream **didn't answer in time** | Gunicorn hung, DB query too slow |

> **Backend rule:** never leak internals in 5xx bodies. Return a generic "something went wrong" message and log the *real* error server-side (or fire it to Sentry). Exposing exception tracebacks in production is a security hole.

---

## The Status Code Decision Flow (How to Pick the Right One)

When you write an endpoint, walk this chain:

```
Did the request reach your code? 
 ├─ No  → 404 (URL doesn't exist)  or 405 (method not allowed)
 └─ Yes →
      Is the client authenticated?
       ├─ No → 401
       └─ Yes → Is the client authorized for THIS resource?
            ├─ No → 403
            └─ Yes → Is the request valid (schema, types, ids)?
                 ├─ No → 400  (or 409 for a state conflict, 422 for validation)
                 └─ Yes → Did the operation succeed?
                      ├─ Yes → 200 / 201 / 204 (depending on what happened)
                      └─ No → 500 (unknown) / 503 (overloaded) / etc.
```

This is the "skeleton" of correct status-code usage. Internalize the ordering and you'll stop returning `500` for every error like most junior code does.

---

## The Two "One More Thing" Codes Every API Needs

### 405 Method Not Allowed
The URL exists, but the *verb* doesn't apply:

```http
GET /orders/123 → 200
DELETE /orders/123 → 405 Method Not Allowed   ← you only support GET on this path
```
Return it automatically with the allowed methods in the `Allow` header (`Allow: GET, PUT`). Django REST Framework does this for you.

### 422 Unprocessable Entity
Payload is *valid JSON* but the *contents* are wrong (e.g., a required field is `null`, an email is malformed):

```json
HTTP/1.1 422 Unprocessable Entity

{"email": ["Enter a valid email address."]}
```

> **Note on 400 vs 422:** some frameworks (like old Django versions) use `400` for everything. Modern practice pushes structural validation errors to `422` so clients can distinguish "your JSON is broken" from "your data is semantically wrong."

---

## How to Memorize Status Codes

### The 5-classes story (already above)
> **"1-info, 2-success, 3-redirect, 4-yours, 5-ours"**

### The "must-know" grid — one code per class, hung on a real moment
> - **1xx** `100 Continue` — *"please continue sending"*
> - **2xx** `201 Created` — *"made a thing"*
> - **3xx** `304 Not Modified` — *"you have the fresh copy already"*
> - **4xx** `404 Not Found` — *"not here, and it's your URL"*
> - **5xx** `503 Unavailable` — *"can't do it right now, try later"*

### The idempotent-verb↔code recipe
> - **POST → 201** (created something new)
> - **DELETE → 204** (gone, nothing back)
> - **GET → 200** (here's the data)
> - **PUT/PATCH → 200 or 204** (updated, data or no-data)

### The one-line "blame" rule
> **"4xx = the request is wrong. 5xx = the server is wrong. Never return 500 for a client's mistake."**

---

## Quick Self-Check (Interview-Style Questions)

1. What are the 5 status-code classes, and what does each one communicate?
2. `401` vs `403` — what's the real difference?
3. A `POST /users` succeeds. Which status should your backend return, and what header?
4. Your nginx returns `502 Bad Gateway`. What does that tell you about your stack?
5. Why must you never return `500` for a client's bad input?

<details>
<summary>Answers (try first, then peek)</summary>

1. **1xx** informational ("working on it"), **2xx** success, **3xx** redirection, **4xx** client error, **5xx** server error.
2. `401` = **not authenticated** (server doesn't know who you are). `403` = **not authorized** (server knows you, but you lack permission).
3. **`201 Created`** plus a **`Location`** header pointing to the new resource.
4. The proxy (nginx) reached an upstream (e.g., Gunicorn) that returned a bad/unreachable response — your **app server or its dependency** is down.
5. Because `500` means "the server crashed" — it hides whether the *client* made the mistake and breaks automation/caching/monitoring. Client-side problems must get 4xx codes.
</details>