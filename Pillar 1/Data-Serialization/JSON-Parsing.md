# 📦 JSON Parsing: Turning Objects Into Text (and Back)

> **One-sentence summary:** **Serialization** converts your language's native objects (a Python `dict`, a JS object) into a JSON **string** for transport over HTTP; **deserialization** (parsing) converts that string back into objects on the other side.

---

## Why Serialization Exists (The Problem)

Networks only move **bytes** — they have no idea what a Python `dict` or a JavaScript object is. Try sending a live object over HTTP and… you can't. Memory addresses mean nothing on another machine (possibly running another language entirely).

So both sides agree on a **text format** that every language can read and write:

| Problem | JSON's answer |
|---|---|
| Python `dict` ≠ JavaScript object ≠ Go struct | One shared text format all languages parse |
| HTTP bodies are byte streams | JSON is just a UTF-8 string — fits perfectly |
| Need human-readable debugging | You can *read* JSON with your eyes (unlike Protobuf bytes) |
| Schema must stay flexible | No pre-registration needed — just send it |

**Serialization** = object → string (packing the suitcase 🧳).
**Deserialization / parsing** = string → object (unpacking it).

---

## 🎭 The Classic Analogy

- **Serialization** 📝 = writing a letter describing your house ("2 bedrooms, red door…"). The house itself can't travel — the *description* can.
- **Deserialization** 🏠 = the receiver reads the letter and rebuilds the idea of the house in their head.
- As long as both sides agree on the *language of the letter* (JSON grammar), a Python house becomes a JavaScript house with zero loss.

---

## 🖼️ What Valid JSON Looks Like — The Official Grammar

