# Week 45 — Normalization: Exercises

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

---

## 1. TrailShop Project Task

### Task 1.1: Analyze a Denormalized Table

Consider this denormalized table that TrailShop's old system used:

```
supplier_products(
    supplier_id, supplier_name, supplier_email, supplier_city,
    product_id, product_name, category_id, category_name,
    supply_price, lead_time_days
)
```

Sample data:

| supplier_id | supplier_name | supplier_email      | supplier_city | product_id | product_name    | category_id | category_name | supply_price | lead_time_days |
| ----------- | ------------- | ------------------- | ------------- | ---------- | --------------- | ----------- | ------------- | ------------ | -------------- |
| 1           | NordGear Oy   | info@nordgear.fi    | Helsinki      | 101        | Trail Runner X  | 10          | Footwear      | 45.00        | 7              |
| 1           | NordGear Oy   | info@nordgear.fi    | Helsinki      | 102        | Summit Boot     | 10          | Footwear      | 62.00        | 7              |
| 2           | OutdoorPro AB | sales@outdoorpro.se | Stockholm     | 201        | Rain Jacket Pro | 30          | Clothing      | 55.00        | 14             |
| 2           | OutdoorPro AB | sales@outdoorpro.se | Stockholm     | 101        | Trail Runner X  | 10          | Footwear      | 43.00        | 10             |

**Tasks:**

1. Identify the primary key of this table.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > It seems that this is a junction table and a weak entity, there is a composite key made of supplier_id and product_id, since the table seems to be dependent on these two values to make a tuple unique.
   > ```
2. Identify any insertion, update, and deletion anomalies (give specific examples).
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > Insertion anomalies would be supplier name, email, city, category name, product name. It does not need all of this information, because everytime you would need to insert it all again, its just redundant. Updating the emails or cities or category names would lead to inconsistent data if even one value is incorrectly written or forgotten. Deletion of a tuple would also lead to deletion of the supply price, if rain jacket pro was deleted, so would the Clothing_name. 
   >
   >
   > ```
3. Normalize this table to 3NF. Show each step (1NF, then 2NF, then 3NF) with the resulting tables.
    > [!NOTE]
    > **_Your SQL_**
    >
    > ```sql
    > CREATE TABLE suppliers(
    >   supplier_id SERIAL PRIMARY KEY,
    >   supplier_name VARCHAR(255) NOT NULL UNIQUE,
    >   supplier_email VARCHAR(255) NOT NULL UNIQUE,
    >   supplier_city VARCHAR(255) NOT NULL
    >);
    > CREATE TABLE products(
    >   product_id SERIAL PRIMARY KEY,
    >   product_name VARCHAR(255) NOT NULL UNIQUE
    >);
    > CREATE TABLE categories(
    >   category_id SERIAL PRIMARY KEY,
    >   category_name VARCHAR(255) NOT NULL UNIQUE
    >);
    > CREATE TABLE productcategories(
    >   category_id SERIAL REFERENCES categories(category_id),
    >   product_id SERIAL REFERENCES products(product_id),
    >   PRIMARY KEY(category_id, product_id)
    >);
    > CREATE TABLE supplier_products(
    >   supplier_id SERIAL NOT NULL REFERENCES suppliers(supplier_id),
    >   product_id SERIAL NOT NULL REFERENCES products(product_id),
    >   supply_price NUMERIC(10,2) NOT NULL CHECK(supply_price >=0),
    >   PRIMARY KEY(supplier_id, product_id);
    >);
   >
   > ```

### Task 1.2: Review Your Existing TrailShop Schema

1. For each table, confirm it is in 3NF by verifying:
   - No partial dependencies (check tables with composite keys or surrogate keys)
   - No transitive dependencies
   > [!NOTE]
   > **_Your SQL_**
   > There are no repeating lists, there are no partial dependencies, all partial dependencies are in separate tables. 
