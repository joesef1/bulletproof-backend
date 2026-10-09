# Resource-Based URLs: Nouns in the Path, Verbs in the Method

## The One-Sentence Summary

**In REST, your URL identifies *what* you're talking about (a resource, the noun), and the HTTP method says *what you're doing to it* (the action, the verb) — so `/users/123/orders` with `GET` is the standard, and `/getOrdersForUser?id=123` is the anti-pattern you must unlearn.**

This single rule — **nouns in URLs, verbs in methods** — is what separates REST APIs from RPC-style endpoint soup. And it's not aesthetics: it's what makes APIs predictable, cacheable, and self-documenting.

---

## The Core Rule (And Why It Exists)

**RESTful URI design: a resource is a thing (noun), not an action (verb).**

| Conceptual part | What carries it | Example |
|-----------------|-----------------|---------|
| **The noun** (what resource) | The **URL path** | `POST` **`/users`** |
| **The verb** (what action) | The **HTTP method** | **`POST`** `/users` |

HTTP already comes with verbs built in (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`, …). If you also put verbs in the URL, you're telling the same story twice — and breaking all the conventions underneath (caching, monitoring, client predictions).

Here's the practical difference, side by side:

| Bad (verbs in URL — RPC style) | Good (nouns in URL — REST style) |
|--------------------------------|----------------------------------|
| `POST /getOrdersForUser?id=123` | `GET /users/123/orders` |
| `POST /createUser` | `POST /users` |
| `POST /deleteUser/123` | `DELETE /users/123` |
| `POST /updateUser/123` | `PATCH /users/123` |
| `GET /getOrders` | `GET /orders` |

> **Why does the "bad" version break things?**
> - It can't be **cached** — everything is a `POST`, and `POST` responses aren't cacheable.
> - It can't be **monitored** well — can't tell reads from writes.
> - It's **impossible to predict** — every endpoint is `POST /someNewVerb`.
> - Browsers, crawlers, and CDNs all look at the *method* to decide how to treat a request.

---

## The ByteByteGo REST API Cheat Sheet

This figure packs the whole RESTful API design story into one picture — including URL conventions, status codes, and best practices:

![REST API Cheat Sheet — diagram by ByteByteGo](https://assets.bytebytego.com/diagrams/0040-rest-api-cheatsheet.png)

*Diagram source: [ByteByteGo System Design 101 — REST API Cheat Sheet](https://github.com/ByteByteGoHq/system-design-101)*

---

## The 2-URL Rule: Every Resource Gets Exactly Two Base URLs

Here's the single most useful design heuristic (popularized by Google's Cloud API design): **every resource in your system gets exactly 2 base URLs** — one for the *collection*, one for a *specific element*.

| Base URL | What it represents | Example |
|----------|--------------------|---------|
| `/dogs` | The **collection** of all dogs | `GET /dogs` (list), `POST /dogs` (create) |
| `/dogs/1234` | **One specific** dog | `GET /dogs/1234`, `PUT /dogs/1234`, `PATCH /dogs/1234`, `DELETE /dogs/1234` |

Everything else is derived from these two. Notice: **no verbs anywhere** — the 4 HTTP verbs sit on top of the 2 nouns and give you all the operations you need.

---

## Building the URL: The 5 Convention Rules

### 1. Use **plural nouns** for collections — always

```
/orders          ✅   (a collection of orders)
/order           ❌   (inconsistent — single vs plural varies per endpoint)
```

Use `/orders/123` even when referring to ONE order. Plural consistently = one less decision to make.

### 2. Use **lowercase** and **hyphens** for multi-word names

```
/order-items     ✅
/orderItems      ❌  (camelCase in URLs = pain for case-sensitive systems)
/order_items     ❌  (underscores vs hyphens — pick hyphens, the common convention)
```

### 3. **No trailing slash**, **no file extension**

```
/users          ✅
/users/         ❌  (trailing slash is redundant)
/users.json     ❌  (format belongs in the Accept header, not the URL)
```

### 4. Nest to show **ownership**, but cap it at ~2 levels

When a child resource *belongs to* a parent, nest it:

```
/users/123/orders            ✅  "orders belonging to user 123"
/orders/456/line-items       ✅  "line items belonging to order 456"
```

But **don't nest deeper than 2-3 levels**:

```
/users/123/orders/456/items/77        ❌  too deep, hard to read
/companies/1/teams/2/members/3/tasks/4 ❌  a URL you never want to type
```

When you're tempted to go deep, **flatten** it and use a query parameter instead:

```
GET /line-items?order_id=456     ✅  flat + filter (same result, simpler)
GET /orders/456/line-items       ✅  also fine — nested form
```

**The rule of thumb:** if a resource has a *global identity* (it can be addressed on its own), promote it to top-level. Only keep nested URLs where the child truly cannot exist without the parent.

### 5. Use **query parameters for filtering / sorting / pagination** — not for actions

```
GET /orders?status=paid&page=2&sort=created_at      ✅
GET /getOrdersByStatus?status=paid                   ❌  verb → belongs in method
```

---

## The Action That Doesn't Fit: "Custom methods" (the one valid exception)

Sometimes you need an action that isn't a CRUD verb — "activate", "cancel", "reset-password", "publish". The REST-idiomatic way is **NOT** to carve a verb into the URL. Instead, model the action as a **sub-resource** you create or update:

```
POST /orders/456/cancellations        ✅  "create a cancellation for order 456"
POST /users/123/password-resets       ✅  "create a password reset"
POST /posts/89/publishes              ✅  or PATCH /posts/89 with {status: published}

POST /orders/456/cancel               ❌  verb in URL
GET  /users/123/resetPassword         ❌  verb + unsafe method
```

**The test:** can you phrase it as *"create a new sub-resource"* or *"set this field"*? If yes, you don't need a verb in the path.

---

## Naming Your Resources: The Collection → Item Hierarchy Tree

A good URL design is **predictable**. If a developer knows one endpoint, they should be able to guess the rest. That predictability comes from a consistent hierarchy:

```
/api/v1
 ├── /users                  (collection)
 │    ├── /users/{id}        (one user)
 │    ├── /users/{id}/orders (user's orders — nested collection)
 │    └── /users/{id}/avatar (user's profile image)
 ├── /orders
 │    ├── /orders/{id}
 │    └── /orders/{id}/items
 ├── /products
 │    ├── /products/{id}
 │    └── /products/{id}/reviews
 └── /categories
      └── /categories/{id}
```

> **Self-documentation in action:** knowing `GET /users/123` exists, a developer can *infer* `GET /users/123/orders` and `POST /users/123/orders` exist. That inference is the fingerprint of a well-designed API — no README needed.

---

## Common Anti-Patterns Cheat Sheet (Red Flags to Spot in Codebases)

| Anti-pattern | What it looks like | Why it's bad | Fix |
|--------------|--------------------|--------------|-----|
| **Verbs in URL** | `POST /createUser` | HTTP already has the verb | `POST /users` |
| **Get-verb for reads** | `GET /getOrders` | reads look like actions | `GET /orders` |
| **Action in URL** | `POST /deleteUser/5` | `DELETE` exists for this | `DELETE /users/5` |
| **Singular/plural mix** | `/user/1`, `/orders` | unpredictable pattern | always plural `/users/1` |
| **CamelCase** | `/orderItems` | case-sensitivity issues | `/order-items` |
| **Deep nesting** | `/a/1/b/2/c/3/d/4` | unreadable, brittle | flatten + query param |
| **Over-filtering** | `/orders?userId=123&getAll=true` | ownership buried in params | `/users/123/orders` |

---

## How to Memorize It

### The whole rule in one phrase
> **"Nouns in the URL, verbs in the method."**
> > "The URL picks the *thing*; the method picks the *action*."

### The Google 2-URL rule as a mantra
> **"Two URLs per resource: `/things` and `/things/{id}`. Get them right and everything else is derivable."**

### The "is it a verb?" test
> When designing any endpoint, ask: **"If I remove the HTTP method, can I still tell what resource this is?"**
> - `POST /users/123/cancellations` → yes, it's a cancellation. ✅
> - `POST /cancelOrder` → no, that's an action. ❌

### The hierarchy mnemonic — **"C-I-C-I-C"**
> **C**ollection → **I**tem → **C**hild-collection → **I**tem → …
>
> `/users` (C) → `/users/{id}` (I) → `/users/{id}/orders` (C) → `/orders/{id}` (I)
>
> Every endpoint is either a collection (list, create) or an item (read, update, delete). There's nothing else.

### The one-liner summary of *why*
> **"Making URLs nouns lets the whole web stack (browsers, caches, CDNs, monitoring) treat your API like the rest of the web — predictable, safe, cacheable, and boring in a good way."**

---

## Quick Self-Check (Interview-Style Questions)

1. Where do the "verb" and the "noun" belong in a REST request, and why?
2. Rewrite `POST /getOrdersForUser?id=123` in REST style. What changes and why?
3. What are the two "base URLs" per resource, and what operations does each support?
4. When should you nest a resource in the URL path vs. flatten it with a query param?
5. How would you design an endpoint for "cancel order 456" without putting a verb in the URL?

<details>
<summary>Answers (try first, then peek)</summary>

1. The **noun (resource)** goes in the URL path; the **verb (action)** goes in the HTTP method (`GET`/`POST`/`PUT`/`PATCH`/`DELETE`).
2. `GET /users/123/orders`. The `get` verb and the `ForUser` suffix disappear — the collection `/orders` nests under the user; reading is expressed by `GET`. Now it's cacheable, predictable, and infinitely extendable.
3. `/things` (the **collection**) supports `GET` (list) + `POST` (create). `/things/{id}` (the **item**) supports `GET`, `PUT`, `PATCH`, `DELETE`.
4. Nest when the child **cannot exist without the parent** (e.g., order line-items) and depth stays ≤ 2-3. Flatten with a query param (`?order_id=456`) when the resource has global identity or nesting gets too deep.
5. `POST /orders/456/cancellations` — model the action as a *sub-resource* (create a cancellation). This is the idiomatic REST replacement for action-verbs in URLs.
</details>