# Week 47 — Exercises: Indexes and Query Performance

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises walk you through measuring, creating, and evaluating indexes on your TrailShop database. You will use PostgreSQL's `EXPLAIN ANALYZE` to gather real execution data and make evidence-based decisions about indexing.

---

## 1. TrailShop Project Task

### Step 1 — Baseline Measurement

Before creating any indexes, run `EXPLAIN ANALYZE` on the following five queries. For each, record the **scan type**, **execution time (ms)**, and **rows removed by filter**.

```sql
-- Query A: Find a product by name
SELECT * FROM products WHERE name = 'Trail Runner Pro 3';

-- Query B: Get all items for a specific order
SELECT * FROM order_items WHERE order_id = 42;

-- Query C: Get all orders for a customer
SELECT * FROM orders WHERE customer_id = 7;

-- Query D: Get products in a category
SELECT * FROM product_categories WHERE category_id = 3;

-- Query E: Join order_items to products
SELECT oi.quantity, p.name, p.price
FROM order_items oi
JOIN products p ON p.product_id = oi.product_id
WHERE oi.order_id = 42;
```

Record your results in a table like this:

| Query | Scan Type | Exec Time (ms) | Rows Removed by Filter |
| ----- | --------- | -------------- | ---------------------- |
| A     | Seq Scan  |   4.102        |            11          |
| B     | Seq Scan  |   1.948        |            10          |
| C     | Seq Scan  |   2.117        |            5           |
| D     | Bitmap Heap Scan |  2.317              |           11        |
| E     | Hash Join |   0.172        |            10           |

### Step 2 — Create Indexes

Now create the following indexes:

```sql
CREATE INDEX idx_product_name ON products (name);
CREATE INDEX idx_order_item_order_id ON order_items (order_id);
CREATE INDEX idx_order_item_product_id ON order_items (product_id);
CREATE INDEX idx_orders_customer_id ON orders (customer_id);
CREATE INDEX idx_product_categories_category_id ON product_categories (category_id);
```

### Step 3 — Re-measure

Because the project database is so small, the PostgreSQL optimizer will most likely default to Seq Scan even after creating indexes.

You can force PostgreSQL to use indexes by disabling the Seq scan using following commands:

```sql
--- Turn the seq scan off
SET enable_seqscan = OFF;

--- Turn the seq scan back on
SET enable_seqscan = ON;
```

Run the same five queries with `EXPLAIN ANALYZE` again. Fill in your comparison table:

| Query | Before Scan | Before Time (ms) | After Scan | After Time (ms) | Improvement Factor |
| ----- | ----------- | ---------------- | ---------- | --------------- | ------------------ |
| A     | Seq Scan    | 4.102            | Seq Scan   |     0.053       |     77.4           |
| B     | Seq Scan    | 1.948            | Seq Scan   |     0.049       |     39.8           |
| C     | Seq Scan    | 2.117            | Seq Scan   |     0.139       |     15.2           |
| D     | Bitmap HS   | 2.317            | Seq Scan   |     0.048       |     48.3           |
| E     | Hash Join   | 0.172            | Hash Join  |     0.089       |     1.9            |

### Step 4 — Recommendation

Write a paragraph (4–6 sentences) recommending which indexes to keep in production and why. Consider table size, query frequency, and write overhead. If any index showed minimal improvement, explain why you might drop it.

> [!NOTE]
>
> The times show strong improvement for A–D, from 15.2 to 77.4 times, so I would keep indexes on products.name, order_items.order_id, orders.customer_id, and product_categories.category_id if these queries are common. The join in E improved only 1.9 times, so I would reconsider the order_items.product_id index unless other frequent joins need it. However, the After Scan column still shows Seq Scan or Hash Join rather than index scans, so I would check the EXPLAIN plans before attributing the time changes to the indexes. Indexes use storage and add work to inserts and updates, so I would keep only those that help common production queries and review them again as the tables grow.

---

## 2. Theory Review

Answer the following questions in your own words. See the corresponding theory sections for reference.

**Question 1** — What is a database index, and why is it needed? (See Theory Section 1)

> [!NOTE]
>
> A database index is an extra structure that helps PostgreSQL find rows without checking the whole table. It is useful for making searches and joins faster.

**Question 2** — Explain the difference between B-tree search at O(log n) and sequential scan at O(n). If a table has 1,000,000 rows, approximately how many comparisons does each approach need? (See Theory Section 2)

> [!NOTE]
>
> A B-tree can find a value in about 20 comparisons for 1,000,000 rows, while a sequential scan could check up to 1,000,000 rows. The B-tree narrows down the search at each step instead of checking every row.

**Question 3** — PostgreSQL automatically creates indexes on primary keys. Why doesn't it also auto-index foreign key columns? (See Theory Section 3)

