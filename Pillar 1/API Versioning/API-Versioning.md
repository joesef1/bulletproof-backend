# 🔖 API Versioning: Evolve Without Breaking Anyone

> **One-sentence summary:** **API versioning** runs multiple generations of your API side-by-side (`/api/v1/`, `/api/v2/…`) so you can ship breaking changes for new clients while old clients keep working untouched.

---

## Why Versioning Exists (The Problem)

You launch `/users` returning `{"name": "Ana"}`. A thousand apps integrate. Six months later you need `{"first_name": "Ana", "last_name": "…", "email": …}`.

If you just *change* it:

| What you changed | Who breaks |
|---|---|
| Renamed `name` → `first_name` | Every old app crashes (`KeyError`) |
| Made `email` required | Old clients that never sent it get `422` |
| Removed an endpoint | Mobile apps **in the wild that can never be force-updated** die forever |
| Changed `200` shape on errors | Everyone's error handling misfires |

You can't call every developer and say "update by Friday." **Versioning is the contract**: v1 frozen in time, v2 free to improve, and a polite deprecation path between them.

---

## 🎭 The Classic Analogy

- **Your API** 🍽️ = a restaurant menu. Regulars (old clients) ordered "the usual" for years.
- **A breaking change** = silently replacing their dish with a new recipe. Outrage.
- **Versioning** = printing a **new menu (v2)** while still serving the old one (v1) to anyone who asks — plus a small note: *"v1 retires in December, try the new menu!"* Nobody goes hungry during the transition.

---

## 🖼️ When Does a Change Deserve a New Version? — See It

> This diagram from **ByteByteGo** explains **Semantic Versioning (SemVer)**: `MAJOR.MINOR.PATCH`. The rule that matters for APIs — **incompatible (breaking) changes bump MAJOR**, new-but-compatible features bump MINOR, fixes bump PATCH. A new API version (`v1` → `v2`) is exactly a MAJOR bump.

