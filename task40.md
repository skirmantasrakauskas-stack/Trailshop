# Week 40 — Exercises: SQL Fundamentals

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

## Exercise 1: TrailShop Project Task

This week you'll build the TrailShop database from scratch and practice manipulating data.

### Task 1.1: Create the Database

1. Open your PostgreSQL terminal (psql) or pgAdmin
2. Create a new database called `trailshop`
3. Connect to it

### Task 1.2: Create All Tables

Write and execute the CREATE TABLE statements for all six TrailShop tables in the correct order:

- categories
- customers
- products
- product_categories
- orders
- order_items

**Requirements:**

- Use appropriate data types for each column
- Include all constraints from the theory (NOT NULL, UNIQUE, CHECK, FOREIGN KEY, DEFAULT)
- Use SERIAL for primary keys
- Ensure foreign keys reference the correct parent tables

**Verify** by running `\dt` in psql to list all tables.

> [!NOTE]
> **_Your SQL_**
>
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

### Task 1.3: Insert Sample Data

Insert the following data:

**Categories** (at least 5):

- Footwear, Backpacks, Tents, Clothing, Accessories

> [!NOTE]
> ```sql
> INSERT INTO categories(name, description)
> VALUES
>     ('Footwear', 'Different footwear for different occasions'),
>     ('Backpacks', 'Backpacks that come in different sizes'),
>     ('Tents', 'Tents, small or large'),
>     ('Clothing', 'Stuff you wear'),
>     ('Accessories', 'For when you need something else');
> ```

**Customers** (at least 5):

- Use easy to write names with realistic email addresses

> [!NOTE]
> ```sql
> INSERT INTO customers(first_name,last_name,email)
> VALUES
>     ('Skirmantas','Rakauskas','skirmantas.rakauskas@gmail.com'),
>     ('Rokas','Burokas','pituhgaming3@gmail.com'),
>     ('Dalia','Kavaliauskiene','dalyte232@gmail.com'),
>     ('Pituhas','Pituhovicius','ne.rasykman@gmail.com'),
>     ('Viliovskaja', 'Malaciauskas','vilius.malaciauskas@gmail.com');
> ```

**Products** (at least 10):

- At least 2 products per category
- At least one product assigned to **two or more** categories
- Prices ranging from €20 to €500
- Various stock levels

> [!NOTE]
> ```sql
> INSERT INTO products(name, price, stock, description)
> VALUES
>     ('Hiking Boots', 120.00, 50, 'Durable boots for hiking'),
>     ('Running Shoes', 80.00, 100, 'Lightweight shoes for running'),
>     ('Travel Backpack', 150.00, 30, 'Spacious backpack for travel'),
>     ('Daypack', 60.00, 70, 'Compact backpack for daily use'),
>     ('Family Tent', 300.00, 20, 'Large tent suitable for families'),
>     ('Camping Tent', 200.00, 40, 'Tent for camping trips'),
>     ('Winter Jacket', 250.00, 25, 'Warm jacket for winter'),
>     ('T-Shirt', 25.00, 200, 'Casual t-shirt for everyday wear'),
>     ('Sunglasses', 50.00, 80, 'Stylish sunglasses for sunny days'),
>     ('Watch', 200.00, 15, 'Elegant watch for all occasions');
> ```

**Product categories:**

- Insert rows into `product_categories` so every sample product is linked to at least one category

> [!NOTE]
> ```sql
> INSERT INTO product_categories(product_id, category_id)
> VALUES
>     (1, 1), (2, 1),
>     (3, 2), (4, 2),
>     (5, 3), (6, 3),
>     (7, 4), (8, 4),
>     (9, 5), (10, 5);
> ```

**Orders** (at least 5):

- Different customers, different statuses

> [!NOTE]
> ```sql
> INSERT INTO orders(customer_id, status)
> VALUES
>     (1, 'shipped'),
>     (2, 'delivered'),
>     (3, 'pending'),
>     (4, 'shipped'),
>     (5, 'cancelled');
> ```

**Order Items** (at least 10):

- Multiple items in some orders, single items in others

> [!NOTE]
> ```sql
> INSERT INTO order_items(order_id, product_id, quantity, unit_price)
> VALUES
>     (1, 1, 2, 120.00),
>     (1, 3, 1, 150.00),
>     (2, 2, 1, 80.00),
>     (2, 4, 2, 60.00),
>     (3, 5, 1, 300.00),
>     (4, 6, 3, 200.00),
>     (4, 7, 1, 250.00),
>     (5, 8, 2, 25.00),
>     (5, 9, 1, 50.00),
>     (3, 10, 1, 200.00);
> ```