> This is the **official JSON syntax diagram** from [json.org](https://www.json.org/json-en.html) (by Douglas Crockford, who defined JSON). It shows exactly what a JSON **object** may contain: `{` then string-value pairs separated by commas, then `}`. Follow the railroad tracks — any path you can trace is valid JSON.

![JSON object syntax diagram — json.org](https://www.json.org/img/object.png)

📎 Source: [Introducing JSON — json.org](https://www.json.org/json-en.html)

---

## ⚙️ How It Works (step by step)

### Sending side — serialize

```python
import json

user = {"id": 7, "name": "Ana", "admin": False, "tags": ["dev", "ml"]}
text = json.dumps(user)   # dict → string
# '{"id": 7, "name": "Ana", "admin": false, "tags": ["dev", "ml"]}'
```

1. Your code builds a native object (`dict` / object / struct).
2. `json.dumps` / `JSON.stringify` walks it and emits a UTF-8 **string**.
3. That string goes into the HTTP body with `Content-Type: application/json`.

### Receiving side — parse

```python
data = json.loads(request.body)  # string → dict
print(data["name"])  # "Ana"
```

1. Server reads the raw body **bytes** → decodes to string.
2. `json.loads` / `JSON.parse` validates the grammar (like the railroad diagram above) and rebuilds native objects.
3. Invalid JSON → parse error → your API should answer `400 Bad Request`.

### The only 6 data types JSON allows

| JSON | Python | JavaScript | Notes |
|---|---|---|---|
| `string` | `str` | `string` | Always double quotes — `'hi'` is **invalid** JSON |
| `number` | `int` / `float` | `number` | No distinction int vs float; no `NaN`/`Infinity` |
| `true` / `false` | `True` / `False` | `true` / `false` | Lowercase in JSON! |
| `null` | `None` | `null` | Not `"null"`, not `undefined` |
| `object` `{…}` | `dict` | object | Keys **must** be strings |
| `array` `[…]` | `list` / `tuple` | `Array` | Mixed types allowed |

---

## 🧠 Backend Developer Depth — What Nobody Tells Beginners

### 1. `datetime`, `Decimal`, `UUID` are NOT JSON-native 📅
This bites every beginner:
```python
json.dumps({"at": datetime.now()})  # 💥 TypeError: not JSON serializable
```
Fixes: convert to ISO string first (`obj.isoformat()`), use `str()`, or pass a custom `default=` handler. Same story in every language (Go's `time.Time` marshals to RFC3339 automatically — Python doesn't).

### 2. Numbers are lossy across languages 🔢
JSON has one `number` type. A 64-bit ID like `9007199254740993` survives Python fine but **loses precision in JavaScript** (all JS numbers are float64). Rule: send big IDs/timestamps as **strings** if a JS frontend will read them.

### 3. Parse errors are a client error — return 400, not 500 ⚠️
If the body isn't valid JSON, that's the *client's* fault. Catch it explicitly:
```python
try:
    data = json.loads(request.body)
except json.JSONDecodeError:
    return JsonResponse({"error": "malformed JSON"}, status=400)
```
Uncaught → 500 → your error tracker screams about someone else's typo.

### 4. Never trust parsed JSON — validate it (next file 🔜)
Parsing only checks *grammar* ("is this JSON?"). It says nothing about *meaning* ("is `age` a positive int?"). That's **Data Validation** — our next topic (Pydantic / DRF serializers / Zod).

### 5. `Content-Type: application/json` is a contract 📜
Always set it when sending JSON, always check it when receiving. Without it, frameworks won't parse the body (Express needs `express.json()`; Django reads `request.body` raw). A missing content-type is the #1 "my API gets empty data" bug.

### 6. JSON is schemaless — that's a feature AND a footgun 🔫
No pre-registration (unlike Protobuf/gRPC) = ship fast. But rename `userName` → `username` and every client breaks silently. Mitigations: versioning (`/api/v1/`), contract tests, and validation schemas.

### 7. Performance footnote ⚡
`json.dumps/loads` is fast enough for 99% of APIs. If you ever profile and JSON is the bottleneck (huge payloads), drop-in speedups exist (`orjson` in Python is ~5–10× faster) — but don't optimize this before you need to.

---

## 🛠️ Real Tools & Commands

- **Pretty-print any JSON in the terminal:**
  ```bash
  curl -s https://api.github.com/users/octocat | python3 -m json.tool
  ```
- **Validate JSON from a file:**
  ```bash
  python3 -c "import json,sys; json.load(open('data.json')); print('valid ✅')"
  ```
- **Explore nested JSON visually:** [JSON Crack](https://jsoncrack.com) turns blobs into graph diagrams (via ByteByteGo's [`json-files` guide](https://github.com/ByteByteGoHq/system-design-101/blob/main/data/guides/json-files.md)).
- **JS side — the two functions you need:**
  ```js
  const text = JSON.stringify({ a: 1 }); // object → string
  const obj = JSON.parse(text);           // string → object (throws on bad JSON!)
  ```
- **Python side:**
  ```python
  json.dumps(obj)          # object → string
  json.loads(text)         # string → object (raises json.JSONDecodeError)
  json.load(file)          # file → object   |  json.dump(obj, file)
  ```

---

## 🔥 Cheat Sheet

| Direction | Python | JavaScript | Name |
|---|---|---|---|
| Object → string | `json.dumps()` | `JSON.stringify()` | **Serialize** (dump **s** = **s**tring) |
| String → object | `json.loads()` | `JSON.parse()` | **Deserialize / parse** |
| File → object | `json.load()` | — | no-**s** = file/stream |

---

## 🧲 How to Memorize

- **"dump the suitcase shut 🧳, load it open"** — `dumps` packs object→string, `loads` unpacks string→object. The **`s`** = **string**.
- **"JSON is JavaScript's handwriting"** — object literals with double quotes. If JS would write it (minus functions/`undefined`), it's probably valid JSON.
- **Single quotes are a lie** ❌ — `'name'` looks right, fails parsing. JSON demands `"`.
- One-liner: **Serialize to send, parse to use.**

---

## ✅ Quick Self-Check

<details>
<summary>1. What's the difference between serialization and deserialization?</summary>

Serialization = native object → JSON string (for sending). Deserialization/parsing = JSON string → native object (for using). `dumps`/`stringify` vs `loads`/`parse`.
</details>

<details>
<summary>2. Is `{'name': 'Ana'}` valid JSON? Why / why not?</summary>

No — JSON requires **double quotes** for strings and keys: `{"name": "Ana"}`. Single quotes are a `JSONDecodeError`.
</details>

<details>
<summary>3. `json.dumps({"at": datetime.now()})` crashes. Why, and what's the fix?</summary>

`datetime` isn't one of JSON's 6 native types. Convert first: `{"at": dt.isoformat()}` — or pass a custom `default=str` handler.
</details>

<details>
<summary>4. A client sends a body that isn't valid JSON. What status code should you return?</summary>

**400 Bad Request** — malformed input is the client's fault, not a server crash (500).
</details>

<details>
<summary>5. Why send large IDs (e.g. 64-bit) as strings instead of JSON numbers?</summary>

JavaScript numbers are float64 — integers above 2⁵³ lose precision when a JS frontend parses them. Strings preserve exact digits.
</details>