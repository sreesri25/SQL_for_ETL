# SQL Joins — Complete Reference Guide

> A structured, reference guide covering all SQL join types with real-world use cases in data quality, ETL testing, and pipeline transformations. This guide can also be used for quick revision for interview preparation.

---

## Table of Contents

1. [Join Types Overview](#join-types-overview)
2. [INNER JOIN](#1-inner-join)
3. [LEFT JOIN](#2-left-join)
4. [RIGHT JOIN](#3-right-join)
5. [FULL OUTER JOIN](#4-full-outer-join)
6. [CROSS JOIN](#5-cross-join)
7. [SELF JOIN](#6-self-join)
8. [Common Mistakes & Fixes](#common-mistakes--fixes)
9. [Practical Use Cases](#practical-use-cases)
10. [Mini Project: Customers & Orders](#mini-project-customers--orders)
11. [Key Takeaways](#key-takeaways)

---

## Join Types Overview

| Join Type | Returns | NULL Side | Best For |
|-----------|---------|-----------|----------|
| INNER JOIN | Matching rows only | None | Active/confirmed relationships |
| LEFT JOIN | All left + matches | Right | Finding missing data, optional relationships |
| RIGHT JOIN | All right + matches | Left | Target-side validation in ETL |
| FULL OUTER JOIN | All rows from both | Both sides | Reconciliation, full gap analysis |
| CROSS JOIN | All combinations | None | Dimension tables, test data generation |
| SELF JOIN | Rows joined to same table | Optional | Hierarchies, org charts, parent-child |

---

## 1. INNER JOIN

Returns only rows that have matching values in **both** tables.

```sql
SELECT a.col1, b.col2
FROM table_a a
INNER JOIN table_b b
  ON a.id = b.a_id;
```

**Example — Customers who placed orders:**
```sql
SELECT c.customer_name, o.order_id, o.amount
FROM customers c
INNER JOIN orders o
  ON c.customer_id = o.customer_id;
```
> Customers without any orders are excluded from the result.

**✓ Use when:** You only want records that exist in both tables, joining foreign keys to primary keys.  
**✗ Avoid when:** You need all records from one side regardless of matches.  
**Real-world:** Sales dashboards, active user queries, confirmed transaction reports.

---

## 2. LEFT JOIN

Returns **all rows from the left table**, with matching rows from the right. Non-matching right rows fill with `NULL`.

```sql
SELECT a.col1, b.col2
FROM table_a a
LEFT JOIN table_b b
  ON a.id = b.a_id;
```

**Example — All customers, with or without orders:**
```sql
SELECT c.customer_name, o.order_id, o.amount
FROM customers c
LEFT JOIN orders o
  ON c.customer_id = o.customer_id;
```
> Customers without orders appear with `NULL` in order columns.

**✓ Use when:** You want all records from the primary table and optional matches from another.  
**✗ Avoid when:** You only care about matched records — use INNER JOIN.  
**Real-world:** User activity audits, "never logged in" reports, optional profile data.

**Pro tip — Finding unmatched rows:**
```sql
-- Customers who have NEVER placed an order
SELECT c.customer_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;
```

---

## 3. RIGHT JOIN

Returns **all rows from the right table**, with matching rows from the left. Functionally equivalent to a LEFT JOIN with swapped table order.

```sql
SELECT a.col1, b.col2
FROM table_a a
RIGHT JOIN table_b b
  ON a.id = b.a_id;
```

**Example — All orders, including those with deleted customers:**
```sql
SELECT c.customer_name, o.order_id, o.amount
FROM customers c
RIGHT JOIN orders o
  ON c.customer_id = o.customer_id;
```

**✓ Use when:** The right (second) table must be fully preserved — common in ETL target validation.  
**✗ Avoid when:** You can rewrite as a LEFT JOIN by swapping tables (most style guides prefer this).  
**Real-world:** Post-migration validation, ensuring all target records are accounted for.

---

## 4. FULL OUTER JOIN

Returns **all rows from both tables**, with NULLs wherever there's no match on either side.

```sql
SELECT a.col1, b.col2
FROM table_a a
FULL OUTER JOIN table_b b
  ON a.id = b.a_id;
```

**Example — All customers AND all orders, matched where possible:**
```sql
SELECT c.customer_name, o.order_id, o.amount
FROM customers c
FULL OUTER JOIN orders o
  ON c.customer_id = o.customer_id;
```

**✓ Use when:** Reconciling two systems, finding records missing on either side.  
**✗ Avoid when:** Large tables without indexes — can be expensive. Use LEFT/RIGHT if you only need one side.  
**Real-world:** Bank statement vs. internal ledger reconciliation, source vs. target ETL comparison.

> **Note:** Not supported in MySQL — emulate with `LEFT JOIN UNION ALL RIGHT JOIN WHERE left IS NULL`.

---

## 5. CROSS JOIN

Returns the **Cartesian product** — every row in table A paired with every row in table B.

```sql
SELECT a.col1, b.col2
FROM table_a a
CROSS JOIN table_b b;
```

**Example — Every product-color combination:**
```sql
SELECT p.product_name, c.color_name
FROM products p
CROSS JOIN colors c;
-- 10 products × 5 colors = 50 rows
```

**✓ Use when:** Generating all possible combinations intentionally — dimension tables, date scaffolding, QA test data.  
**✗ Avoid when:** On large tables (result = M × N rows). Also avoid implicit Cartesian joins from missing `ON` clauses.  
**Real-world:** Calendar table generation, product-location inventory scaffolding.

---

## 6. SELF JOIN

Joins a table to **itself** using table aliases to treat the same table as two separate entities.

```sql
SELECT a.col1, b.col2
FROM table_a a
JOIN table_a b
  ON a.parent_id = b.id;
```

**Example — Employee and their manager:**
```sql
SELECT e.employee_name, m.employee_name AS manager_name
FROM employees e
LEFT JOIN employees m
  ON e.manager_id = m.employee_id;
```

**✓ Use when:** Hierarchical or recursive data lives in a single table — org charts, threaded comments, bill-of-materials.  
**✗ Avoid when:** The hierarchy is very deep — use recursive CTEs (`WITH RECURSIVE`) instead.  
**Real-world:** Reporting lines, geographic hierarchies, parent-child product relationships.

---

## Common Mistakes & Fixes

### 1. Joining on Non-Unique Columns

```sql
-- ✗ Wrong: names can have duplicates
SELECT * FROM orders o
JOIN customers c ON o.customer_name = c.customer_name;

-- ✓ Correct: join on primary/foreign keys
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

### 2. Data Type Mismatch

```sql
-- ✗ Wrong: INT vs VARCHAR — implicit cast disables index
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.id;

-- ✓ Correct: cast explicitly, or fix the schema
SELECT * FROM orders o
JOIN customers c ON CAST(o.customer_id AS VARCHAR) = c.id;
```

### 3. Accidental CROSS JOIN (Missing ON Clause)

```sql
-- ✗ Wrong: implicit comma join — Cartesian product!
SELECT * FROM orders, customers;

-- ✓ Correct: explicit JOIN with ON clause
SELECT * FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id;
```

### 4. Ambiguous Column Names

```sql
-- ✗ Wrong: which table does 'id' come from?
SELECT id, name FROM orders JOIN customers ON ...;

-- ✓ Correct: always alias and prefix columns
SELECT o.order_id, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;
```

### 5. NULL Keys That Never Match

```sql
-- NULLs never equal NULLs in join conditions
-- NULL = NULL → UNKNOWN, not TRUE

-- ✓ Handle NULLs explicitly (or investigate their root cause)
SELECT * FROM orders o
LEFT JOIN customers c
  ON COALESCE(o.customer_id, -1) = COALESCE(c.customer_id, -1);
```

---

## Practical Use Cases

### Data Quality Assurance

**Orphan Record Detection:**
```sql
SELECT o.order_id, o.customer_id
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

**Duplicate Detection (SELF JOIN):**
```sql
SELECT a.customer_id, b.customer_id, a.email
FROM customers a
JOIN customers b
  ON a.email = b.email AND a.customer_id < b.customer_id;
```

**Cross-System Reconciliation:**
```sql
SELECT
  s.order_id AS source_id,
  t.order_id AS target_id,
  s.amount   AS source_amt,
  t.amount   AS target_amt
FROM source_orders s
FULL OUTER JOIN target_orders t ON s.order_id = t.order_id
WHERE s.amount != t.amount
   OR s.order_id IS NULL
   OR t.order_id IS NULL;
```

---

### ETL Testing

**Row Count Validation:**
```sql
SELECT
  COUNT(*)                                               AS total_source,
  SUM(CASE WHEN t.id IS NOT NULL THEN 1 ELSE 0 END)     AS matched,
  SUM(CASE WHEN t.id IS NULL     THEN 1 ELSE 0 END)     AS unmatched
FROM source s
LEFT JOIN target t ON s.id = t.id;
```

**Schema Drift Detection:**
```sql
SELECT s.column_name
FROM information_schema.columns s
LEFT JOIN information_schema.columns t
  ON s.column_name = t.column_name
  AND t.table_name = 'target_table'
WHERE s.table_name = 'source_table'
  AND t.column_name IS NULL;
```

---

### Data Pipeline Transformations

**Star Schema Enrichment:**
```sql
SELECT
  f.sale_date, f.amount,
  d.product_name, d.category,
  c.customer_name, c.region
FROM fact_sales f
INNER JOIN dim_product  d ON f.product_id  = d.product_id
INNER JOIN dim_customer c ON f.customer_id = c.customer_id;
```

**SCD Type 2 Temporal Lookup:**
```sql
SELECT f.*, d.product_name, d.price
FROM fact_sales f
INNER JOIN dim_product d
  ON f.product_id = d.product_id
  AND f.sale_date BETWEEN d.effective_date AND d.expiry_date
  AND d.is_current = TRUE;
```

---

## Mini Project: Customers & Orders

### Schema

```sql
CREATE TABLE customers (
  customer_id   INT PRIMARY KEY,
  customer_name VARCHAR(100),
  email         VARCHAR(100),
  region        VARCHAR(50),
  signup_date   DATE
);

CREATE TABLE orders (
  order_id    INT PRIMARY KEY,
  customer_id INT REFERENCES customers(customer_id),
  product     VARCHAR(100),
  amount      DECIMAL(10,2),
  order_date  DATE,
  status      VARCHAR(20)
);
```

### With SQL JOIN — Single Query

```sql
SELECT
  c.region,
  COUNT(DISTINCT c.customer_id) AS active_customers,
  COUNT(o.order_id)             AS total_orders,
  SUM(o.amount)                 AS total_revenue,
  AVG(o.amount)                 AS avg_order_value
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE o.status = 'completed'
GROUP BY c.region
ORDER BY total_revenue DESC;
```

### Without JOIN — Anti-Pattern

```sql
-- Step 1: Fetch customers (one round-trip)
SELECT customer_id, region FROM customers;

-- Step 2: Fetch orders (second round-trip)
SELECT customer_id, amount, status FROM orders WHERE status = 'completed';

-- Step 3: Merge in application code (Python, Java, etc.)
-- Manual grouping, aggregation, and null handling — all in memory
```

### Why JOINs Win

| Aspect | With JOIN | Without JOIN |
|--------|-----------|--------------|
| DB round-trips | 1 | 2+ |
| Optimization | DB query planner + indexes | None |
| Memory usage | Server-side filtered result | Full datasets in app memory |
| Aggregation | Server-side (SUM, COUNT) | Manual in application code |
| Maintainability | Single SQL artifact | Split across DB + app layers |
| Testability | One query to validate | Multiple components to test |

---

## Key Takeaways

1. **Default to INNER JOIN** when you need confirmed matches. It's the most performant.
2. **Use LEFT JOIN + IS NULL** to find missing/orphaned data — the most common DQ check.
3. **FULL OUTER JOIN** is your reconciliation tool — replaces two separate LEFT/RIGHT queries.
4. **CROSS JOIN is intentional** — if you see one and didn't mean it, you're looking at a bug.
5. **Always alias tables and prefix columns** — prevents ambiguity and makes queries self-documenting.
6. **NULL keys never match** — treat them as data quality signals, not edge cases to paper over.
7. **Join on primary/foreign keys**, not names or emails — uniqueness guarantees correct cardinality.
8. **The DB engine is faster than your application code** at joining, filtering, and aggregating.

---

## Contributing

Found an error or want to add a use case? Open a PR or issue. All skill levels welcome.

---

*Built for knowledge sharing and interview preparation. Use freely.*