**Verify** each insert with `SELECT * FROM table_name;`

### Task 1.4: Practice UPDATE

> [!TIP]
> **Recommended practice.** Do Tasks 1.4–1.6. They are not required to finish the TrailShop project. They prepare you for the exams. Task 1.6 renames `stock` to `quantity_in_stock`. Later weeks still use `stock`, so after you practice the rename, change the column name back.
>
>
> **My note:** I'm going to ignore this because of time restrictions. I will finish it in a few days after the submission to prepare for the exam.
>
>
Perform the following updates and verify each one:

1. Increase the price of all products in the Footwear category by 10% (join through `product_categories`)
2. Change customer #3's email to a new address
3. Update the status of order #2 from 'shipped' to 'delivered'
4. Set the stock of 'HydroFlask 1L' to 85
5. Add a description to any product that currently has NULL in description

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your queries here
>
>
> ```

### Task 1.5: Practice DELETE

1. Delete the most recently created order (and observe what happens to its order_items if you used CASCADE)
2. Try to delete a product that appears in `order_items` — what error do you get?
3. Delete a category that has products linked through `product_categories`. The products should remain; only the link rows should disappear. Confirm this.
4. Delete a customer who has no orders

### Task 1.6: Practice ALTER TABLE

1. Add a column `phone VARCHAR(20)` to the customers table
2. Add a column `weight_grams INTEGER` to the products table
3. Add a CHECK constraint to ensure `weight_grams > 0` (allow NULL though — not all products have weight recorded yet)
4. Rename the `stock` column in products to `quantity_in_stock`

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your queries here
>
>
> ```

---

## Exercise 2: Theory Review Questions

Answer the following questions in your own words using the answer fields below:

1. What does SQL stand for, and why was the language designed to look like English?

> [!NOTE]
> SQL stands for Structured English Query Language. It was designed for readability, it intentionally looks like english sentences.

2. Explain the difference between DDL and DML. Give two example commands for each.

> [!NOTE]
> DDL defines the structure of the database, it creates, modifies and removes databases and etc., whereas DML works with the actual data inside of the tables in the databases. DDL - CREATE TABLE , DROP TABLE, DML - SELECT, INSERT INTO.

3. What is the difference between DCL and TCL? When would you use each?

> [!NOTE]
> DCL is more about permissions, defining who does what in the database. TCL is just a way to execute groups of commands at once where all of them have to succeed or none do. I mean i would use DCL when i want to set the permissions of employees for example, and would use TCL for when i want to test the possible changes of DML commands without serious data repercussions.

4. Why must you create tables in a specific order? What determines that order?

> [!NOTE]
> You have to create tables in a specific order because of dependency on other tables, starting with the tables that don't depend on any other ones first, then the ones that depend on others. It is mainly because some tables are redundant unless given data from other parent tables ( foreign key moment ), so creating a child table that serves no purpose doesn't make sense.

5. What is the difference between a column-level constraint and a table-level constraint? When _must_ you use a table-level constraint?

> [!NOTE]
> A column level constraint is applied to a single column and is declared right after the datatype of a column. A table level constraint is declared after all columns and is separated by commas. You must use a table level constraint when you have a composite primary or foreign key and when you have a check constraint referencing multiple columns.

6. Explain the difference between `DELETE FROM products;` and `TRUNCATE TABLE products;`. When would you prefer each?

> [!NOTE]
> DELETE is more specific, creates dead tuples and usually lower. Truncate just empties a table fast and is good during testing. I would prefer delete when i don't want to lose all the data, i'd use truncate during testing to empty out everything fast.

7. What does `ON DELETE CASCADE` do on a foreign key? Give a real-world scenario where it's appropriate and one where it would be dangerous.

> [!NOTE]
> The real world scenario is me doing this grueling homework, so i'd say Order_items would need on delete cascade for the order_id in order_items, so if an order is cancelled or deleted, the order items are deleted too. It would be dangerous to ON DELETE CASCADE the customer_id, because even if the customer is deleted from the database, the order info would be nice to have for legal and tax reasons.

8. Why should you store `unit_price` in the `order_items` table instead of just looking it up from the `products` table?

> [!NOTE]
> Because product prices can change, discounts, seasons, the economy, inflation, etc. and what matters is the factual amount paid for the product for audit reasons.

9. What is the difference between SERIAL and GENERATED ALWAYS AS IDENTITY? Which would you use in a new project and why?

> [!NOTE]
> Serial is a legacy data type lol, GENERATED ALWAYS AS IDENTITY prevents manual insert accidents.

10. Explain why `UPDATE products SET price = 9.99;` is dangerous. What steps should you take before running any UPDATE statement?

