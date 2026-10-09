# HTTP Verbs: The Five Words That Run Every API

## The One-Sentence Summary

**HTTP methods (also called *verbs*) tell the server what *action* the client wants to perform on a resource — `GET` (read), `POST` (create), `PUT` (replace), `PATCH` (partially update), `DELETE` (remove) — and sticking to them makes every API self-documenting, cacheable, and predictable.**

That's the whole point of REST: **the URL is the noun (the resource), and the HTTP method is the verb (the action).**

---

## Why HTTP Verbs Exist (The Problem)

Without HTTP verbs, APIs look like this:

```
POST /getUsers
POST /createUser
POST /updateUser
POST /deleteUser
POST /getOrders
```

Every endpoint is a `POST` with a magic URL path. The same method means nothing — the URL does all the work. There's no way to know whether a request is safe, cacheable, or idempotent just by looking at the method. Browsers can't optimize it. Proxies can't cache it. Developers must read the docs for every endpoint.

REST fixes this by using HTTP's built-in vocabulary properly:

![REST API Architecture — overview of resources, URIs, and HTTP methods](https://restfulapi.net/wp-content/uploads/restful-api.png)

*Diagram source: [restfulapi.net](https://restfulapi.net/http-methods/)*

The same five actions happen in every app you've ever built. Now they have names.

---

## The Five Core Verbs: CRUD in One Table

Every REST API on Earth boils down to four operations — **CRUD** (Create, Read, Update, Delete) — mapped directly onto HTTP verbs:

| CRUD | HTTP Verb | What it does | URL pattern |
|------|-----------|--------------|-------------|
| **Create** | `POST` | Create a **new** resource inside a collection | `POST /users` |
| **Read** | `GET` | Read/retrieve a resource (or collection) | `GET /users/123` |
| **Update (full)** | `PUT` | **Replace** an existing resource entirely | `PUT /users/123` |
| **Update (partial)** | `PATCH` | **Partially modify** a resource | `PATCH /users/123` |
| **Delete** | `DELETE` | Remove a resource | `DELETE /users/123` |

> **Memory hook — "CRUD, CRUD, CRUD… but how?"**
> **C**reate = `POST` · **R**ead = `GET` · **U**pdate = `PUT`/`PATCH` · **D**elete = `DELETE`.
>
> Say it as: *"I **POST** to create, **GET** to read, **PUT** to replace, **PATCH** to tweak, and **DELETE** to erase."*

---

## Each Verb Explained (with a Real Request)

Here's a real HTTP/1.1 request for each verb, annotated with a real-world example — a simple users API.

### GET — Read

```http
GET /users/123 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGci...
```

**What it means:** "Give me the current state of user 123. Don't change anything."

| Property | Value |
|----------|-------|
| **Safe?** | ✅ Yes — it **never modifies** server state |
| **Idempotent?** | ✅ Yes — calling it 10 times returns the same result every time |
| **Cacheable?** | ✅ Yes — browsers, CDNs, and proxies can cache the response |
| **Has request body?** | ❌ No — all info is in the URL (query params) |

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 123, "name": "Mona", "email": "mona@example.com"}
```

> **Backend tip:** never use `GET` to change data (like `GET /deleteUser/123`). Crawlers and browsers pre-fetch GET links. The user will be very surprised.

---

### POST — Create

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGci...

{"name": "Mona", "email": "mona@example.com"}
```

**What it means:** "Add this new user to the `/users` collection. Assign it a new ID."

| Property | Value |
|----------|-------|
| **Safe?** | ❌ No — it changes server state (creates something new) |
| **Idempotent?** | ❌ No — calling it twice creates **two** identical users |
| **Cacheable?** | ❌ No — never cache a mutation |
| **Has request body?** | ✅ Yes — the resource being created |

```http
HTTP/1.1 201 Created
Location: /users/124
Content-Type: application/json

{"id": 124, "name": "Mona", "email": "mona@example.com"}
```

**Backend tip:** always return **`201 Created`** with a `Location` header pointing to the new resource on successful creation. This is REST best practice and what clients expect.

---

### PUT — Full Replace

```http
PUT /users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGci...

{"name": "Mona Amiri", "email": "m.amiri@example.com"}
```

**What it means:** "Replace user 123 **entirely** with this new version. If the user doesn't exist, create it."

| Property | Value |
|----------|-------|
| **Safe?** | ❌ No |
| **Idempotent?** | ✅ Yes — calling it 5 times with the same body = same final state |
| **Has request body?** | ✅ Yes — the **entire** new resource (every field) |

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 123, "name": "Mona Amiri", "email": "m.amiri@example.com"}
```

> **PUT vs POST — the one thing that matters:**
> `POST` means **"add something new, you choose the ID."**
> `PUT` means **"I already know the ID, replace what's there (or create if missing)."**

---

### PATCH — Partial Update

```http
PATCH /users/123 HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGci...

{"email": "newemail@example.com"}
```

**What it means:** "Only change the `email` field on user 123. Leave everything else alone."

| Property | Value |
|----------|-------|
| **Safe?** | ❌ No |
| **Idempotent?** | Technically yes (if same patch applied twice → same state), but not guaranteed by spec |
| **Has request body?** | ✅ Yes — **only** the fields to change |

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 123, "name": "Mona", "email": "newemail@example.com"}
```

> **PUT vs PATCH — the gotcha that trips everyone up:**
>
> `PUT` requires the **full** object in the request body — missing a field = that field becomes `null` or its default.
>
> `PATCH` requires **only the field(s) you're changing** — everything else is untouched.
>
> Most modern APIs favor `PATCH` for updates, because clients rarely need to send the entire object.

---

### DELETE — Remove

```http
DELETE /users/123 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGci...
```

**What it means:** "Remove user 123 from the system."

