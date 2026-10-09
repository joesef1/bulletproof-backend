# ⚛️ ACID Properties: Promises Your Database Keeps

> **One-sentence summary:** **ACID** is the four-part guarantee behind every database **transaction** — **Atomicity** (all-or-nothing), **Consistency** (rules never broken), **Isolation** (concurrent transactions don't see each other's half-work), **Durability** (committed = saved, even if the power dies).

---

## Why ACID Exists (The Problem)

Transfer $100 from Ana → Bob. Behind the scenes that's **two** writes:

```sql
UPDATE accounts SET balance = balance - 100 WHERE user = 'ana';  -- step 1
UPDATE accounts SET balance = balance + 100 WHERE user = 'bob';  -- step 2
```

Now imagine the server **crashes between step 1 and step 2**. $100 vanishes into thin air. Or two transfers run **at the same time** and overwrite each other. Or the power dies a millisecond after "success" — was it saved or not?

Money can't tolerate "probably." ACID exists so grouped operations behave **as one trustworthy unit**:

| Nightmare | ACID promise that kills it |
|---|---|
| Crash halfway through multi-step write | **Atomicity** — all steps, or zero (rollback) |
| Write leaves impossible data (negative balance, orphan order) | **Consistency** — rules/constraints always hold |
| Two users act simultaneously, results interleave wrongly | **Isolation** — concurrent work stays separated |
| "Success!" then power outage loses it | **Durability** — committed means persisted |

---

## 🎭 The Classic Analogy

- A transaction = **a wedding ceremony** 💒 — vows, rings, kiss, paperwork.
- **Atomicity** = all-or-nothing: you can't be "half married." Ceremony interrupted → as if nothing happened.
- **Consistency** = the rules held: both adults, licensed officiant, signed papers — no impossible states.
- **Isolation** = two weddings in neighboring halls don't mix up their vows or rings.
- **Durability** = once pronounced married (committed 📜), the certificate survives even if the venue burns down.

---

## 🖼️ ACID in One Picture — See It

> This diagram from **ByteByteGo** walks through all four properties with concrete transaction examples.

