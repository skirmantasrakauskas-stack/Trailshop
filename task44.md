# Week 44 — Advanced SQL: Exercises

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

---

## 1. TrailShop Project Task

Continue building the TrailShop database. This week, use advanced query techniques to extract meaningful insights.

### Task 1.1: Subqueries

Write the following queries using subqueries:

1. Find all products priced above the average price (use a scalar subquery).
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   >SELECT name, price
   >FROM products
   >WHERE price > (SELECT AVG(price) FROM products);
   >
   > ```
2. Find all customers who have placed more than 2 orders (use a subquery in WHERE with IN).
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   >SELECT * 
   >FROM customers
   >WHERE customer_id IN(
   >  SELECT customer_id 
   >  FROM orders
   >  GROUP BY customer_id
   >  HAVING COUNT(*) > 2
   >   );
   >
   >
   > ```
3. Show each product's name, price, and how much it differs from its category's average price (use a correlated subquery or derived table).
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   SELECT p.name, p.price
   FROM products p
   WHERE p.price > (
      SELECT AVG(p2.price)
      FROM products p2
      JOIN product_categories pc2 ON pc2.product_id = p2.product_id
      JOIN product_categories pc1 ON pc1.product_id = p.product_id
      AND pc1.category_id = pc2.category_id
   );
   > ```

### Task 1.2: EXISTS

1. Use `EXISTS` to find all customers who have ordered at least one product from the "Footwear" category.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   SELECT c.first_name ||' '|| c.last_name AS full_name FROM customers c
   WHERE EXISTS (
      SELECT 1
      FROM orders o
      JOIN order_items oi ON oi.order_id = o.order_id
      JOIN products p ON p.product_id = oi.product_id
      JOIN product_categories pc ON pc.product_id = p.product_id
      JOIN categories cat ON cat.category_id = pc.category_id
      WHERE o.customer_id = c.customer_id
      AND cat.name = 'Footwear'
   );
   >
   >
   > ```
2. Use `NOT EXISTS` to find all products that have never been ordered.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   SELECT product_id, name FROM products p
   WHERE NOT EXISTS (
      SELECT 1
      FROM order_items oi
      WHERE oi.product_id = p.product_id
   );
   > ```

### Task 1.3: CASE Expressions

1. Add a `price_tier` column to a product query that labels products as 'Budget' (< €25), 'Standard' (€25–€74.99), or 'Premium' (≥ €75).
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   SELECT price, name,
      CASE
         WHEN price < 25 THEN 'Budget'
         WHEN price >= 25 AND price < 74.99 THEN 'Standard'
         WHEN price >= 75 THEN 'Premium'
         ELSE 'Unknown'
      END AS price_tier
   FROM products;
   >
   > ```
2. Write a query that counts orders by status using CASE inside COUNT.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT
   >     COUNT(CASE WHEN status = 'pending' THEN 1 END) AS pending_orders,
   >     COUNT(CASE WHEN status = 'shipped' THEN 1 END) AS shipped_orders,
   >     COUNT(CASE WHEN status = 'delivered' THEN 1 END) AS delivered_orders,
   >     COUNT(CASE WHEN status = 'cancelled' THEN 1 END) AS cancelled_orders
   > FROM orders;
   > ```

### Task 1.4: Create Three Views

Create the following views in your TrailShop database:

1. `v_product_catalog` — Shows product name, category name, price, and stock status ('In Stock', 'Low Stock' for < 5 units, 'Out of Stock' for 0). Join through `product_categories`. A product in two categories may appear twice (once per category).
2. `v_customer_summary` — Shows each customer's full name, email, number of orders, and total spent.
3. `v_monthly_revenue` — Shows revenue grouped by month.

Verify each view works with a simple SELECT.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> CREATE VIEW v_product_catalog AS
> SELECT p.name AS product_name,
>        cat.name AS category_name,
>        p.price,
>        CASE
>            WHEN p.stock = 0 THEN 'Out of Stock'
>            WHEN p.stock < 5 THEN 'Low Stock'
>            ELSE 'In Stock'
>        END AS stock_status
> FROM products p
> JOIN product_categories pc ON pc.product_id = p.product_id
> JOIN categories cat ON cat.category_id = pc.category_id;
>
> CREATE VIEW v_customer_summary AS
> SELECT c.first_name || ' ' || c.last_name AS full_name,
>        c.email,
>        (SELECT COUNT(*)
>         FROM orders o
>         WHERE o.customer_id = c.customer_id) AS number_of_orders,
>        COALESCE(
>            (SELECT SUM(oi.quantity * oi.unit_price)
>             FROM orders o
>             JOIN order_items oi ON oi.order_id = o.order_id
>             WHERE o.customer_id = c.customer_id),
>            0
>        ) AS total_spent
> FROM customers c;
>
> CREATE VIEW v_monthly_revenue AS
> SELECT DATE_TRUNC('month', o.order_date) AS revenue_month,
>        SUM(oi.quantity * oi.unit_price) AS total_revenue
> FROM orders o
> JOIN order_items oi ON oi.order_id = o.order_id
> GROUP BY DATE_TRUNC('month', o.order_date);
>
> SELECT * FROM v_product_catalog;
> SELECT * FROM v_customer_summary;
> SELECT * FROM v_monthly_revenue;
> ```

