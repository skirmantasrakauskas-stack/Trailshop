# Week 41 — Exercises: Querying Data

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

## Exercise 1: TrailShop Project Task — Business Questions

Use the TrailShop database you created in Week 40. Write SQL queries to answer each business question below. Run each query and verify the results make sense.

> [!IMPORTANT]
> **_Tools to use_**
> You can use both the pgAdmin or psql (terminal) to test the queries

### Basic Queries (SELECT + WHERE)

1. List all products in the 'Footwear' category (show name, price, stock).

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT products.name, products.price, products.stock
   > FROM products
   > JOIN product_categories
   >     ON products.product_id = product_categories.product_id
   > JOIN categories
   >     ON product_categories.category_id = categories.category_id
   > WHERE categories.name = 'Footwear';
   > ```

2. Find all products priced between €50 and €150, sorted by price ascending.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT price FROM products;
   > WHERE price => 50 AND price =< 150
   > ORDER BY price;
   > ```

3. Show all customers whose last name starts with the letter 'M' or 'K'.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT last_name FROM customers
   > WHERE last_name LIKE 'M%' OR 'K%';
   > ```

4. List all orders with status 'pending' or 'shipped', sorted by order date (most recent first).

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT * FROM orders
   > WHERE status IN ('shipped','pending');
   >
   > ```

5. Find all products that have the word 'Pro' or 'pro' somewhere in their name.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT * FROM products
   > WHERE name LIKE '%Pro%';
   >
   > ```

### Aggregate Queries

6. What is the total number of products in the database?

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT COUNT(*) FROM products; 
> ```

7. What is the average price of all products? Round to 2 decimal places.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT ROUND(AVG(price),2) FROM products;
   > ```

8. Which is the most expensive product and which is the cheapest? Show both in one query.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT MIN(price), MAX(price) FROM products;
   > ```

9. How many orders does each customer have? Show customer name and order count, sorted by count descending.

   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > SELECT c.first_name || ' ' || c.last_name AS customer_name, COUNT(o.order_id) AS order_count
   > FROM customers c
   > LEFT JOIN orders o ON c.customer_id = o.customer_id
   > GROUP BY c.customer_id, c.first_name, c.last_name
   > ORDER BY order_count DESC;
   > ```

10. What is the total revenue (sum of quantity × unit_price from order_items) for each order status?

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> SELECT o.status, SUM(oi.quantity * oi.unit_price) AS total_revenue
> FROM orders o
> JOIN order_items oi ON o.order_id = oi.order_id
> GROUP BY o.status;
> ```

### JOIN Queries

11. List all products with their category names (join through `product_categories`). A product in two categories should appear twice. Sort by category name, then product name.

    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > SELECT c.name AS category_name, p.name AS product_name
    > FROM product_categories pc
    > JOIN categories c ON pc.category_id = c.category_id
    > JOIN products p ON pc.product_id = p.product_id
    > ORDER BY c.name, p.name;
    > ```

12. Show each order with the customer's full name, order date, and status.

    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > SELECT o.order_id,
    >        c.first_name || ' ' || c.last_name AS customer_name,
    >        o.order_date,
    >        o.status
    > FROM orders o
    > JOIN customers c ON o.customer_id = c.customer_id
    > ORDER BY o.order_id;
    > ```

13. Show a detailed breakdown of order #1: product name, quantity, unit price, and line total.

    > [!NOTE]
    >
    > ```sql
    > SELECT p.name AS product_name,
    >        oi.quantity,
    >        oi.unit_price,
    >        oi.quantity * oi.unit_price AS line_total
    > FROM order_items oi
    > JOIN products p ON oi.product_id = p.product_id
    > WHERE oi.order_id = 1;
    > ```

14. Find all customers who have NOT placed any orders. (Hint: use LEFT JOIN + IS NULL pattern.)

    > [!NOTE]
    >
    > ```sql
    > SELECT c.customer_id,
    >        c.first_name,
    >        c.last_name
    > FROM customers c
    > LEFT JOIN orders o ON c.customer_id = o.customer_id
    > WHERE o.order_id IS NULL
    > ORDER BY c.last_name, c.first_name;
    > ```

15. For each category, show the category name, number of products, average price, and total inventory value (price × stock summed). Only include categories with total inventory value greater than €500.
    > [!NOTE]
    >
    > ```sql
    > SELECT c.name AS category_name,
    >        COUNT(pc.product_id) AS number_of_products,
    >        ROUND(AVG(p.price), 2) AS average_price,
    >        ROUND(SUM(p.price * p.stock), 2) AS total_inventory_value
    > FROM categories c
    > JOIN product_categories pc ON c.category_id = pc.category_id
    > JOIN products p ON pc.product_id = p.product_id
    > GROUP BY c.category_id, c.name
    > HAVING SUM(p.price * p.stock) > 500
    > ORDER BY c.name;
    > ```

---

## Exercise 2: Theory Review Questions

Answer in your own words:

1. What is the logical execution order of a SQL query? Why does it matter?

> [!NOTE]
> FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT / OFFSET. It matters because it dictates how data is processed, what information is available at each stage and how errors are prevented.
>

2. What is the difference between WHERE and HAVING? Give an example of when you would use each.

> [!NOTE]
> WHERE filters individual rows before grouping and aggregation, while HAVING filters the grouped results after aggregation has been calculated, so WHERE is used to narrow down raw data and HAVING is used to narrow down summaries such as totals, averages, or counts. For example, in a sales table, you might use WHERE country = 'USA' to keep only US orders, then GROUP BY customer_id and use HAVING SUM(total_amount) > 1000 to show only customers whose total spend exceeds $1,000.