> ```sql
> CREATE TABLE categories (
>     category_id SERIAL PRIMARY KEY,
>     name        VARCHAR(255) NOT NULL UNIQUE,
>     description TEXT
> );
>
> CREATE TABLE customers (
>     customer_id SERIAL PRIMARY KEY,
>     first_name  VARCHAR(50) NOT NULL,
>     last_name   VARCHAR(50) NOT NULL,
>     email       VARCHAR(50) NOT NULL UNIQUE,
>     address     VARCHAR(50),
>     created_at  TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
> );
>
> CREATE TABLE products (
>     product_id  SERIAL PRIMARY KEY,
>     name        VARCHAR(50) NOT NULL,
>     description TEXT,
>     price       NUMERIC(10,2) NOT NULL CHECK(price > 0),
>     stock       INTEGER DEFAULT 0 CHECK(stock >= 0),
>     created_at  TIMESTAMPTZ
> );
>
> CREATE TABLE product_categories (
>     product_id  INTEGER NOT NULL
>                 REFERENCES products(product_id)
>                 ON DELETE CASCADE,
>     category_id INTEGER NOT NULL
>                 REFERENCES categories(category_id)
>                 ON DELETE CASCADE,
>     PRIMARY KEY (product_id, category_id)
> );
>
> CREATE TABLE orders (
>     order_id    SERIAL PRIMARY KEY,
>     customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
>     order_date  TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
>     status      VARCHAR(255) NOT NULL DEFAULT 'pending' CHECK(status IN ('pending', 'shipped','delivered','cancelled'))
> );
>
> CREATE TABLE order_items (
>     order_item_id   SERIAL PRIMARY KEY,
>     order_id    INTEGER NOT NULL REFERENCES orders(order_id) ON DELETE CASCADE,
>     product_id  INTEGER NOT NULL REFERENCES products(product_id),
>     quantity    INTEGER NOT NULL CHECK(quantity > 0),
>     unit_price  NUMERIC(10,2) NOT NULL CHECK(unit_price > 0)
> );
> ```
2. Explain why `unit_price` in `order_items` is NOT a normalization violation.
   > [!NOTE]
   > It is because unit_price can change over time, therefore it is more logical to save the price data only related to that specific order at that specific time due to real life economical situations that are subject to change.
   >

### Task 1.3: Add a Suppliers Table (Optional Extension)

You may skip this task. Suppliers are not part of the TrailShop project schema.

If TrailShop wanted to track suppliers:

1. Design a normalized schema that includes suppliers. Consider:
   - Each product can have multiple suppliers
   - Each supplier offers different prices and lead times
   - Each supplier has contact info (name, email, city)
2. Write the `CREATE TABLE` statements.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >
   >
   > ```
3. Verify your design is in 3NF.

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

---

## 2. Theory Review Questions

Answer in your own words (2–4 sentences each):

1. What is normalization, and what problem does it solve?

> [!NOTE]
> Normalization is decomposition - splitting a poorly structured table into multiple well-structured tables. It solves several database issues, of which is eliminating redundancy, preventing anomalies, preserving data integrity, solving readability issues and granting a clear view of the database contents.

2. Explain the difference between an insertion anomaly and a deletion anomaly, using an example.

> [!NOTE]
> An insertion anomaly basically means you can't create a record without creating another separate record, for example, you can't insert a customer without inserting an order, in this case it is a problem, because a customer can exist without placing orders. A deletion anomaly is a problem where deleting a record can delete an unrelated important attribute completely, such as deleting an order can accidentally remove for example a category or customer name if there is only one record of that category or customer name in the entire poorly structured table.

3. What is the difference between a partial dependency and a transitive dependency?

> [!NOTE]
> A partial dependency is different from a transitive dependency because of the fact that partial dependencies only happen in tables where there is a composite key and the non key attribute depends on only part of the composite key. A transitive dependency is where a non key attribute depends on another non key attribute, such as products table containing category_name, which depends on category_id, which depends on product_id, its almost like a daisy-chain, its a problem because you have to update every single row with that information, otherwise there will be data inconsistency.

4. State the "informal rule" for 3NF. Explain what each part means.

> [!NOTE]
> "Every non-key attribute must depend on the key, the whole key, and nothing but the key - so help me Codd". The table must have a primary key (1NF), every non key column depends on the entire key (2NF), there must be no transitive dependencies(3NF).

5. Why can partial dependencies only occur in tables with composite primary keys?

> [!NOTE]
> Because the meaning in itself means partial of something, the only way it can be partial ( part of something that is more than one) is in a composite key, which can have two or more keys as the primary key. 

6. What is BCNF, and how does it differ from 3NF? In what rare case can a table be in 3NF but not BCNF?

> [!NOTE]
> BCNF is a stricter version of 3NF, it is basically the same as 3NF but with a stricter rule, that every determinant must be a candidate key. A table can be in 3NF but not BCNF if there is a non key attribute that determines another non key attribute, which is a violation of BCNF.

7. Give two legitimate reasons to denormalize a database.

> [!NOTE]
> 1. To improve query performance by reducing the number of joins required.
> 2. To accommodate reporting and analysis needs that may not fit well with a fully normalized structure.

---

## 3. Normalization Exercise (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Normalize each table to 3NF. Show your work for each step.

### Table 1: event_registrations

| event_id | event_name     | event_date | venue_id | venue_name      | venue_city | attendee_id | attendee_name | ticket_type | ticket_price |
| -------- | -------------- | ---------- | -------- | --------------- | ---------- | ----------- | ------------- | ----------- | ------------ |
| E1       | Trail Run 2025 | 2025-06-15 | V1       | City Park Arena | Helsinki   | A1          | Matti Aho     | VIP         | 75.00        |
| E1       | Trail Run 2025 | 2025-06-15 | V1       | City Park Arena | Helsinki   | A2          | Sara Lund     | Standard    | 35.00        |
| E2       | Mountain Fest  | 2025-07-20 | V2       | Summit Center   | Tampere    | A1          | Matti Aho     | Standard    | 40.00        |

**Business rules:**

- Each event has one name, date, and venue.
- Each venue has one name and city.
- Each attendee has one name.
- Ticket price depends on the event and ticket type combination.

**Steps:**

1. Identify the PK
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >
   >
   > ```
