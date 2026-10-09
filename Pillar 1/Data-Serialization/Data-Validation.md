# 🛂 Data Validation: Trust No Input

> **One-sentence summary:** **Validation** enforces a strict **schema** on incoming data — parsing (`JSON-Parsing.md`) only checks *"is this JSON?"*, validation checks *"is this JSON correct, complete, and safe to use?"*

---

## Why Validation Exists (The Problem)

Parsing gets you a `dict`. It does **not** get you safety:

```python
data = json.loads('{"age": "banana", "admin": true}')
# Parses fine! ✅ (valid grammar)
# ...but "banana" as an age? And why is a signup form setting admin=true?! 🚨
```

Without validation, every handler becomes a minefield of `if` checks — and one forgotten check means:

| Unvalidated input | What happens |
|---|---|
| `{"age": "banana"}` | Crash deep in business logic (`500`) or corrupt DB row |
| `{"admin": true}` on signup | **Privilege escalation** — anyone makes themselves admin |
| `{"name": "<script>…"}` | Stored XSS payload served to every visitor |
| `{"price": -50}` | Negative-price order / money bug |
| Missing `"email"` | `KeyError` three functions later, impossible to debug |

**Rule #1 of backends: never trust the client.** The client is the wilderness — validate at the border, then your inner code can relax.

---

## 🎭 The Classic Analogy

- **Parsing** 📝 = checking the letter is written in English (grammar).
- **Validation** 🛂 = the **bouncer with a guest list**: right name? right age? on the list? No ID, no entry — turned away *at the door* with a clear reason (`422`), never let inside to cause trouble.

---

## 🖼️ Validation Is a Security Pillar — See It

> This cheatsheet from **ByteByteGo** lists the core strategies for secure APIs — and **"Validation of Inputs"** is one of them: *"defends against injection attacks and unexpected data format — validate headers, inputs, and payload."* Validation isn't just tidiness; it's defense.