### Task 1.5: UNION

1. Write a UNION query that combines product names from two different categories into one list (without duplicates).
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT p.name AS product_name
   > FROM products p
   > JOIN product_categories pc ON pc.product_id = p.product_id
   > JOIN categories c ON c.category_id = pc.category_id
   > WHERE c.name = 'Footwear'
   > UNION
   > SELECT p.name AS product_name
   > FROM products p
   > JOIN product_categories pc ON pc.product_id = p.product_id
   > JOIN categories c ON c.category_id = pc.category_id
   > WHERE c.name = 'Backpacks';
   > ```
2. Write a query using EXCEPT to find categories that have products but no orders.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT c.name AS category_name
   > FROM categories c
   > JOIN product_categories pc ON pc.category_id = c.category_id
   > EXCEPT
   > SELECT c.name AS category_name
   > FROM categories c
   > JOIN product_categories pc ON pc.category_id = c.category_id
   > JOIN order_items oi ON oi.product_id = pc.product_id;
   > ```

### Task 1.6: CTE

1. Write a CTE that calculates total revenue per category, then use it to find categories with above-average revenue.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > WITH category_revenue AS (
   >     SELECT c.category_id,
   >            c.name AS category_name,
   >            COALESCE(SUM(oi.quantity * oi.unit_price), 0) AS total_revenue
   >     FROM categories c
   >     LEFT JOIN product_categories pc ON pc.category_id = c.category_id
   >     LEFT JOIN order_items oi ON oi.product_id = pc.product_id
   >     GROUP BY c.category_id, c.name
   > )
   > SELECT category_name, total_revenue
   > FROM category_revenue
   > WHERE total_revenue > (
   >     SELECT AVG(total_revenue)
   >     FROM category_revenue
   > );
   > ```
2. Write a query with two CTEs: one for customer order counts, another for average order count, then find "top customers" (above average).
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > WITH customer_order_counts AS (
   >     SELECT c.customer_id,
   >            c.first_name,
   >            c.last_name,
   >            COUNT(o.order_id) AS order_count
   >     FROM customers c
   >     LEFT JOIN orders o ON o.customer_id = c.customer_id
   >     GROUP BY c.customer_id, c.first_name, c.last_name
   > ),
   > average_order_count AS (
   >     SELECT AVG(order_count) AS average_orders
   >     FROM customer_order_counts
   > )
   > SELECT coc.customer_id, coc.first_name, coc.last_name, coc.order_count
   > FROM customer_order_counts coc
   > CROSS JOIN average_order_count aoc
   > WHERE coc.order_count > aoc.average_orders;
   > ```

---

## 2. Theory Review Questions

Answer these in your own words (2–4 sentences each):

1. What is the difference between an INNER JOIN and a LEFT JOIN? When would you choose LEFT JOIN?

> [!NOTE]
> An INNER JOIN returns only rows that have matching values in both tables. A LEFT JOIN returns values all rows from the left table and also matching rows from the right table, where if no value matches from the right table, a null value gets filled in.

2. Explain what a correlated subquery is. Why can it be slower than an uncorrelated subquery?

> [!NOTE]
> A correlated subquery is basically just a query inside a query, that depends on the outer query for information. It might be slower because it basically works like a for loop.

3. What is the difference between `IN` and `EXISTS`? When might you prefer EXISTS?

> [!NOTE]
> IN basically just checks if the pool of values given matches something. EXISTS checks whether a subquery returns atleast one row.

4. What rules must two SELECT statements follow to be combined with UNION?

> [!NOTE]
> They must return the same number of columns and have compatible data types in the according columns.

5. Explain the difference between UNION and UNION ALL. When would you use each?

> [!NOTE]
> UNION removes duplicates, UNION ALL keeps duplicates. I would use UNION when i dont want duplicates and i would use UNION ALL when i want duplicates.

6. What is a view? Name two advantages of using views.

> [!NOTE]
> A view is a named query inside of the database, which runs an underlying select when using it and returns the information.
>
> 7. What does `WITH CHECK OPTION` do on an updatable view?