![What ACID means — ByteByteGo](https://assets.bytebytego.com/diagrams/0407-what-does-acid-mean.png)

📎 Source: [`What does ACID mean?` guide — ByteByteGo](https://github.com/ByteByteGoHq/system-design-101/blob/main/data/guides/what-does-acid-mean.md)

---

## ⚙️ The Four Properties (step by step)

Wrap steps in a transaction — either **everything commits**, or **everything rolls back**:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE user = 'ana';
UPDATE accounts SET balance = balance + 100 WHERE user = 'bob';
COMMIT;   -- success: both visible, forever. Any failure before this → ROLLBACK: neither visible.
```

### A — Atomicity: "all or nothing" ⚛️
The transaction is **indivisible**. Crash, error, or constraint violation mid-way → the DB undoes every partial write (rollback). Your code never sees half-transfers. In Django: `with transaction.atomic():` — an exception inside rolls everything back.

### C — Consistency: "rules never break" 📏
Every transaction moves the DB from **one valid state to another**. Constraints (`CHECK balance >= 0`, foreign keys, `UNIQUE`, `NOT NULL`) are the enforcers — a transaction that would violate them is rejected entirely. Note: this is *not* CAP-theorem consistency (that's about replicas agreeing); it's about **invariants holding**.

### I — Isolation: "mind your own business" 🧊
Concurrent transactions behave **as if running alone** — no seeing each other's uncommitted drafts. Strictness is a dial (isolation *levels*, weakest → strongest):
`READ UNCOMMITTED` → `READ COMMITTED` → `REPEATABLE READ` → `SERIALIZABLE`.
Stronger = safer but slower (more locking). Postgres defaults to `READ COMMITTED` — good enough for most apps; reach for stronger only when money/counters demand it. (Full level-by-level guide 🔜 in a later file.)

### D — Durability: "committed means carved in stone" 🗿
Once `COMMIT` returns success, the data **survives crashes** — via write-ahead logs flushed to disk (and replication to other nodes in distributed setups). If the server dies a millisecond after your commit ack, the data is still there on reboot.

---

## 🧠 Backend Developer Depth — What Nobody Tells Beginners

### 1. `transaction.atomic()` is your money-transfer seatbelt 🐍
```python
from django.db import transaction

with transaction.atomic():          # BEGIN
    ana.balance -= 100; ana.save()
    bob.balance += 100; bob.save()  # exception here? → ROLLBACK, ana untouched
# exiting cleanly → COMMIT
```
Without it, each `.save()` auto-commits separately — a crash between them corrupts reality. **Any multi-write operation that must succeed-or-fail-together belongs in one transaction.**

### 2. Atomicity ≠ "one query" — and single queries are already atomic ⚡
One `UPDATE` touching 10,000 rows either fully applies or fully doesn't — the DB guarantees that without you asking. Transactions extend the guarantee *across* statements.

### 3. Isolation levels are a performance/correctness tradeoff 🎚️
`SERIALIZABLE` (strictest) can **abort your transaction** under contention (`could not serialize access`) — your code must **retry**. Weaker levels permit anomalies (dirty reads, lost updates). Default `READ COMMITTED` + application-level care (e.g., `SELECT … FOR UPDATE` on hot rows) beats blindly maxing isolation.

### 4. The lost-update trap (why isolation matters concretely) 💸
Two requests read `balance = 500` simultaneously, both subtract 100, both write 400. Correct answer: 300. Without proper isolation/locking, you just gave away $100. Fixes: row locks (`select_for_update()`), atomic DB increments (`F()` expressions / `UPDATE … SET balance = balance - 100`), or optimistic locking.

### 5. Durability has a speed cost — and knobs 💾
True durability = flushing to disk before acking (`synchronous_commit`, WAL fsync). Disabling it is faster but can lose seconds of "committed" data on crash. Defaults are safe; only tune when benchmarks prove you must — and understand exactly what you'd lose.

### 6. NoSQL often trades ACID away (knowingly) 🔀
MongoDB/Redis/Cassandra historically offered weaker guarantees (single-document atomicity, eventual consistency) for speed/scale — that's the CAP-theorem tradeoff (NoSQL files 🔜). Choose ACID SQL when correctness beats raw throughput: payments, inventory, bookings.

---

## 🛠️ Real Tools & Commands

- **Raw SQL transaction (psql):**
  ```sql
  BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE user = 'ana';
  -- oops, check first:
  SELECT balance FROM accounts WHERE user = 'ana';  -- still old value until COMMIT!
  COMMIT;   -- or ROLLBACK;
  ```
- **Django shell demo of rollback:**
  ```python
  from django.db import transaction
  try:
      with transaction.atomic():
          ana.balance -= 100; ana.save()
          raise ValueError("boom")   # simulated crash
  except ValueError: pass
  ana.refresh_from_db()  # balance unchanged ✅
  ```
- **See your isolation level (Postgres):**
  ```sql
  SHOW default_transaction_isolation;  -- typically 'read committed'
  ```
- **Lock a hot row while you work (Django):**
  ```python
  with transaction.atomic():
      acc = Account.objects.select_for_update().get(user="ana")  -- row locked
      acc.balance -= 100; acc.save()
  ```

---

## 🔥 Cheat Sheet

| Letter | Promise | Violation looks like |
|---|---|---|
| **A**tomicity | All-or-nothing | Half-transfer after crash |
| **C**onsistency | Rules always hold | Negative balance, orphan rows |
| **I**solation | Concurrent = as-if-alone | Lost updates, dirty reads |
| **D**urability | Committed = persisted | "Success" lost on reboot |

---

## 🧲 How to Memorize

- **"ACID test"** 🧪 — like the chemistry acid test for gold: only *real* databases pass all four.
- **Alphabet story**: **A**ll-or-nothing → **C**hecks hold → **I**solated workers → **D**isk remembers.
- **Wedding test** 💒 — interrupted ceremony = never married (atomicity); valid couple (consistency); neighboring weddings don't mix (isolation); certificate outlives the venue (durability).
- One-liner: **Begin together, commit together, survive everything.**

---

## ✅ Quick Self-Check

<details>
<summary>1. The server crashes between debiting Ana and crediting Bob. Which property saves the $100, and how?</summary>

**Atomicity** — the multi-statement transfer is one indivisible unit; anything short of `COMMIT` triggers rollback, so neither write survives. Money can't be half-moved.
</details>

<details>
<summary>2. What's the difference between ACID-consistency and CAP-consistency?</summary>

ACID consistency = database **rules/invariants always hold** (no negative balances). CAP consistency = every **read sees the latest write** across replicas. Different words, same spelling — classic interview trap.
</details>

<details>
<summary>3. Two requests read balance=500, subtract 100 each, write 400. Correct balance is 300. What went wrong?</summary>

A **lost update** — an isolation failure. Both transactions read the same starting value and overwrote each other. Fix with row locks (`select_for_update()`), atomic `UPDATE … SET balance = balance - 100`, or stronger isolation.
</details>

<details>
<summary>4. Why does Django's `transaction.atomic()` matter for multi-write operations?</summary>

Without it each `.save()` auto-commits separately, so a crash mid-sequence leaves partial writes. `atomic()` wraps them in one transaction: exception inside → full rollback.
</details>

<details>
<summary>5. When would you choose a NoSQL store over an ACID SQL database?</summary>

When raw throughput/scale/flexibility beats strict correctness (caches, analytics, feeds) — accepting weaker guarantees per the CAP tradeoff. Money, inventory, bookings stay on ACID SQL.
</details>