> [!NOTE]
>
> A primary key always needs an index, but foreign keys are not always searched or joined in the same way. Automatically indexing every foreign key would use extra space and slow down inserts and updates, so PostgreSQL leaves that choice to the database designer.

**Question 4** — What is the difference between an Index Scan and an Index Only Scan? When can PostgreSQL use Index Only Scan? (See Theory Section 4)

> [!NOTE]
>
> An Index Scan uses the index to find matching rows and then reads those rows from the table. An Index Only Scan can return the requested columns directly from the index, when the index contains all of them and PostgreSQL can confirm row visibility.

**Question 5** — Give three situations where you should NOT create an index. Explain each briefly. (See Theory Section 5)

> [!NOTE]
>
> I would not create an index on a very small table, on a column with only a few repeated values if the query returns many rows, or on a column that is rarely searched but updated often. In those cases the index may not speed things up enough to justify the extra space and write work.

**Question 6** — What does `EXPLAIN ANALYZE` show that plain `EXPLAIN` does not? Why does this matter? (See Theory Section 6)

> [!NOTE]
>
> Plain EXPLAIN shows the plan PostgreSQL estimates it will use, while EXPLAIN ANALYZE runs the query and shows actual rows and execution time. This matters because the actual results can be different from the estimates.

**Question 7** — What is a partial index? Give one concrete example where a partial index on a TrailShop table would be useful. (See Theory Section 7)

> [!NOTE]

>
> A partial index only indexes rows that match a condition. For example, if TrailShop often searches pending orders by date, an index on orders.order_date where status = 'pending' would cover only those orders. With the small sample data, it probably would not be needed yet.

**Question 8** — What is the leftmost prefix rule for multi-column indexes? If you create `CREATE INDEX idx ON orders (customer_id, order_date)`, which of the following queries can use the index and why?

```sql
-- a)
SELECT * FROM orders WHERE customer_id = 5;
-- b)
SELECT * FROM orders WHERE order_date = '2026-01-15';
-- c)
SELECT * FROM orders WHERE customer_id = 5 AND order_date = '2026-01-15';
```

(See Theory Section 8)

> [!NOTE]
>
> The index starts with `customer_id`, so query a can use it, and query c can use it for both conditions. Query b cannot use this index to search by `order_date` alone because it does not include the leftmost column, `customer_id`.

---

## 3. EXPLAIN Reading Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Below are four `EXPLAIN ANALYZE` outputs. For each one, identify:

1. The scan type used
2. Estimated rows vs actual rows
3. Total execution time
4. Whether an index would help (and which column to index)

### Output A — Sequential Scan

```
Seq Scan on products  (cost=0.00..25.00 rows=5 width=72) (actual time=0.015..0.285 rows=3 loops=1)
  Filter: (category_id = 4)
  Rows Removed by Filter: 497
Planning Time: 0.065 ms
Execution Time: 0.312 ms
```

**Your analysis:**

- Scan type:
- Estimated vs actual rows:
- Execution time:
- Would an index help? Which one?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Output B — Index Scan

```
Index Scan using idx_orders_customer_id on orders  (cost=0.28..8.30 rows=1 width=48) (actual time=0.021..0.024 rows=4 loops=1)
  Index Cond: (customer_id = 12)
Planning Time: 0.082 ms
Execution Time: 0.041 ms
```

**Your analysis:**

- Scan type:
- Estimated vs actual rows:
- Execution time:
- Is the index effective here?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Output C — Bitmap Index Scan

```
Bitmap Heap Scan on order_items  (cost=4.32..35.76 rows=25 width=24) (actual time=0.045..0.098 rows=28 loops=1)
  Recheck Cond: (product_id = 15)
  Heap Blocks: exact=12
  ->  Bitmap Index Scan on idx_order_item_product_id  (cost=0.00..4.31 rows=25 width=0) (actual time=0.030..0.030 rows=28 loops=1)
        Index Cond: (product_id = 15)
Planning Time: 0.091 ms
Execution Time: 0.132 ms
```

**Your analysis:**

- Scan type:
- Estimated vs actual rows:
- Execution time:
- Why did PostgreSQL choose a Bitmap scan instead of a plain Index Scan?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Output D — Hash Join

```
Hash Join  (cost=33.50..75.82 rows=120 width=52) (actual time=0.452..1.205 rows=115 loops=1)
  Hash Cond: (oi.product_id = p.id)
  ->  Seq Scan on order_items oi  (cost=0.00..30.40 rows=2040 width=16) (actual time=0.008..0.195 rows=2040 loops=1)
  ->  Hash  (cost=20.00..20.00 rows=500 width=40) (actual time=0.410..0.411 rows=500 loops=1)
        Buckets: 1024  Batches: 1  Memory Usage: 45kB
        ->  Seq Scan on products p  (cost=0.00..20.00 rows=500 width=40) (actual time=0.005..0.180 rows=500 loops=1)
Planning Time: 0.215 ms
Execution Time: 1.340 ms
```