![What version numbers mean (SemVer) — ByteByteGo](https://assets.bytebytego.com/diagrams/0415-what-do-version-numbers-mean.png)

📎 Source: [`What do version numbers mean?` guide — ByteByteGo](https://github.com/ByteByteGoHq/system-design-101/blob/main/data/guides/what-do-version-numbers-mean.md)

**The API translation:** adding a field = MINOR (no new version needed ✅). Renaming/removing/requiring = MAJOR = ship `/api/v2/` ⚠️.

---

## ⚙️ The 3 Versioning Strategies (step by step)

### 1. URL versioning — `/api/v1/users` ✅ (most common, recommended for beginners)

```http
GET /api/v1/users/7
GET /api/v2/users/7
```

- **Pros:** visible, trivial to test in a browser/curl, cache-friendly (different URLs = different cache entries), obvious in logs.
- **Cons:** "unpure" REST (the URL names the API generation, not just the resource) — nobody cares in practice.
- **Verdict:** default choice. Stripe, GitHub, Twitter all do this.

### 2. Header versioning — `Accept: application/vnd.api+json;version=2`

```http
GET /users/7
Accept: application/vnd.myapp+json; version=2
```

- **Pros:** URLs stay "pure" (`/users` forever); version is content negotiation, REST-theoretically-clean.
- **Cons:** invisible — can't test in a browser bar, harder to debug, caches need `Vary: Accept` care, custom MIME types confuse tooling.
- **Verdict:** elegant, used by GitHub's API too — pick it when URL cleanliness matters more than debuggability.

### 3. Query-param versioning — `/users/7?version=2`

```http
GET /users/7?version=2
```

- **Pros:** as visible/testable as URL versioning, zero routing changes.
- **Cons:** params are for *filtering*, not identity — `?version=2` looks optional (it's not), messy logs, easy to forget.
- **Verdict:** fine for internal/quick APIs; avoid for public ones.

|  | URL | Header | Query param |
|---|---|---|---|
| Visible & debuggable | ✅ | ❌ | ✅ |
| Browser-testable | ✅ | ❌ | ✅ |
| Cache-safe by default | ✅ | ⚠️ needs `Vary` | ✅ |
| REST-pure | ❌ | ✅ | ❌ |
| Beginner-friendly | ✅✅ | ❌ | ✅ |

---

## 🧠 Backend Developer Depth — What Nobody Tells Beginners

### 1. Not every change needs a version — learn breaking vs safe 🟢🔴
**Safe (no new version):** adding an optional field, adding an endpoint, adding an optional query param, fixing a bug to match docs.
**Breaking (new version):** renaming/removing fields, making optional → required, changing types (`id`: int → string), removing endpoints, changing auth, changing error shapes.
Senior move: design v1 so most evolution is *additive* — version bumps become rare.

### 2. Deprecation is a process, not a switch 📢
Never delete v1 overnight. The professional sequence:
1. Ship v2, announce v1 deprecated (changelog + docs banner + `Sunset: <date>` response header — an actual standard, RFC 8594).
2. Optionally log/notify remaining v1 traffic ("you have 3 months").
3. After the sunset date, v1 → `410 Gone` (or redirect), not a mysterious 404.
Rushed sunsets break mobile apps stranded on old binaries.

### 3. DRF makes URL versioning nearly free 🐍
```python
# settings.py
REST_FRAMEWORK = {
    "DEFAULT_VERSIONING_CLASS": "rest_framework.versioning.URLPathVersioning",
    "DEFAULT_VERSION": "v1",
    "ALLOWED_VERSIONS": ["v1", "v2"],
}
# urls.py
path("api/<version>/users/", UserList.as_view())
```
Inside the view: `request.version` → branch logic (`if request.version == "v2": …`) or route to separate view/serializer classes per version. Separate serializers per version > `if` soup.

### 4. Version the representation, share the logic 🧩
v1 and v2 usually read the **same database and business logic** — only the *shape* differs (serializers/presenters). Duplicating whole viewsets per version rots fast; share services, fork only the schema layer.

### 5. Default version + docs per version 📚
- Unversioned requests (`/api/users` with no version) should resolve to a **documented default** (usually latest stable) — never crash.
- Docs must be version-switchable (Swagger UI per version, changelog with migration guide v1→v2). An unmigratable v2 is a failed v2.

### 6. How long to keep v1 alive? ⏳
As long as meaningful traffic uses it *and* you promised. Track version usage in metrics; sunset when v1 traffic ≈ noise *after* warnings. Public APIs: 6–12+ months. Internal: negotiate with your 3 frontend devs over coffee.

---

## 🛠️ Real Tools & Commands

- **Try all three strategies with curl:**
  ```bash
  curl -s localhost:8000/api/v1/users/7 | python3 -m json.tool        # URL
  curl -s localhost:8000/api/v2/users/7 | python3 -m json.tool        # URL
  curl -s -H "Accept: application/vnd.myapp+json; version=2" localhost:8000/users/7  # header
  curl -s "localhost:8000/users/7?version=2" | python3 -m json.tool   # query param
  ```
- **Check what version a request hit (DRF):** `request.version` in any view — log it to measure v1 vs v2 traffic before sunsetting.
- **Sunset header (set on deprecated versions):**
  ```http
  Sunset: Sat, 31 Dec 2026 23:59:59 GMT
  Deprecation: true
  Link: </api/v2/users/7>; rel="successor-version"
  ```

---

## 🔥 Cheat Sheet

| Change | New version? |
|---|---|
| Add optional field / endpoint | ❌ no (MINOR) |
| Rename / remove field | ✅ yes (MAJOR → v2) |
| Optional → required | ✅ yes |
| Change field type | ✅ yes |
| Bug fix matching docs | ❌ no (PATCH) |
| Kill v1 | Only after Sunset date → `410 Gone` |

---

## 🧲 How to Memorize

- **"Additive is free, destructive has a fee"** 💰 — additive changes ride along; destructive ones pay with a new version.
- **MAJOR = "may-jorly break you"** 💥 — incompatible changes bump MAJOR, which for APIs means `/v2/`.
- **Sunset = "sun sets on v1"** 🌅 — the `Sunset` header is literally the retirement date.
- One-liner: **Freeze v1, build v2, sunset with manners.**

---

## ✅ Quick Self-Check

<details>
<summary>1. Which of these needs a new API version: adding an optional `avatar` field, or renaming `name` to `first_name`?</summary>

Renaming needs v2 (breaking — old clients crash on the missing `name`). Adding an optional field is backward-compatible (MINOR), no new version.
</details>

<details>
<summary>2. Name the 3 versioning strategies and the beginner-recommended one.</summary>

URL (`/api/v1/`), header (`Accept: …;version=2`), query param (`?version=2`). URL versioning is recommended: visible, browser-testable, cache-safe.
</details>

<details>
<summary>3. What's wrong with deleting v1 the day v2 launches?</summary>

Old clients — especially mobile apps that can't be force-updated — break instantly. Deprecate properly: announce, set a `Sunset` date, monitor v1 traffic, then retire to `410 Gone`.
</details>

<details>
<summary>4. In DRF, how does a view know which version was requested?</summary>

With `URLPathVersioning`, `request.version` gives `"v1"`/`"v2"` — branch logic or (better) route to per-version serializer classes.
</details>

<details>
<summary>5. Should v1 and v2 duplicate business logic?</summary>

No — share the database and services; fork only the *representation* (serializers). Duplicated logic rots and diverges.
</details>