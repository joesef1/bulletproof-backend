# 🔗 Joins: Combining Data Across Tables

> **One-sentence summary:** **SQL joins** let you combine rows from two or more tables based on related columns, turning fragmented data into a unified result set.

---

## Why Joins Exist (The Problem)

In a normalized database, information is split across multiple tables to avoid redundancy and update anomalies. For example, you might have a `users` table storing user profiles and an `orders` table storing purchase records, linked by a `user_id` foreign key. To answer questions like “Show all orders with the customer’s name and email,” you need to pull data from both tables simultaneously—this is exactly what joins do.

Without joins, you’d have to run separate queries and manually merge the results in application code, which is inefficient, error‑prone, and doesn’t scale.

---

## 🎯 The Classic Analogy

Think of a join as a **database version of a spreadsheet VLOOKUP** (or INDEX/MATCH):

- The **left table** is like your main sheet with a list of IDs.
- The **right table** is a lookup sheet that holds extra details for those IDs.
- The join condition (`ON left.id = right.foreign_id`) tells the database how to match rows from the two sheets.
- The result is a new “virtual sheet” where each row contains columns from both tables, matched by the key.

Just as VLOOKUP pulls a price from a product list into an invoice, a join pulls a user’s name from the `users` table into each row of the `orders` table.

---

## 🖼️ Joins in One Picture — See It

> This diagram from **Mode Analytics** visualizes the different join types using two overlapping circles (left and right tables).

