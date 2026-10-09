# 💾 Transactions: Grouping Operations for Data Integrity

> **One-sentence summary:** **Database transactions** group multiple SQL operations into a single atomic unit that either all succeed together or all fail together, ensuring data consistency even during system failures.

---

## Why Transactions Exist (The Problem)

Modern applications often need to perform multiple related database operations as a cohesive unit. For example, transferring money between accounts requires:
1. Debiting the source account
2. Crediting the destination account
3. Logging the transaction

If the system fails after step 1 but before step 2, money disappears from the system. Without transactions, developers would need to manually handle rollbacks, error recovery, and consistency checks—complex, error-prone, and not feasible for high-concurrency systems.

Transactions provide a built-in mechanism to treat these operations as **indivisible units of work**, guaranteeing that either all changes are applied permanently (committed) or none are (rolled back).

---

## 🎯 The Classic Analogy

Think of a transaction as **a legal contract signing ceremony**:
- All parties must sign for the contract to be valid
- If anyone refuses or walks away mid-process, the entire agreement is void
- There's no such thing as "partially signed"—it's either fully executed or not at all
- Once signed and notarized, the agreement survives even if the building burns down

Similarly, a database transaction requires all steps to complete successfully. If any step fails, the entire transaction is rolled back as if it never happened. Once committed, the changes are permanent and survive system crashes.

---

## 🖼️ Transactions in One Picture — See It

> This diagram from **Google Cloud** illustrates the transaction lifecycle with commit and rollback paths.