| Property | Value |
|----------|-------|
| **Safe?** | ❌ No |
| **Idempotent?** | ✅ Yes — deleting twice returns the same result |
| **Has request body?** | ❌ No (typically) |

```http
HTTP/1.1 204 No Content
```

**Backend tip:** on successful `DELETE`, return **`204 No Content`** (no body) or **`200 OK`** with the deleted resource. Don't forget that a soft-delete (setting `deleted_at` timestamp) and a hard-delete (`DELETE FROM users`) are both valid backend strategies — the HTTP method stays the same.

---

## Idempotency: The Concept That Separates Juniors from Seniors

**Idempotent** means: calling the same request 1, 10, or 100 times produces **the exact same server state** as calling it once.

| Verb | Idempotent? | Why |
|------|-------------|-----|
| `GET` | ✅ Yes | Reading doesn't change anything |
| `PUT` | ✅ Yes | Replacing 5 times = same as replacing once |
| `DELETE` | ✅ Yes | Deleting 5 times = same as deleting once |
| `PATCH` | ✅ Yes (usually) | Applying the same partial change repeatedly yields the same state |
| **`POST`** | ❌ **NO** | Creating 5 times = 5 separate resources |

**Why it matters:** when a network request fails, the client can safely **retry** an idempotent request without fear of duplication or side effects. Retrying a `POST` could create a duplicate user, a duplicate order, a duplicate charge. That's why payment APIs (`POST /charge`) use **idempotency keys** — a unique token the client sends, so the server can detect and reject duplicate charges even if the client retries.

> **Interview trap:** "What happens when a non-idempotent request fails mid-flight?"
> Answer: the client doesn't know if the server received it, so it can't blindly retry. `POST` is the only non-idempotent verb — that's why idempotency keys exist for critical operations.

---

## The Safe Methods: Why GET Is Special

**Safe** means: the request **does not modify server state**. It's purely a read.

| Safe? | Verbs |
|-------|-------|
| ✅ Yes | `GET`, `HEAD`, `OPTIONS` |
| ❌ No | `POST`, `PUT`, `PATCH`, `DELETE` |

**Why it matters:** because safe methods don't change state, intermediate systems can aggressively optimize them:
- **Browsers** can prefetch `GET` links.
- **CDNs** cache `GET` responses globally.
- **Proxies** can serve cached `GET` responses without hitting your server.
- **Crawlers** follow `GET` links freely; they ignore `POST`.

Using a safe method for anything unsafe breaks the entire web infrastructure's ability to cache and optimize. That's why you never use `GET` for mutations.

---

## Real-World Patterns (What Every API Actually Uses)

A complete resource in a real Django/FastAPI project:

```
GET    /api/v1/users         → 200  [list of users]
POST   /api/v1/users         → 201  (creates a user)
GET    /api/v1/users/123     → 200  (single user)
PUT    /api/v1/users/123     → 200  (full update)
PATCH  /api/v1/users/123     → 200  (partial update)
DELETE /api/v1/users/123     → 204  (user removed)
```

And when you build a nested resource (orders belonging to a user):

```
GET    /api/v1/users/123/orders        → 200  (user's orders)
POST   /api/v1/users/123/orders        → 201  (create an order for user 123)
DELETE /api/v1/users/123/orders/456    → 204  (remove one order)
```

> **Golden rule:** the **URL is always a noun** (`/users/123`, not `/getUser/123`). The **HTTP verb is always the action** (`GET`, `POST`, `PUT`, `DELETE`). If you find yourself using verbs in URLs, you're fighting the protocol.

---

## Non-RESTful Anti-Patterns (Red Flags You'll See in Bad APIs)

Every junior team does these. Learn them so you can spot and fix them:

| Anti-pattern | Why it's bad |
|--------------|-------------|
| `POST /getUser?id=123` | Uses `POST` for reading — browser can't cache, crawler can't prefetch |
| `GET /deleteUser/123` | Uses `GET` for deletion — Google indexes your delete links |
| `POST /users` for everything | Ignores HTTP semantics — all requests look the same |
| `POST /users/123/update-email` | Verb in URL means the URL is the verb instead of the HTTP method |

> **The URL-is-a-noun rule is not aesthetic — it's functional.** Caches, firewalls, CDNs, and monitoring tools all rely on the HTTP method to know what kind of request this is. Using `POST` for reads means your CDN can never cache anything, and your monitoring tool can't distinguish reads from writes.

---

## Quick Self-Check (Interview-Style Questions)

1. What CRUD operation does each HTTP verb map to?
2. What's the difference between `PUT` and `PATCH`? When do you use each?
3. Why is `GET` safe and idempotent, but `POST` is neither?
4. A client sends a `POST /payments` and the network drops mid-response. Can the client safely retry? What should it do instead?
5. Why would you never use `GET /deleteProduct/456`?

<details>
<summary>Answers (try first, then peek)</summary>

1. `GET` = Read, `POST` = Create, `PUT` = Replace (full update), `PATCH` = Partial Update, `DELETE` = Delete.
2. `PUT` replaces the **entire** resource (you must send every field). `PATCH` updates **only the fields you send**. Use `PATCH` when you're changing one field; use `PUT` when you're rewriting the whole object.
3. `GET` doesn't change anything — reading is safe and always returns the same result (idempotent). `POST` creates a **new** resource each time — calling it twice creates two resources (not idempotent).
4. **No** — it might have been processed, so retrying blindly creates a duplicate payment. Use an **idempotency key** (a unique token in the request header) so the server can detect and reject the duplicate.
5. Because `GET` is supposed to be safe — browsers and crawlers follow `GET` links automatically. A link to `GET /deleteProduct/456` would trigger deletions in search indexes, prefetchers, and cached links.
</details>