3. Explain the difference between COUNT(*), COUNT(column), and COUNT(DISTINCT column).

> [!NOTE]
> Count(*) counts all rows even where NULL appears, Count(column) counts all rows where that column is not null, Count(DISTINCT column) counts unique non-null values.

4. What is the difference between INNER JOIN and LEFT JOIN? When would you choose one over the other?

> [!NOTE]
> INNER JOIN is a JOIN where both values need to be the same to stay, and it finds intersections, LEFT JOIN is a JOIN more commonly used, where it joins all the data from the right table to the left, keeping the entirety of the left table and only the corresponding data from the right. I would use INNER JOIN when i need to see matching information, LEFT JOIN when i need to keep a list of names and only count stuff for it from the right table.

5. Why should you avoid SELECT \* in production code?

> [!NOTE]
> It makes code harder to understand, more data and load is transferred than needed, if there are columns added, the query will return different results.

6. What does DISTINCT do? On what level does it operate (columns or entire rows)?

> [!NOTE]
> DISTINCT removes all duplicate rows, where the entire row is the exact same as another one.

7. Can you use a column alias in a WHERE clause? Why or why not?

> [!NOTE]
> You can't use a column alias in a where clause, because of execution order.

> 8. Explain what happens when you GROUP BY a column and there's a column in SELECT that isn't aggregated and isn't in GROUP BY.

> [!NOTE]
> SELECT customer_id, order_id
> FROM orders
> GROUP BY customer_id;
> 
>It's not allowed because customer_id is grouped, order_id is not aggregated, order_id is not included in GROUP BY. The database doesn't know which order_id to show for each group, so it will throw an error.
---

## Exercise 3: Query Writing Exercises (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Write the SQL for each task. Use the TrailShop schema (categories, customers, products, product_categories, orders, order_items).

### Simple SELECT + WHERE

**3.1** — Select product names and prices for all products with stock less than 15.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.2** — Find all customers who registered (created_at) in 2026. Show first name, last name, and registration date.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.3** — Show all products that do NOT have a description (description IS NULL).

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### Multi-condition Filtering

**3.4** — Find products in category 1 OR category 2, priced above €100, with stock greater than 0. Sort by price descending.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.5** — Find orders that are either 'delivered' or placed by customer_id 1. Show order_id, customer_id, status.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### Aggregation

**3.6** — For each category, show the category name, minimum, maximum, and average product price. Round averages to 2 decimal places. Join through `product_categories`.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.7** — Count how many distinct customers have placed at least one order.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.8** — Find the total quantity of items sold across all orders.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### JOIN Queries

**3.9** — Show each product name alongside its category name. Include all products (even if somehow a category was deleted — use LEFT JOIN).

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.10** — List all orders showing: order_id, customer full name, order date, number of items in the order, and order total cost.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.11** — Show all products that have NEVER been ordered. (Hint: LEFT JOIN order_items, then IS NULL on order_item_id.)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### GROUP BY + HAVING

**3.12** — Show categories where the average product price exceeds €100. Display category name and average price.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

**3.13** — Find customers who have placed more than 1 order. Show their name and order count.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### Pagination

**3.14** — Write a paginated query that returns products 4 through 6 (page 2, page size 3), ordered by product_id.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### Complex

**3.15** — Write a "sales report" query that shows: category name, total units sold (from order_items), total revenue, and number of distinct products sold — for each category. Sort by revenue descending.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

---

## Exercise 4: Query Reading Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each query below, explain in **plain English** what it does and what the result would look like.

### 4.1

```sql
SELECT c.name, COUNT(p.product_id) AS num_products
FROM categories c
LEFT JOIN product_categories pc ON pc.category_id = c.category_id
LEFT JOIN products p ON p.product_id = pc.product_id
GROUP BY c.name
ORDER BY num_products DESC;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- Write the corrected statement here
>
>
> ```

### 4.2

```sql
SELECT first_name, last_name
FROM customers
WHERE customer_id NOT IN (
    SELECT DISTINCT customer_id FROM orders
);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- Write the corrected statement here
>
>
> ```

### 4.3

```sql
SELECT p.name, p.price, p.stock,
       p.price * p.stock AS inventory_value
FROM products p
WHERE p.stock > 0
ORDER BY inventory_value DESC
LIMIT 3;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- Write the corrected statement here
>
>
> ```

### 4.4

```sql
SELECT o.order_id,
       SUM(oi.quantity * oi.unit_price) AS order_total
FROM orders o
INNER JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.status <> 'cancelled'
GROUP BY o.order_id
HAVING SUM(oi.quantity * oi.unit_price) > 200
ORDER BY order_total DESC;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- Write the corrected statement here
>
>
> ```

### 4.5

```sql
SELECT c.first_name || ' ' || c.last_name AS customer,
       COUNT(DISTINCT o.order_id) AS num_orders,
       SUM(oi.quantity) AS total_items,
       ROUND(SUM(oi.quantity * oi.unit_price), 2) AS total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN order_items oi ON o.order_id = oi.order_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY total_spent DESC NULLS LAST;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_
>
> [!NOTE]
> **_Corrected SQL_**
>
> ```sql
> -- Write the corrected statement here
>
>
> ```

---

## Submission Checklist

**Required**

- [x] All 15 business questions answered with working SQL
- [x] Theory review questions answered in your own words
- [x] Results of the business questions verified by running them against your TrailShop database

**Recommended practice**

- [ ] All 15 query writing exercises completed
- [ ] All 5 query reading explanations written