> [!NOTE]
> Because this would literally change every single product price to 9.99, its better to start with a TCL command, so you can rollback and prevent being the intern that affected 20,000 product prices without a rollback. To prevent this though you need to add a WHERE statement to really specify which product price you want to change.

---

## Exercise 3: SQL Writing Exercises (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.
>
>
> IM GOING TO IGNORE THIS BECAUSE OF TIME RESTRICTIONS, I WILL FINISH IT IN A FEW DAYS AFTER THE SUBMISSION TO PREPARE FOR THE EXAM.
>
>

Write the SQL statements for each task in the **Your SQL** fields below. Verify by running them when ready.

### 3.1 CREATE TABLE

Write a CREATE TABLE statement for a `suppliers` table with the following columns:

- supplier_id (auto-incrementing primary key)
- company_name (required, max 200 characters, must be unique)
- contact_name (max 150 characters)
- email (max 255 characters, required, unique)
- phone (max 20 characters)
- country (max 100 characters, required, default 'Finland')

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.2 CREATE TABLE with Foreign Key

Write a CREATE TABLE statement for a `product_reviews` table:

- review_id (auto-incrementing primary key)
- product_id (required, references products)
- customer_id (required, references customers)
- rating (required integer, must be between 1 and 5 inclusive)
- review_text (optional, unlimited length)
- created_at (required, defaults to current timestamp)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.3 INSERT — Single Row

Write an INSERT statement to add a new category called 'Electronics' with description 'GPS devices, solar chargers, and tech gear'.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.4 INSERT — Multiple Rows

Write a single INSERT statement that adds three new customers:

- Eero Lahtinen, eero.l@email.com
- Maria Salminen, maria.s@email.com
- Petri Kallio, petri.k@email.com

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.5 INSERT with RETURNING

Write an INSERT statement that adds a new product called 'NorthStar GPS' priced at €229.99 with stock of 12, then assign it to category 'Electronics' (assume `category_id = 6`) using `product_categories`. Return the product_id and created_at.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.6 UPDATE — Simple

Write an UPDATE statement that changes the email of the customer with customer_id = 2 to 'mikko.korhonen@newmail.com'.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.7 UPDATE — Expression

Write an UPDATE statement that reduces the stock of all products by 1 where the stock is currently greater than 0.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.8 UPDATE — Multiple Columns

Write an UPDATE statement that changes order #3 to status 'cancelled' and sets a (hypothetical) cancelled_at timestamp to the current time. (Assume you've already added a cancelled_at column.)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.9 DELETE — With Condition

Write a DELETE statement that removes all orders with status 'cancelled'.

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

### 3.10 ALTER TABLE

Write the ALTER TABLE statements to:
a) Add a `discount_percent NUMERIC(5,2) DEFAULT 0 CHECK (discount_percent >= 0 AND discount_percent <= 100)` column to products
b) Drop the `description` column from categories
c) Add a composite unique constraint on (customer_id, product_id) in the product_reviews table (preventing a customer from reviewing the same product twice)

> [!NOTE]
> **_Your SQL_**
>
> ```sql
> -- Write your query here
>
>
> ```

---

## Exercise 4: Error Diagnosis (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Each of the following SQL statements contains one or more errors. Identify the error(s) and write the corrected version.

### 4.1

```sql
CREATE TABLE warehouses
    warehouse_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    city VARCHAR(100
);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_

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
INSERT INTO products (name, price, stock)
VALUES ("Alpine Sleeping Bag", 89.99, 20);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_

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
CREATE TABLE shipments (
    shipment_id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(order_id)
    shipped_date DATE NOT NULL,
    carrier VARCHAR(100)
);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_

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
UPDATE products
SET price = price * 0.9
SET stock = stock + 10
WHERE product_id = 3;
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_

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
CREATE TABLE wishlists (
    wishlist_id SERIAL PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(customer_id),
    product_id INTEGER NOT NULL REFERENCES products(product_id),
    added_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (customer_id, product_id)
);
```

> [!NOTE]
> **_Error(s) Identified_**
>
> _(Describe what is wrong.)_

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

- [x] All 6 TrailShop tables created successfully
- [x] Sample data inserted (at least 5 categories, 5 customers, 10 products, product_categories links, 5 orders, 10 order items)
- [x] Theory review questions answered

**Recommended practice**

- [ ] UPDATE exercises completed and verified
- [ ] DELETE exercises completed and verified
- [ ] ALTER TABLE exercises completed, then `quantity_in_stock` renamed back to `stock`
- [ ] SQL writing exercises completed
- [ ] Error diagnosis completed with corrections