![Cheatsheet to build secure APIs — ByteByteGo](https://assets.bytebytego.com/diagrams/0064-a-cheatsheet-to-build-secure-apis.png)

📎 Source: [`A Cheatsheet to Build Secure APIs` guide — ByteByteGo](https://github.com/ByteByteGoHq/system-design-101/blob/main/data/guides/a-cheatsheet-to-build-secure-apis.md)

---

## ⚙️ How Validation Works (step by step, with Pydantic)

**Pydantic** is the Python standard (it powers FastAPI). You declare a **model** = the guest list:

```python
from pydantic import BaseModel, EmailStr, Field

class Signup(BaseModel):
    name: str = Field(min_length=2, max_length=50)
    email: EmailStr
    age: int = Field(ge=13, le=120)   # 13 ≤ age ≤ 120
    admin: bool = False               # default, client can't sneak it in... see §4

# 1. Raw JSON arrives → parsed to dict
raw = {"name": "A", "email": "not-an-email", "age": 7}

# 2. Validate against the model
try:
    user = Signup(**raw)
except ValidationError as e:
    print(e.json())  # every problem listed, field by field
```

3. **Valid** → you get a typed object (`user.age` is a real `int`, guaranteed). Your handler code never checks types again.
4. **Invalid** → a `ValidationError` detailing *each* broken field → API returns **`422 Unprocessable Entity`** with that list (FastAPI does this automatically).

```json
// POST /signup  {"name": "A", "email": "not-an-email", "age": 7}  →  422
{
  "detail": [
    {"loc": ["name"], "msg": "String should have at least 2 characters"},
    {"loc": ["email"], "msg": "value is not a valid email address"},
    {"loc": ["age"], "msg": "Input should be greater than or equal to 13"}
  ]
}
```

---

## 🧠 Backend Developer Depth — What Nobody Tells Beginners

### 1. Parsing (400) vs Validation (422) — different layers, different codes 🔢
- **400 Bad Request** = "I can't even *read* this" (malformed JSON, wrong content-type).
- **422 Unprocessable Entity** = "I read it perfectly, but it's *wrong*" (bad email, age 7, missing field).
FastAPI returns these automatically at the two different layers. Knowing which is which makes debugging 10× faster — for you *and* your frontend dev.

### 2. Validate ONCE at the boundary, never scatter `if`s 🧹
The validated model should be the **first thing** your endpoint touches; everything downstream receives the clean object. If you find yourself writing `if not isinstance(age, int)` inside business logic, the validation layer has a hole. One gate, well-guarded.

### 3. Coercion vs strict mode — know which you're getting 🔄
By default Pydantic **coerces**: `"7"` → `7`, `"true"` → `True`. Convenient (forms/query strings are all text!) but can hide bugs. `StrictInt` / `model_config = ConfigDict(strict=True)` disables it. Rule of thumb: coercion ON for query params/forms, consider strict for money/IDs.

### 4. Mass assignment: the `admin=true` attack 🎭
If your model includes sensitive fields, clients can set them. Defenses:
- **Separate input vs internal models** — signup schema has no `admin` field at all.
- `extra="forbid"` — reject unknown fields instead of silently ignoring them.
- Mark server-owned fields read-only / exclude them from input schemas (DRF: `read_only_fields`).
Real breaches (GitHub 2012, etc.) came from exactly this.

### 5. Validate on the way OUT too 📤
Response models strip secrets (`password_hash` never leaves the server) and guarantee shape for frontends. In Pydantic/FastAPI: `response_model=UserPublic`. Your DB row ≠ your API contract.

### 6. DB constraints are the LAST line, not the first 🧱
`NOT NULL`, `UNIQUE`, `CHECK` in the database are essential — but a DB error is a 500 with a cryptic message. Validation gives users a **422 with field-level messages** *before* anything touches storage. Both, in that order.

### 7. Custom & cross-field rules 🧩
Built-ins cover types/ranges; real logic needs custom validators — password strength, `end_date > start_date`, "coupon exists and isn't expired". Pydantic `@field_validator` / `@model_validator`, DRF `validate_<field>` / `validate()`. Keep DB-hitting checks (does this username exist?) in validators too — but mind the query cost.

---

## 🛠️ Real Tools & Commands

- **Python:** **Pydantic v2** (FastAPI's engine), **DRF Serializers** (Django world), **Marshmallow**.
- **TypeScript/frontend mirror:** **Zod** — same idea, infers TS types from the schema.
- **Trigger a 422 yourself and read it:**
  ```bash
  curl -s -X POST http://localhost:8000/signup \
    -H "Content-Type: application/json" \
    -d '{"name":"A","email":"nope","age":7}' | python3 -m json.tool
  ```
- **Quick Pydantic check in a REPL:**
  ```python
  from pydantic import BaseModel, ValidationError
  class M(BaseModel):
      age: int
  try:
      M(age="banana")
  except ValidationError as e:
      print(e.errors())  # [{'loc': ('age',), 'msg': 'Input should be a valid integer', ...}]
  ```
- **JSON Schema** fans: the `jsonschema` package validates raw dicts against a `schema.json` — language-agnostic contracts you can share with any client.

---

## 🔥 Cheat Sheet

| Layer | Question answered | Failure code | Tool |
|---|---|---|---|
| Parse | Is this JSON? | `400` | `json.loads` |
| Validate input | Is this JSON *right*? | `422` | Pydantic / Serializer / Zod |
| Authorize | May *you* do this? | `401` / `403` | Auth (Pillar 3 🔜) |
| DB constraints | Last-resort integrity | `500` (avoid!) | `NOT NULL` / `UNIQUE` / `CHECK` |

---

## 🧲 How to Memorize

- **"Parse the letters, check the guest list"** 📝🛂 — parsing = language, validation = permission.
- **Bouncer's three questions**: *Present?* (required) → *Correct type?* (int, email…) → *In range?* (13–120). Every schema field answers all three.
- **422 = "four-twenty-NO"** 🚫 — I understood you, and the answer is no.
- One-liner: **Never trust the client; validate at the border.**

---

## ✅ Quick Self-Check

<details>
<summary>1. Parsing passed but the data is still dangerous. Give one example.</summary>

`{"admin": true}` on a signup form parses perfectly (valid JSON grammar) but escalates privileges — parsing checks syntax, not meaning. Only schema validation catches it.
</details>

<details>
<summary>2. When do you return 400 vs 422?</summary>

**400** = unreadable (malformed JSON, wrong content-type). **422** = readable but wrong (bad email, age out of range, missing required field).
</details>

<details>
<summary>3. What is a mass-assignment attack and how do you prevent it?</summary>

Client sets server-owned fields (e.g. `admin: true`) because the schema accepts them. Prevent with separate input models (no sensitive fields), `extra="forbid"`, and read-only field declarations.
</details>

<details>
<summary>4. Why validate response (outgoing) data too?</summary>

To strip secrets (`password_hash`) and guarantee a stable contract for frontends — your DB row shape should never leak directly to clients.
</details>

<details>
<summary>5. If the DB has NOT NULL / CHECK constraints, why bother validating in Python?</summary>

DB errors surface as 500s with cryptic messages. App-level validation fails fast with clear per-field 422 messages — better UX, cheaper, and keeps business logic clean. Use both, validate first.
</details>