> [!NOTE]
> WITH CHECK OPTION prevents users from inserting or updating rows through the view that would not be visible in the view itself. It ensures that any modifications made through the view adhere to the view's defining query conditions.

8. How does a CTE differ from a derived table (subquery in FROM)? What's the main advantage of CTEs?

> [!NOTE]
> A CTE  is defined using the WITH clause and can be referenced multiple times within the same query, making it more readable and maintainable. A derived table is a subquery in the FROM clause that can only be used once in that specific query. The main advantage of CTEs is their ability to improve query organization and readability, especially for complex queries.

---

## 3. Query Writing Exercises (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Write SQL queries for the following (use the TrailShop schema):

1. Find the most expensive product in each category using a correlated subquery.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >
   >
   > ```
2. List all customers along with their most recent order date. Include customers who have never ordered (show NULL).
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >
   >
   > ```
3. Find categories where the total revenue exceeds €500 (use a subquery or CTE).
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >
   >
   > ```
4. Write a query that shows each order's total and whether it's 'Small' (< €50), 'Medium' (€50–€149.99), or 'Large' (≥ €150).
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >
   >
   > ```
5. Create a view called `v_never_ordered` that shows products that have never appeared in an order_item.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >
   >
   > ```
6. Use EXCEPT to find products that exist in the products table but have never been ordered.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >
   >
   > ```
7. Write a CTE that ranks categories by number of products, then selects only the top 3.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >
   >
   > ```
8. Use `NOT EXISTS` to find categories that have no products.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >
   >
   > ```
9. Write a query using CASE that creates a "customer loyalty" label: 'New' (1 order), 'Regular' (2–5 orders), 'VIP' (6+ orders).
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >
   >
   > ```
10. Use GREATEST to find the higher value between each product's current price and the average price of all products in its category.
    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > -- Write your query here
    >
    >
    > ```
11. Write a query using COALESCE that shows order shipping dates, substituting 'Not yet shipped' for NULL values.
    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > -- Write your query here
    >
    >
    > ```
12. Use a scalar subquery in the SELECT clause to show each product alongside the total number of times it has been ordered.
    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > -- Write your query here
    >
    >
    > ```

---

## 4. Query Comparison Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each pair of queries below, both produce the same result. Explain which approach you'd prefer and why (consider readability, performance, and maintainability).

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Pair A: Finding customers with orders

**Version 1 — Subquery with IN:**

```sql
SELECT first_name, last_name
FROM customers
WHERE customer_id IN (SELECT customer_id FROM orders);
```

**Version 2 — JOIN:**

```sql
SELECT DISTINCT c.first_name, c.last_name
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id;
```

### Pair B: Finding products never ordered

**Version 1 — LEFT JOIN + IS NULL:**

```sql
SELECT p.product_name
FROM products p
LEFT JOIN order_items oi ON oi.product_id = p.product_id
WHERE oi.order_item_id IS NULL;
```

**Version 2 — NOT EXISTS:**

```sql
SELECT p.product_name
FROM products p
WHERE NOT EXISTS (
    SELECT 1 FROM order_items oi WHERE oi.product_id = p.product_id
);
```

### Pair C: Customer total spending

**Version 1 — Derived table (subquery in FROM):**

```sql
SELECT sub.full_name, sub.total_spent
FROM (
    SELECT c.first_name || ' ' || c.last_name AS full_name,
           SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM customers c
    JOIN orders o ON o.customer_id = c.customer_id
    JOIN order_items oi ON oi.order_id = o.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
) sub
WHERE sub.total_spent > 200;
```

**Version 2 — CTE:**

```sql
WITH customer_totals AS (
    SELECT c.first_name || ' ' || c.last_name AS full_name,
           SUM(oi.quantity * oi.unit_price) AS total_spent
    FROM customers c
    JOIN orders o ON o.customer_id = c.customer_id
    JOIN order_items oi ON oi.order_id = o.order_id
    GROUP BY c.customer_id, c.first_name, c.last_name
)
SELECT full_name, total_spent
FROM customer_totals
WHERE total_spent > 200;
```

For each pair, discuss: readability, performance (in PostgreSQL), and when you'd choose one over the other.

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

---

## Submission Checklist

**Required**

- [x] TrailShop queries written for subqueries, EXISTS, and CASE expressions
- [x] Three views created (`v_product_catalog`, `v_customer_summary`, `v_monthly_revenue`)
- [x] UNION/EXCEPT and CTE queries written
- [x] Theory review questions answered

**Recommended practice**

- [ ] Query writing exercises completed
- [ ] Query comparison exercise completed