![SQL Joins Visualized — Mode Analytics](https://cdn.modeanalytics.com/blog/images/2013/08/SQL-Joins.png)

📎 Source: [A Visual Explanation of SQL Joins — Mode Analytics](https://mode.com/blog/sql-join-visualized/)

---

## ⚙️ The Join Types (step by step)

Assume two tables:

```sql
-- users
+----+----------+------------------+
| id | name     | email            |
+----+----------+------------------+
| 1  | Alice    | alice@example.com|
| 2  | Bob      | bob@example.com  |
| 3  | Charlie  | charlie@example.com|
+----+----------+------------------+

-- orders
+----+---------+--------+
| id | user_id | amount |
+----+---------+--------+
| 101| 1       | 50.00  |
| 102| 1       | 30.00  |
| 103| 4       | 20.00  | -- user_id 4 does not exist in users
+----+---------+--------+
```

### INNER JOIN: Only matching rows

```sql
SELECT users.name, orders.amount
FROM users
INNER JOIN orders ON users.id = orders.user_id;
```

Result:

| name   | amount |
|--------|--------|
| Alice  | 50.00  |
| Alice  | 30.00  |

Only rows where the join condition succeeds appear.

### LEFT (OUTER) JOIN: All rows from left table, matched rows from right

```sql
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

Result:

| name     | amount |
|----------|--------|
| Alice    | 50.00  |
| Alice    | 30.00  |
| Bob      | NULL   |
| Charlie  | NULL   |

Every user appears; if there’s no matching order, the order columns are NULL.

### RIGHT (OUTER) JOIN: All rows from right table, matched rows from left

```sql
SELECT users.name, orders.amount
FROM users
RIGHT JOIN orders ON users.id = orders.user_id;
```

Result:

| name     | amount |
|----------|--------|
| Alice    | 50.00  |
| Alice    | 30.00  |
| NULL     | 20.00  | -- order for user_id 4 (no matching user)
```

Every order appears; if there’s no matching user, the user columns are NULL.

### FULL (OUTER) JOIN: All rows from both tables, with NULLs where missing

```sql
SELECT users.name, orders.amount
FROM users
FULL JOIN orders ON users.id = orders.user_id;
```

Result:

| name     | amount |
|----------|--------|
| Alice    | 50.00  |
| Alice    | 30.00  |
| Bob      | NULL   |
| Charlie  | NULL   |
| NULL     | 20.00  |
```

Combines LEFT and RIGHT: all users and all orders appear, with NULLs filling gaps.

### CROSS JOIN: Cartesian product (every left row paired with every right row)

```sql
SELECT users.name, orders.amount
FROM users
CROSS JOIN orders;
```

Result: 3 users × 3 orders = 9 rows, each user paired with each order regardless of user_id.

### SELF JOIN: Joining a table to itself

Useful for hierarchical data (e.g., an `employees` table with a `manager_id` foreign key referencing the same table).

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

### NATURAL JOIN: Joins on all columns with the same name

```sql
SELECT *
FROM users
NATURAL JOIN orders;
```

Only works if the tables share identically‑named columns (e.g., both have `user_id`). Generally avoided in production because it’s fragile—adding a column with the same name changes the join implicitly.

---

## 🧠 Backend Developer Depth — What Nobody Tells Beginners

### 1. Join order and performance matter  
The optimizer usually picks the best order, but for large tables you can influence it with `JOIN` syntax (e.g., putting the smaller table first in a `LEFT JOIN` can reduce rows early). Always examine `EXPLAIN ANALYZE` to see the chosen plan.

### 2. Use table aliases for readability  
`SELECT u.name, o.amount FROM users u JOIN orders o ON u.id = o.user_id;` is cleaner, especially with long names or self‑joins.

### 3. Beware of NULLs in outer joins  
When using `LEFT JOIN`, remember that columns from the right table can be NULL. Aggregations like `COUNT(o.id)` will count only non‑NULLs; `COUNT(*)` counts rows (including those with NULL order data). Choose the correct metric.

### 4. Index join columns  
Foreign‑key columns used in `ON` clauses should be indexed (most ORMs create these automatically). Missing indexes cause full table scans and turn a fast join into a performance nightmare.

### 5. Avoid “join fever”  
Sometimes denormalizing or using application‑level caching is faster than a complex multi‑table join. Profile real‑world queries; a join of five large tables might be slower than two simpler queries plus in‑memory merging.

### 6. Different SQL dialects have nuances  
- MySQL: `RIGHT JOIN` is rarely used (you can rewrite as `LEFT JOIN` by swapping tables).  
- PostgreSQL: Supports `FULL JOIN` and `LATERAL` joins (correlated subqueries expressed as joins).  
- SQLite: Lacks `RIGHT JOIN` and `FULL OUTER JOIN`; emulate with `UNION` of left and right joins.  
- SQL Server: Supports all join types plus `APPLY` (similar to lateral).

---

## 🛠️ Real Tools & Commands

### Raw SQL (psql)

```sql
-- Inner join
SELECT u.name, o.amount
FROM users u
JOIN orders o ON u.id = o.user_id;

-- Left join with aggregation
SELECT u.name, COUNT(o.id) AS order_count, SUM(o.amount) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;
```

### Django ORM

```python
# inner join (default)
User.objects.filter(orders__amount__gt=0).values('name', 'orders__amount')

# left join (annotate with Count)
from django.db.models import Count, Sum
User.objects.annotate(
    order_count=Count('orders'),
    total_spent=Sum('orders__amount')
).values('name', 'order_count', 'total_spent')
```

### SQLAlchemy Core

```python
from sqlalchemy import select, join

j = join(users_table, orders_table, users_table.c.id == orders_table.c.user_id)
stmt = select(users_table.c.name, orders_table.c.amount).select_from(j)
```

### Explain plan (Postgres)

```sql
EXPLAIN ANALYZE
SELECT u.name, o.amount
FROM users u
JOIN orders o ON u.id = o.user_id;
```

---

## 🔥 Cheat Sheet

| Join Type          | Keeps rows from…       | When to use                                                                    |
|--------------------|------------------------|--------------------------------------------------------------------------------|
| INNER JOIN         | Both tables (matching) | You need only data that exists in **both** sides.                              |
| LEFT (OUTER) JOIN  | Left table (all)       | You want every record from the left, even if there’s no match on the right.   |
| RIGHT (OUTER) JOIN | Right table (all)      | Same as left, but you think of the right as primary (rarely needed).           |
| FULL (OUTER) JOIN  | Both tables (all)      | You need a complete view of both sides, with NULLs where missing.              |
| CROSS JOIN         | Cartesian product      | You truly need every possible combination (rare, watch for huge results).      |
| SELF JOIN          | Same table (self)      | Hierarchical or graph‑like data (e.g., employee‑manager).                      |
| NATURAL JOIN       | Same‑name columns      | Quick prototyping; avoid in production due to fragility.                       |

---

## 🧲 How to Memorize

- **Venn diagram mental model** – Draw two overlapping circles:  
  - Inner = overlap only  
  - Left = left circle + overlap  
  - Right = right circle + overlap  
  - Full = both circles entirely  
  - Cross = every point in left paired with every point in right (not a Venn, think grid)
- **Ali phrase**: **“Left leaves none behind; Right returns none denied; Full keeps all; Inner needs both.”**
- **Crash course**:  
  - **I**nner = **I**ntersection  
  - **L**eft = **L**eft + intersection  
  - **R**ight = **R**ight + intersection  
  - **F**ull = **F**ull set (left ∪ right)  
  - **C**ross = **C**artesian product  

---

## 🧲 Quick Self-Check

<details>
<summary>1. Write a query that returns all users and the number of orders they’ve placed, showing zero for users with no orders.</summary>

```sql
SELECT u.name, COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;
```
</details>

<details>
<summary>2. What is the difference between `LEFT JOIN` and `LEFT OUTER JOIN`?</summary>
They are synonymous; `OUTER` is optional and implied when you specify `LEFT`, `RIGHT`, or `FULL`.
</details>

<details>
<summary>3. Suppose you want to find orders whose user no longer exists in the `users` table (orphaned orders). Which join would you use?</summary>
Use a `LEFT JOIN` from `orders` to `users` and filter where the user columns are `NULL`:

```sql
SELECT o.*
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;
```
</details>

<details>
<summary>4. Why should you avoid `NATURAL JOIN` in production code?</summary>
Because it automatically joins on every column with the same name. Adding a new column that happens to share a name between tables changes the join semantics silently, potentially breaking the query or producing unexpected results.
</details>

<details>
<summary>5. How can you turn a `RIGHT JOIN` into a `LEFT JOIN` without changing the result set?</summary>
Swap the two tables in the `FROM` clause and change `RIGHT JOIN` to `LEFT JOIN`. For example:

```sql
-- Original
SELECT * FROM A RIGHT JOIN B ON A.id = B.a_id;

-- Equivalent
SELECT * FROM B LEFT JOIN A ON B.a_id = A.id;
```
</details>