2. Check 1NF, 2NF, 3NF
3. Decompose as needed
4. Write final CREATE TABLE statements
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   > -- Write your query here
   >
   >
   > ```

### Table 2: store_inventory

| store_id | store_name         | store_city | product_id | product_name    | category | quantity | reorder_level | supplier_name |
| -------- | ------------------ | ---------- | ---------- | --------------- | -------- | -------- | ------------- | ------------- |
| S1       | TrailShop Helsinki | Helsinki   | 101        | Trail Runner X  | Footwear | 50       | 10            | NordGear Oy   |
| S1       | TrailShop Helsinki | Helsinki   | 201        | Rain Jacket Pro | Clothing | 20       | 5             | OutdoorPro AB |
| S2       | TrailShop Tampere  | Tampere    | 101        | Trail Runner X  | Footwear | 30       | 8             | NordGear Oy   |

**Business rules:**

- Each store has one name and city.
- Each product has one name, category, and supplier.
- Quantity and reorder_level depend on the specific store-product combination.

**Steps:** Same as above.

---

## 4. True/False

For each statement, write TRUE or FALSE and briefly explain why.

1. A table with a single-column primary key can violate 2NF.
   > FALSE
   >A table with a single-column primary key cannot violate 2NF because there are no partial dependencies to consider.

2. If a table is in 3NF, it is automatically in 2NF.
   > TRUE
   > 3NF is a stricter form of normalization than 2NF, so any table that is in 3NF must also be in 2NF.

3. Normalization always improves query performance.
   > FALSE
   > While normalization can improve data integrity and reduce redundancy, it can sometimes lead to more complex queries with multiple joins, which might degrade performance.

4. Storing a customer's current address in the orders table is always a normalization violation.
   > TRUE
   > This violates the principle of normalization by duplicating data that should be stored in a separate customer table.

5. BCNF is stricter than 3NF — every table in BCNF is also in 3NF.
   > TRUE
   > BCNF (Boyce-Codd Normal Form) is indeed stricter than 3NF (Third Normal Form), and any table that satisfies BCNF will automatically satisfy 3NF.

6. Denormalization should be done before normalization to save time.
   > FALSE
   > Denormalization is typically done after normalization when performance optimization is needed, not before it.

---

## Submission Checklist

**Required**

- [x] Existing TrailShop schema reviewed and confirmed in 3NF (Task 1.2)
- [x] Theory review questions answered
- [x] True/False questions answered with a short explanation

**Recommended practice**

- [ ] Denormalized `supplier_products` table analyzed and normalized (Task 1.1)
- [ ] Normalization exercise completed for both tables

**Optional extension**

- [ ] Suppliers schema designed (Task 1.3) — you may skip this