**Your analysis:**

- Scan type:
- Estimated vs actual rows:
- Execution time:
- Would adding an index improve this join? Which column?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

---

## 4. Index Design Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each query below, recommend the best index(es), explain why, and predict the scan type PostgreSQL will use after indexing.

**Query 1**

```sql
SELECT * FROM customers WHERE email = 'anna.korhonen@email.fi';
```

**Query 2**

```sql
SELECT * FROM orders WHERE customer_id = 5 ORDER BY order_date DESC;
```

**Query 3**

```sql
SELECT product_id, SUM(quantity) FROM order_items GROUP BY product_id;
```

**Query 4**

```sql
SELECT * FROM products WHERE price BETWEEN 50 AND 100;
```

**Query 5**

```sql
SELECT p.name, c.name AS category
FROM products p
JOIN product_categories pc ON pc.product_id = p.product_id
JOIN categories c ON c.category_id = pc.category_id
WHERE pc.category_id = 2;
```

**Query 6**

```sql
SELECT * FROM payments WHERE order_id = 88;
```

**Query 7**

```sql
SELECT * FROM orders WHERE status = 'shipped' AND order_date > '2026-06-01';
```

**Query 8**

```sql
SELECT customer_id, COUNT(*) FROM orders GROUP BY customer_id HAVING COUNT(*) > 5;
```

**Query 9**

```sql
SELECT oi.quantity, p.name
FROM order_items oi
JOIN products p ON p.id = oi.product_id
WHERE oi.order_id = 100;
```

**Query 10**

```sql
SELECT * FROM products WHERE name LIKE 'Trail%';
```

For each query, fill in:

| #   | Recommended Index | Why It Helps | Expected Scan Type |
| --- | ----------------- | ------------ | ------------------ |
| 1   |                   |              |                    |
| 2   |                   |              |                    |
| 3   |                   |              |                    |
| 4   |                   |              |                    |
| 5   |                   |              |                    |
| 6   |                   |              |                    |
| 7   |                   |              |                    |
| 8   |                   |              |                    |
| 9   |                   |              |                    |
| 10  |                   |              |                    |

---

## 5. Performance Comparison Exercise

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each query below, follow these steps:

1. Run `EXPLAIN ANALYZE` **without** an index — record the full output
2. Create the recommended index
3. Run `EXPLAIN ANALYZE` **again** — record the full output
4. Calculate the improvement factor (before time ÷ after time)
5. Write one sentence explaining why the improvement occurred (in the **Explanation** field under each comparison)

### Comparison 1

```sql
-- Query
SELECT * FROM order_items WHERE order_id = 55;

-- Index to create
CREATE INDEX idx_oi_order_id ON order_items (order_id);
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —      |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Comparison 2

```sql
-- Query
SELECT * FROM products p
JOIN product_categories pc ON pc.product_id = p.product_id
WHERE pc.category_id = 2 AND p.price < 80;

CREATE INDEX idx_product_categories_category_id ON product_categories (category_id);
CREATE INDEX idx_products_price ON products (price);

-- Index to create
CREATE INDEX idx_products_cat_price ON products (category_id, price);
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —      |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Comparison 3

```sql
-- Query
SELECT * FROM customers WHERE email = 'test.user@example.com';

-- Index to create
CREATE UNIQUE INDEX idx_customers_email ON customers (email);
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —      |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Comparison 4

```sql
-- Query
SELECT order_id, SUM(quantity * unit_price) AS total
FROM order_items
WHERE order_id IN (10, 20, 30, 40, 50)
GROUP BY order_id;

-- Index to create
CREATE INDEX idx_oi_order_id_covering ON order_items (order_id) INCLUDE (quantity, unit_price);
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —      |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Comparison 5

```sql
-- Query
SELECT * FROM orders WHERE status = 'pending';

-- Index to create (partial index)
CREATE INDEX idx_orders_pending ON orders (status) WHERE status = 'pending';
```

| Metric              | Before | After |
| ------------------- | ------ | ----- |
| Scan type           |        |       |
| Execution time (ms) |        |       |
| Rows examined       |        |       |
| Improvement factor  | —      |       |

**Explanation:**

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

---

## Submission Checklist

**Required**

- [ ] Baseline measurements for all 5 project queries
- [ ] All 5 indexes created and re-measured
- [ ] Comparison table filled with before/after data
- [ ] Recommendation paragraph written
- [ ] All 8 theory questions answered

**Recommended practice**

- [ ] All 4 EXPLAIN outputs analyzed
- [ ] Index recommendations for all 10 queries
- [ ] All 5 performance comparisons completed with explanations