![Transaction Lifecycle — Google Cloud](https://cloud.google.com/database/images/transaction-lifecycle.svg)

📎 Source: [Transactions — Google Cloud Documentation](https://cloud.google.com/datastore/docs/concepts/transactions)

---

## ⚙️ The Transaction Lifecycle (step by step)

### Basic Transaction Structure (SQL)

```sql
BEGIN TRANSACTION;  -- or START TRANSACTION, or just BEGIN
-- Multiple SQL statements go here
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
INSERT INTO transaction_log (from_acct, to_acct, amount, timestamp)
VALUES (1, 2, 100, NOW());
COMMIT;  -- Makes all changes permanent
```

### What Happens Behind the Scenes

1. **BEGIN**: Marks the start of a transaction
   - Database starts tracking all changes in a temporary workspace
   - Changes are invisible to other transactions (depending on isolation level)

2. **SQL Statements**: Execute normally but...
   - Writes go to transaction log and temporary buffers
   - Reads see your uncommitted changes (depending on isolation)
   - Other transactions see pre-transaction state (depending on isolation)

3. **COMMIT**: 
   - Database writes all changes to persistent storage (write-ahead log)
   - Makes changes visible to all other transactions
   - Releases locks held by the transaction
   - **If successful**: Changes are permanent
   - **If failed**: Automatic rollback initiated

4. **ROLLBACK** (explicit or automatic on failure):
   - Undoes all changes made in the transaction
   - Restores database to pre-transaction state
   - Releases any locks
   - Transaction ends with no permanent effect

### Transaction Outcomes

| Scenario | Result |
|----------|--------|
| All statements succeed → `COMMIT` | Changes permanently saved |
| Any statement fails → Automatic `ROLLBACK` | No changes saved |
| User executes explicit `ROLLBACK` | No changes saved |
| System crash during transaction | Automatic recovery → `ROLLBACK` |
| System crash after `COMMIT` | Changes preserved via recovery log |

---

## 🧠 Backend Developer Depth — What Nobody Tells Beginners

### 1. Transactions ≠ Performance Killer (When Used Right)
- **Myth**: "Transactions slow down my application"
- **Reality**: Poor transaction design (too long, too much contention) causes slowness
- **Best Practice**: Keep transactions **short and focused**—only include operations that must be atomic
- **Example**: Don't open a transaction, then wait for user input, then close it

### 2. The Two-Phase Commit Illusion
- Single-database transactions use a simplified commit protocol
- **Distributed transactions** (across multiple databases/services) require **two-phase commit (2PC)** or alternatives like **Saga patterns**
- 2PC has blocking problems—modern systems often use eventual consistency or event-driven approaches

### 3. Savepoints: Partial Rollback Within a Transaction
```sql
BEGIN;
UPDATE accounts SET balance = balance - 50 WHERE id = 1;
SAVEPOINT step1;
UPDATE accounts SET balance = balance + 50 WHERE id = 2;
-- Oops, realized we need to check something
ROLLBACK TO SAVEPOINT step1;  -- Undoes step2, keeps step1
UPDATE accounts SET balance = balance + 50 WHERE id = 3;  -- Different action
COMMIT;
```
Useful for complex business logic with conditional steps.

### 4. Implicit vs. Explicit Transactions
- **Autocommit mode** (default in many clients): Each statement is its own transaction
- **Explicit transactions**: You control BEGIN/COMMIT/ROLLBACK
- **Important**: Know your database/client's default behavior!

### 5. Transaction Logs and Recovery
- Databases use **write-ahead logging (WAL)**: Changes logged before applied to data files
- Enables crash recovery: Replay log to redo committed transactions, undo uncommitted ones
- **Why your data survives power loss**: Log is flushed to disk before COMMIT returns success

### 6. Isolation Levels Interact with Transactions
- Transactions define **atomicity and durability**
- **Isolation levels** (READ COMMITTED, REPEATABLE READ, SERIALIZABLE) define **visibility** of concurrent transactions
- Choosing the right isolation level is crucial for correctness within your transaction boundaries

### 7. Application-Level Transactions
- Sometimes you need transaction-like behavior across non-transactional systems (files, APIs, caches)
- Patterns: **Saga** (compensating transactions), **Eventual consistency**, **Idempotency keys**
- Example: Booking a flight (payment service, seat reservation, email notification) might use a Saga

---

## 🛠️ Real Tools & Commands

### Raw SQL (psql/MySQL/SQLite)

```sql
-- Explicit transaction
BEGIN;
UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 123;
UPDATE orders SET status = 'processing' WHERE order_id = 456;
INSERT INTO audit (action, user_id, timestamp) VALUES ('order_processed', 789, NOW());
COMMIT;

-- With error handling (psql)
BEGIN;
DO $$
BEGIN
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    -- Simulate error
    IF (SELECT balance FROM accounts WHERE id = 1) < 0 THEN
        RAISE EXCEPTION 'Insufficient funds';
    END IF;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END $$;
COMMIT;
```

### Django ORM

```python
from django.db import transaction

# Method 1: Context manager (recommended)
with transaction.atomic():
    # All operations in this block are in one transaction
    Account.objects.filter(id=1).update(balance=F('balance') - 100)
    Account.objects.filter(id=2).update(balance=F('balance') + 100)
    TransactionLog.objects.create(
        from_account_id=1,
        to_account_id=2,
        amount=100
    )
# If no exception → COMMIT
# If exception → ROLLBACK

# Method 2: Decorator
@transaction.atomic
def transfer_funds(from_id, to_id, amount):
    # Same logic as above
    pass

# Method 3: Manual control (advanced)
from django.db import connection
with connection.cursor() as cursor:
    cursor.execute("BEGIN")
    try:
        cursor.execute("UPDATE accounts SET balance = balance - 100 WHERE id = 1")
        cursor.execute("UPDATE accounts SET balance = balance + 100 WHERE id = 2")
        cursor.execute("COMMIT")
    except:
        cursor.execute("ROLLBACK")
        raise
```

### SQLAlchemy (Python)

```python
from sqlalchemy import create_engine, text
from sqlalchemy.orm import sessionmaker

# Using session (recommended)
Session = sessionmaker(bind=engine)
session = Session()

try:
    # All operations in this session.flush() are part of same transaction
    session.execute(text("UPDATE accounts SET balance = balance - 100 WHERE id = 1"))
    session.execute(text("UPDATE accounts SET balance = balance + 100 WHERE id = 2"))
    session.add(TransactionLog(from_acct=1, to_acct=2, amount=100))
    session.commit()  # Persists all changes
except:
    session.rollback()  # Undoes all changes
    raise
finally:
    session.close()
```

### Explain Transaction Behavior (Postgres)

```sql
-- See isolation level and transaction status
SHOW transaction_isolation;
SELECT txid_current();  -- Get current transaction ID

-- Monitor active transactions
SELECT * FROM pg_stat_activity WHERE state = 'active';
```

---

## 🔥 Cheat Sheet

| Concept | Key Point | When to Use |
|---------|-----------|-------------|
| **ACID Properties** | Transactions guarantee Atomicity, Consistency, Isolation, Durability | Foundation for reliable data operations |
| **BEGIN/COMMIT** | Explicit transaction boundaries | When you need multiple statements to succeed/fail together |
| **ROLLBACK** | Undo all changes in current transaction | On error, or when business logic dictates cancellation |
| **SAVEPOINT** | Partial rollback within transaction | Complex logic with conditional steps |
| **Autocommit** | Each statement is own transaction (default) | Simple queries, reads, independent operations |
| **Nested Transactions** | Not truly nested in most DBs (savepoints simulate) | Avoid unless using savepoints |
| **Distributed Transactions** | Across multiple systems (complex) | Use Sagas or event-driven instead of 2PC when possible |
| **Transaction Log** | WAL enables crash recovery | Never disable for production durability |
| **Short Transactions** | Hold locks briefly, reduce contention | **Critical for performance** |
| **Long Transactions** | Hold locks, block others, risk timeouts | Avoid—break into smaller units |

---

## 🧲 How to Memorize

- **ATOMIC acronym**:
  - **A**ll-or-nothing (Atomicity)
  - **T**akes effect together (Consistency)
  - **O**perations isolated (Isolation)
  - **M**akes changes permanent (Durability)
  - **I**nitiated with BEGIN, ended with COMMIT/ROLLBACK
- **Contract Analogy**: "All parties sign or deal is void; once signed, it's binding."
- **Bank Transfer Mindset**: "If money leaves one account, it must appear in another—or nowhere."
- **Crash Test**: "If power fails mid-transaction, did money vanish or duplicate?"
- **Phrase**: "Begin together, end together, survive everything."

---

## 🧲 Quick Self-Check

<details>
<summary>1. What happens if a system crashes after the first UPDATE in a money transfer but before the COMMIT?</summary>

The transaction is automatically rolled back during recovery—neither update persists. Money remains in the source account (no loss or creation).
</details>

<details>
<summary>2. Two transactions run concurrently: T1 transfers $100 from A→B, T2 transfers $50 from B→A. Both read balances, then update. What isolation level prevents lost updates?</summary>

`SERIALIZABLE` or `REPEATABLE READ` with proper locking (e.g., `SELECT FOR UPDATE`). At `READ COMMITTED`, lost updates can occur without explicit locking.
</details>

<details>
<summary>3. Why shouldn't you open a transaction, then make an API call to an external service, then commit?</summary>

External calls are slow and unpredictable—holding database resources (locks, connection) while waiting causes poor scalability, potential timeouts, and blocks other transactions. Keep transactions short and database-only.
</details>

<details>
<summary>4. What's the difference between `ROLLBACK TO SAVEPOINT` and a full `ROLLBACK`?</summary>

`ROLLBACK TO SAVEPOINT` undoes only changes made after the savepoint, preserving earlier work. Full `ROLLBACK` undoes all changes in the transaction.
</details>

<details>
<summary>5. In which scenario would you NOT want to use a database transaction?</summary>

When operations must eventually succeed but can tolerate temporary inconsistency (e.g., social media likes, analytics events)—use eventual consistency or message queues instead of blocking on distributed transactions.
</details>