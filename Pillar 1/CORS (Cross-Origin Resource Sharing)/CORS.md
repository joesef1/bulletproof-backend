# 🌐 CORS: The Browser's Bouncer

> **One-sentence summary:** **CORS (Cross-Origin Resource Sharing)** is a *browser* security rule that blocks frontend JavaScript from reading responses from a different **origin** (scheme + host + port) — unless the backend explicitly permits it with `Access-Control-Allow-*` headers.

---

## Why CORS Exists (The Problem)

You're building Django (backend `localhost:8000`) + React (frontend `localhost:3000`). The page loads, JS calls your API… and the browser kills it:

```
Access to fetch at 'http://localhost:8000/api/users/'
from origin 'http://localhost:3000' has been blocked by CORS policy
```

Every full-stack developer meets this error. It's not a bug — it's the **Same-Origin Policy**: by default, a page may only *read* responses from its **own origin**. Without it, any evil site you visit could silently read your bank's API responses using *your* logged-in cookies. 😱

CORS is the **controlled exception**: the server can say *"I trust that frontend — let it in."*

**What counts as "different origin"?** Scheme + host + port must ALL match:

| Frontend | Backend | Same origin? |
|---|---|---|
| `http://localhost:3000` | `http://localhost:8000` | ❌ different port |
| `https://app.com` | `https://api.app.com` | ❌ different host |
| `http://app.com` | `https://app.com` | ❌ different scheme |
| `https://app.com` | `https://app.com/api` | ✅ same origin (path doesn't matter) |

---

## 🎭 The Classic Analogy

- **Same-Origin Policy** 🏰 = a castle rule: *"servants only take orders from inside the castle."*
- **CORS** 🛂 = the **bouncer with a guest list**: the backend hands the browser a list of trusted origins. Request from `localhost:3000`? On the list → enter. Random evil site? Turned away — even though the *server* answered fine, the *browser* refuses to hand the response to the page.

Key twist (see §1 below): the bouncer works for the **browser**, not the server.

---

## 🖼️ The Two CORS Flows — See It

> Diagrams from **MDN Web Docs**. First: a **simple request** — the browser just adds an `Origin` header and checks the response. Second: a **preflighted request** — for "non-simple" calls the browser first sends an `OPTIONS` scout, and only sends the real request if the server approves.

![Simple CORS request — MDN](https://mdn.github.io/shared-assets/images/diagrams/http/cors/simple-request.svg)

![Preflighted CORS request — MDN](https://mdn.github.io/shared-assets/images/diagrams/http/cors/preflight-correct.svg)

📎 Source: [Cross-Origin Resource Sharing (CORS) — MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS)

---

## ⚙️ How CORS Works (step by step)

### Case A — Simple request (`GET`/`POST` with plain headers, e.g. a basic fetch)

1. Browser sends the request **normally**, adding `Origin: http://localhost:3000`.
2. Server responds **normally**, optionally adding:
   ```http
   Access-Control-Allow-Origin: http://localhost:3000
   ```
3. Browser checks: response header matches the page's origin? ✅ → JS gets the data. Missing/mismatch? ❌ → JS gets a CORS error (the response *arrived*, the browser just hides it).

### Case B — Preflighted request (anything "fancy": `PUT`/`DELETE`, `Authorization` header, `Content-Type: application/json`…)

1. Browser first sends a **scout** — an `OPTIONS` request with no body:
   ```http
   OPTIONS /api/users HTTP/1.1
   Origin: http://localhost:3000
   Access-Control-Request-Method: PUT
   Access-Control-Request-Headers: content-type, authorization
   ```
   Translation: *"Would you allow a PUT with those headers from me?"*
2. Server answers (no body needed, `204` is typical):
   ```http
   HTTP/1.1 204 No Content
   Access-Control-Allow-Origin: http://localhost:3000
   Access-Control-Allow-Methods: GET, POST, PUT, DELETE
   Access-Control-Allow-Headers: content-type, authorization
   Access-Control-Max-Age: 86400
   ```
3. Browser satisfied? → sends the **real** `PUT`. Otherwise → blocks it before it ever leaves.

---

## 🧠 Backend Developer Depth — What Nobody Tells Beginners

### 1. CORS is enforced by the BROWSER, not your server 🌍
Your API happily answers everyone — `curl` and Postman **never** hit CORS errors (they're not browsers, no Origin enforcement). CORS headers are just *permission slips the browser reads*. Debugging rule: if `curl` works but the browser fails → it's CORS, not your endpoint.

```bash
# Prove it: same request curl (works) vs browser (blocked without ACAO header)
curl -i -H "Origin: http://localhost:3000" http://localhost:8000/api/users/
```

### 2. `Access-Control-Allow-Origin: *` + credentials = forbidden combo 🔐
Cookies/`Authorization` (fetch `credentials: "include"`) require:
- `Access-Control-Allow-Credentials: true`, AND
- an **explicit** origin (`https://app.com`) — `*` is **rejected** by browsers.
Never reflect `Origin` blindly (`Allow-Origin: <whatever was sent>`) — that's disabling the bouncer while pretending he's on duty. Whitelist, don't echo.

### 3. Preflights cost a round-trip — cache them ⚡
`Access-Control-Max-Age: 86400` tells the browser to remember the scout's answer for 24h instead of preflighting every call. Cheap, big win for chatty frontends.

### 4. The Django setup (you'll do this on every project) 🐍
```bash
pip install django-cors-headers
```
```python
# settings.py
INSTALLED_APPS = [..., "corsheaders", ...]
MIDDLEWARE = ["corsheaders.middleware.CorsMiddleware", ...]  # as high as possible
CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "https://app.com",
]
# If the frontend sends cookies:
CORS_ALLOW_CREDENTIALS = True
```
Put the middleware **above** `CommonMiddleware` — preflight `OPTIONS` must be answered before anything else touches it.

### 5. The `Vary: Origin` gotcha 🧊
If your server returns different `Allow-Origin` per requester, add `Vary: Origin` so caches don't serve `app.com`'s permission slip to `evil.com`. (`django-cors-headers` handles this automatically.)

### 6. Reading the error message properly 🔍
- *"No 'Access-Control-Allow-Origin' header"* → simple request, server didn't allow you. Fix server config.
- *"Response to preflight… doesn't pass"* → `OPTIONS` failed (often 404/405 — your server doesn't handle `OPTIONS`, or middleware order is wrong).
- *"…credentials mode is 'include'… wildcard…"* → the `*` + credentials combo (§2).
Open DevTools → Network → find the red `OPTIONS` request → read its response headers. The answer is always there.

### 7. CORS ≠ security for your API 🛡️
CORS only protects *browser users* from *other sites*. Attackers with `curl`, scripts, or mobile apps bypass it entirely. Real API security = auth, validation, rate limiting (Pillars 3–4 🔜). CORS is a seatbelt, not a vault.

---

## 🛠️ Real Tools & Commands

- **Simulate a preflight by hand:**
  ```bash
  curl -i -X OPTIONS http://localhost:8000/api/users/ \
    -H "Origin: http://localhost:3000" \
    -H "Access-Control-Request-Method: PUT" \
    -H "Access-Control-Request-Headers: content-type, authorization"
  # Look for Access-Control-Allow-Origin in the reply. Absent? That's your bug.
  ```
- **Chrome DevTools:** Network tab → the failed request turns red → *Headers* sub-tab shows exactly which `Access-Control-*` headers came back (or didn't).
- **Quick local unblock (dev only!):** browser extensions that strip CORS checks exist — fine for debugging, never a "fix". The fix is always server headers.
- **Express equivalent** (for reference): `app.use(require("cors")({ origin: "http://localhost:3000" }))`.

---

## 🔥 Cheat Sheet

| Situation | Fix |
|---|---|
| `GET` blocked, no ACAO header | Add frontend origin to allow-list |
| `PUT`/`DELETE`/JSON blocked | Handle `OPTIONS` preflight (middleware) |
| Cookies/Auth + `*` | Explicit origin + `Allow-Credentials: true` |
| Works in curl, fails in browser | It's CORS (not your endpoint) — check response headers |
| Different header per origin | Add `Vary: Origin` |
| Preflight on every request | Set `Access-Control-Max-Age` |

---

## 🧲 How to Memorize

- **"CORS = Can Other Resources See?"** 👀 — the browser asks the server exactly that, per origin.
- **Three letters, three parts**: **O**rigin = **s**cheme + **h**ost + **p**ort. Change any one → new origin.
- **OPTIONS = "opt-in question"** ❓ — the scout asks permission before the real request.
- One-liner: **Server writes the guest list; browser works the door.**

---

## ✅ Quick Self-Check

<details>
<summary>1. Who enforces CORS — the server or the browser?</summary>

The **browser**. The server merely sends `Access-Control-Allow-*` permission headers. That's why `curl`/Postman never see CORS errors.
</details>

<details>
<summary>2. When does the browser send a preflight OPTIONS request?</summary>

For "non-simple" requests: methods other than GET/HEAD/POST, or custom headers (`Authorization`), or `Content-Type: application/json`. The OPTIONS scout asks permission before the real request is sent.
</details>

<details>
<summary>3. Why does `Allow-Origin: *` fail when your fetch uses credentials?</summary>

Browsers reject the wildcard with `credentials: "include"` — cookies require an explicit origin plus `Access-Control-Allow-Credentials: true`. Otherwise any site could ride the user's session.
</details>

<details>
<summary>4. `http://localhost:3000` → `http://localhost:8000`: same origin?</summary>

No — same scheme and host, but **different ports** (3000 vs 8000) = different origins. Path never matters; port always does.
</details>

<details>
<summary>5. Your endpoint works in curl but the browser says "blocked by CORS policy". What's the very first thing to check?</summary>

The **response headers** of the failed request (DevTools → Network): is `Access-Control-Allow-Origin` present and matching your frontend's origin? If missing → server allow-list/middleware config.
</details>