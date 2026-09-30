# Week 39 — Logical Database Design: Exercises

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 39 Theory material. Refer to the theory sections indicated in brackets when you need help.

---

## Exercise 1: TrailShop Project Task — Build the Schema

**Goal:** Convert the TrailShop ER diagram (from Week 38) into a complete PostgreSQL relational schema.

### Instructions

Write `CREATE TABLE` statements for all six TrailShop tables:

1. `categories`
2. `customers`
3. `products`
4. `product_categories`
5. `orders`
6. `order_items`

### Requirements

For each table, you must:

- Choose appropriate PostgreSQL data types for every column (justify at least 3 choices in writing)
- Define primary keys (surrogate or composite as appropriate)
- Define foreign keys with explicit `ON DELETE` and `ON UPDATE` actions (justify each choice)
- Add `NOT NULL`, `UNIQUE`, `CHECK`, and `DEFAULT` constraints where appropriate
- Create tables in the correct dependency order
- Follow the naming conventions from Theory Section 8

### Deliverables

1. A single `.sql` file with all six `CREATE TABLE` statements (executable in PostgreSQL)
2. A short justification for data types, FK actions and design decisions:
   - Justification for 3 data type choices (e.g., why `NUMERIC(10,2)` for price instead of `REAL`)
   - Justification for each FK action choice (e.g., why CASCADE on `order_items.order_id`)
   - One design decision you made that wasn't specified in the requirements (e.g., whether shipping address is optional)

### Bonus Challenge

After creating the tables, insert sample data:
- At least 5 categories
- At least 8 products (across at least 3 categories)
- At least one product assigned to **two or more** categories via `product_categories`
- At least 3 customers
- At least 4 orders (across at least 2 customers)
- At least 10 order items

Verify that your constraints work by attempting at least 2 invalid inserts and showing the error messages.

> [!NOTE]
> ```sql
CREATE TABLE categories (
    category_id SERIAL PRIMARY KEY,
    category_name VARCHAR(50) NOT NULL UNIQUE,
    description TEXT
);


CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(254) NOT NULL UNIQUE,
    phone VARCHAR(20),
    street VARCHAR(100) NOT NULL,
    city VARCHAR(50) NOT NULL,
    postal_code VARCHAR(10) NOT NULL,
    country VARCHAR(50) NOT NULL DEFAULT 'Finland',
    registered_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);


CREATE TABLE products (
    product_id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price NUMERIC(10,2) NOT NULL
        CONSTRAINT products_price_positive CHECK (price > 0),
    weight_kg NUMERIC(6,2)
        CONSTRAINT products_weight_positive CHECK (weight_kg > 0),
    stock_quantity INTEGER NOT NULL DEFAULT 0
        CONSTRAINT products_stock_non_negative CHECK (stock_quantity >= 0),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);


CREATE TABLE product_categories (
    product_id INTEGER NOT NULL
        REFERENCES products(product_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE,

    category_id INTEGER NOT NULL
        REFERENCES categories(category_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE,

    PRIMARY KEY (product_id, category_id)
);


CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,

    customer_id INTEGER NOT NULL
        REFERENCES customers(customer_id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE,

    order_date TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    status VARCHAR(20) NOT NULL DEFAULT 'pending'
        CHECK (status IN (
            'pending',
            'processing',
            'shipped',
            'delivered',
            'cancelled'
        )),

    shipping_street VARCHAR(100),
    shipping_city VARCHAR(50),
    shipping_postal_code VARCHAR(10),
    shipping_country VARCHAR(50)
);


CREATE TABLE order_items (
    order_id INTEGER NOT NULL
        REFERENCES orders(order_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE,

    product_id INTEGER NOT NULL
        REFERENCES products(product_id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE,

    quantity INTEGER NOT NULL
        CHECK (quantity > 0),

    unit_price NUMERIC(10,2) NOT NULL
        CHECK (unit_price > 0),

    PRIMARY KEY (order_id, product_id)
);







--data inserts
INSERT INTO categories (category_name, description) VALUES
    ('Trail Navigation', 'Maps and tools for finding the route.'),
    ('Camp Kitchen', 'Equipment for preparing food outdoors.'),
    ('Climbing Equipment', 'Gear for climbing and scrambling.'),
    ('Rainwear', 'Waterproof clothing and protection.'),
    ('Hydration', 'Bottles and water treatment equipment.');

INSERT INTO customers
    (first_name, last_name, email, phone, street, city, postal_code, country)
VALUES
    ('Elina', 'Makinen', 'elina.makinen@example.com', '+358401112223', '8 Birch Way', 'Oulu', '90100', 'Finland'),
    ('Timo', 'Aalto', 'timo.aalto@example.com', '+358405556667', '14 River Road', 'Lahti', '15110', 'Finland'),
    ('Sari', 'Niemi', 'sari.niemi@example.com', NULL, '6 Hill Street', 'Kuopio', '70100', 'Finland');

INSERT INTO products (name, description, price, weight_kg, stock_quantity) VALUES
    ('Surveyor Compass', 'Compact compass with a clear sighting mirror.', 34.90, 0.12, 32),
    ('Pocket Camp Stove', 'Small gas stove for backcountry cooking.', 79.95, 0.41, 14),
    ('Granite Climbing Helmet', 'Adjustable helmet for rock routes.', 64.50, 0.36, 22),
    ('Northwind Shell', 'Lightweight waterproof hiking jacket.', 189.00, 0.68, 17),
    ('Clearflow Filter Flask', 'Bottle with a replaceable water filter.', 42.75, 0.29, 45),
    ('Summit Chalk Pouch', 'Drawstring chalk pouch for climbing.', 18.50, 0.11, 30),
    ('Foldout Map Case', 'Water-resistant case for trail maps.', 21.00, 0.09, 26),
    ('Three-Piece Cook Set', 'Nested cookware for a small camp group.', 58.25, 0.72, 11);

INSERT INTO product_categories (product_id, category_id) VALUES
    (1, 1), (1, 5),
    (2, 2), (2, 5),
    (3, 3), (3, 1),
    (4, 4), (4, 1),
    (5, 5), (5, 1),
    (6, 3),
    (7, 1), (7, 4),
    (8, 2), (8, 5);

INSERT INTO orders
    (customer_id, status, shipping_street, shipping_city, shipping_postal_code, shipping_country)
VALUES
    (1, 'processing', '8 Birch Way', 'Oulu', '90100', 'Finland'),
    (2, 'shipped', '14 River Road', 'Lahti', '15110', 'Finland'),
    (3, 'pending', '6 Hill Street', 'Kuopio', '70100', 'Finland'),
    (1, 'delivered', '8 Birch Way', 'Oulu', '90100', 'Finland');

INSERT INTO order_items (order_id, product_id, quantity, unit_price) VALUES
    (1, 1, 1, 34.90),
    (1, 5, 2, 42.75),
    (1, 7, 1, 21.00),
    (2, 2, 1, 79.95),
    (2, 8, 1, 58.25),
    (2, 5, 1, 42.75),
    (3, 3, 1, 64.50),
    (3, 6, 2, 18.50),
    (4, 4, 1, 189.00),
    (4, 1, 1, 34.90),
    (4, 7, 1, 21.00);
> ```

> [!NOTE]
> These two inserts should fail when run separately after the valid inserts:
>
> ```sql
> INSERT INTO products (name, price)
> VALUES ('Biking equipment', -10.00);
> ```
> This should fail because the price has to be greater than zero.
>
> ```sql
> INSERT INTO categories (category_name)
> VALUES ('Trail Navigation');
> ```
> This should fail because the category name is already in use.

> Both inserts failed ( With a bit of issues on my part :D, i got it fixed in the end though.)
![Screenshot](https://i.ibb.co/cKgF694W/Windows-Terminal-v-TRC4fa-DAr.png)


> [!NOTE]
> I used NUMERIC(10,2) for prices because it stores money exactly, while REAL can have rounding problems. I used TIMESTAMPTZ for dates because orders and registrations should have a clear time, even across time zones. INTEGER works for IDs, stock, and quantities because those are whole numbers. I used TEXT for descriptions since they can be different lengths.
>
> In product_categories, I used ON DELETE CASCADE for both foreign keys, so the link gets deleted if its product or category is deleted. I used ON UPDATE CASCADE too, so the IDs stay connected if they change. For orders.customer_id, I used ON DELETE RESTRICT so a customer with order history can't be deleted, and ON UPDATE CASCADE so their ID changes in the orders too. For order_items.order_id, I used ON DELETE CASCADE because the items shouldn't stay if the order is deleted, and ON UPDATE CASCADE so the order ID stays updated. For order_items.product_id, I used ON DELETE RESTRICT so a product can't be deleted if it's part of an order, and ON UPDATE CASCADE so the item still points to the product if its ID changes.
>
> One extra decision I made is to save the price in order_items instead of only using the current price from products. That way, if a product's price changes later, old orders still show what the customer actually paid.
>

## Exercise 2: Theory Review Questions

Answer each question in 2–4 sentences. Reference the relevant theory section.

1. List the seven phases of the database development lifecycle in order. Which phase is this week's focus? *(Section 1)*

> [!NOTE]
Requirements gathering, Conceptual design, Logical design, Physical design, Implementation, Testing, Maintenance. Logical design is this week's focus.
>

2. Explain the transformation rule for mapping a 1:N relationship to the relational model. Why is the foreign key placed on the "many" side? *(Section 3.2)*

> [!NOTE]
It's placed on the many side because each order belongs to a single customer. If it was stored in the Customer side, several order Id's would be needed per customer row, which would violate atomicity.
>

3. What is a junction table? When is it needed? Give an example not from TrailShop. *(Section 3.3)*

> [!NOTE]
A junction table is a linking table used to represent a many-to-many (M:N) relationship. It is needed when one row in one table can relate to many rows in another table, and vice versa, because storing multiple values in a single row would break normalization and atomicity. For example, in a music database, one artist can appear on many albums and one album can contain many artists, so we create a junction table such as artist_album with artist_id and album_id as foreign keys.
>

4. When mapping a 1:1 relationship, how do you decide which table gets the foreign key? *(Section 3.4)*

> [!NOTE]
When mapping a 1:1 relationship, the foreign key gets put into the side that makes most sense based on the relationship. If one side is mandatory, its put there, if both sides are mandatory, it can go on either side, if both are optional, its best to put it on the side more likely to have a related record.
>

5. How does the mapping of a weak entity differ from a strong entity? What happens to the primary key? *(Section 3.5)*

> [!NOTE]
A weak entity cannot exist without its parent entity. Unlike a strong entity, it does not have its own independent primary key and is identified by a combination of the parent key and its own partial key. For example, a Room is weak because it belongs to a Hotel, so its key is usually (hotel_id, room_number).
>

6. Why should you never use `REAL` or `DOUBLE PRECISION` for monetary values? What should you use instead? *(Section 4.1)*

> [!NOTE]
REAL and DOUBLE are floating-point types and aren't to be used for monetary values due to precision errors. NUMERIC is the fix, because you can give precise parameters such as NUMERIC(10, 2) for those purposes. It is safer for financial calculations.
>

7. What is the difference between `TIMESTAMP` and `TIMESTAMPTZ`? Which should you prefer and why? *(Section 4.3)*
> [!NOTE]
TIMESTAMP shows the date, TIMESTAMPZ shows date and timezone. I prefer TIMESTAMPZ, because its more widely known and also shows the exact timezone of the displayed date, allowing easier readability and clarity.
>




8. Explain the difference between `CASCADE` and `RESTRICT` as foreign key delete actions. Give a scenario where each is appropriate. *(Section 6)*
> [!NOTE]
CASCADE on deletion removes child rows, such as in a case of deleting customers, their orders will also be deleted. Using RESTRICT on delete for the same customer entity will be prevented, if the customer has any orders (order history is important), and for example deleting a product with RESTRICT will prevent it being deleted if it is mentioned in the same aformentioned order history.
>




9. What is an insertion anomaly? Give an example and explain how proper schema design prevents it. *(Section 7)*
> [!NOTE]
An insertion anomaly is when you cannot add new data without also repeating unrelated information. Example: if each order record stores customer name and address, then inserting a new order for an existing customer forces repeated customer details. Normalized design prevents this by separating customers and orders into different tables and linking them with keys.
>




10. What is the difference between a surrogate key and a natural key? Give one advantage of each. *(Section 9)*
> [!NOTE]
A surrogate key is an artificial key, like a generated ID, while a natural key is a real business field that already identifies the row, like a passport number or email. One advantage of a surrogate key is that it is stable and simple to use as a primary key, while a natural key can be meaningful and avoid extra ID columns.
>




11. Why does PostgreSQL fold unquoted identifiers to lowercase? How does `snake_case` naming help? *(Section 8)*

> [!NOTE]
Because that is how PostgresSQL stores that information. Mixed case words need to use quotation marks to preserve the mixed case, which is tedious and tiresome, thats why it's folded to the form it's saved as anyway.
>

12. What does `SET NULL` do as a foreign key action? When would you use it instead of `CASCADE`? *(Section 6)*
> [!NOTE]
SET NULL means that when the parent row is deleted, the foreign key in the child row is set to NULL instead of the row being deleted. It is used when you want to keep the child record but remove its link to the parent, such as keeping an employee record after the department is deleted but leaving their department_id as NULL. This is a good choice when the child data should remain but no longer belong to a parent. If you want the child rows deleted too, use CASCADE instead.
>




---

## Exercise 3: Transformation Exercise — Hotel Booking System

### Given ER Diagram

A hotel booking system has the following entities and relationships:

**Entities:**

1. **Hotel** — hotel_id (PK), name, city, star_rating, phone
2. **Room** (weak entity, owned by Hotel) — room_number (partial key), room_type, floor, price_per_night, has_balcony
3. **Guest** — guest_id (PK), first_name, last_name, email, phone, passport_number
4. **Booking** — booking_id (PK), check_in_date, check_out_date, total_amount, status
5. **Service** — service_id (PK), name, description, price (e.g., "Room Service", "Spa", "Airport Shuttle")

**Relationships:**

- Hotel (1) → Room (N): A hotel has many rooms. Each room belongs to exactly one hotel. (Identifying relationship — Room is weak.)
- Guest (1) → Booking (N): A guest can make many bookings. Each booking belongs to one guest.
- Booking (M) ↔ Room (N): A booking can include multiple rooms, and a room can appear in many bookings (over time). The junction records the specific dates.
- Booking (M) ↔ Service (N): A booking can use multiple services, and a service can be used by many bookings. The junction records the date used and quantity.

### Task

1. Write `CREATE TABLE` statements for ALL tables (including junction tables).
2. For each table:
   - Choose appropriate data types
   - Define PK, FK, NOT NULL, UNIQUE, CHECK, and DEFAULT constraints
   - Specify ON DELETE and ON UPDATE actions for all FKs
3. Create the tables in the correct dependency order.
4. Explain why Room is a weak entity and how its PK reflects this.

> [!NOTE]
> ```sql
> CREATE TABLE hotels (
>     hotel_id SERIAL PRIMARY KEY,
>     name VARCHAR(150) NOT NULL,
>     city VARCHAR(100) NOT NULL,
>     star_rating SMALLINT NOT NULL CHECK (star_rating BETWEEN 1 AND 5),
>     phone VARCHAR(20) NOT NULL UNIQUE
> );
>
> CREATE TABLE guests (
>     guest_id SERIAL PRIMARY KEY,
>     first_name VARCHAR(100) NOT NULL,
>     last_name VARCHAR(100) NOT NULL,
>     email VARCHAR(255) NOT NULL UNIQUE,
>     phone VARCHAR(20),
>     passport_number VARCHAR(30) NOT NULL UNIQUE
> );
>
> CREATE TABLE services (
>     service_id SERIAL PRIMARY KEY,
>     name VARCHAR(100) NOT NULL UNIQUE,
>     description TEXT,
>     price NUMERIC(10,2) NOT NULL CHECK (price >= 0)
> );
>
> CREATE TABLE bookings (
>     booking_id SERIAL PRIMARY KEY,
>     guest_id INTEGER NOT NULL,
>     check_in_date DATE NOT NULL,
>     check_out_date DATE NOT NULL,
>     total_amount NUMERIC(10,2) NOT NULL CHECK (total_amount >= 0),
>     status VARCHAR(20) NOT NULL DEFAULT 'confirmed'
>         CHECK (status IN ('confirmed', 'completed', 'cancelled', 'pending')),
>     CONSTRAINT fk_bookings_guest
>         FOREIGN KEY (guest_id)
>         REFERENCES guests(guest_id)
>         ON DELETE RESTRICT
>         ON UPDATE CASCADE,
>     CONSTRAINT chk_booking_dates
>         CHECK (check_out_date > check_in_date)
> );
>
> CREATE TABLE rooms (
>     hotel_id INTEGER NOT NULL,
>     room_number INTEGER NOT NULL,
>     room_type VARCHAR(50) NOT NULL,
>     floor INTEGER NOT NULL CHECK (floor > 0),
>     price_per_night NUMERIC(10,2) NOT NULL CHECK (price_per_night > 0),
>     has_balcony BOOLEAN NOT NULL DEFAULT FALSE,
>     PRIMARY KEY (hotel_id, room_number),
>     CONSTRAINT fk_rooms_hotel
>         FOREIGN KEY (hotel_id)
>         REFERENCES hotels(hotel_id)
>         ON DELETE CASCADE
>         ON UPDATE CASCADE
> );
>
> CREATE TABLE booking_rooms (
>     booking_id INTEGER NOT NULL,
>     hotel_id INTEGER NOT NULL,
>     room_number INTEGER NOT NULL,
>     check_in_date DATE NOT NULL,
>     check_out_date DATE NOT NULL,
>     PRIMARY KEY (booking_id, hotel_id, room_number),
>     CONSTRAINT fk_booking_rooms_booking
>         FOREIGN KEY (booking_id)
>         REFERENCES bookings(booking_id)
>         ON DELETE CASCADE
>         ON UPDATE CASCADE,
>     CONSTRAINT fk_booking_rooms_room
>         FOREIGN KEY (hotel_id, room_number)
>         REFERENCES rooms(hotel_id, room_number)
>         ON DELETE RESTRICT
>         ON UPDATE CASCADE,
>     CONSTRAINT chk_booking_room_dates
>         CHECK (check_out_date > check_in_date)
> );
>
> CREATE TABLE booking_services (
>     booking_id INTEGER NOT NULL,
>     service_id INTEGER NOT NULL,
>     service_date DATE NOT NULL,
>     quantity INTEGER NOT NULL DEFAULT 1 CHECK (quantity > 0),
>     PRIMARY KEY (booking_id, service_id, service_date),
>     CONSTRAINT fk_booking_services_booking
>         FOREIGN KEY (booking_id)
>         REFERENCES bookings(booking_id)
>         ON DELETE CASCADE
>         ON UPDATE CASCADE,
>     CONSTRAINT fk_booking_services_service
>         FOREIGN KEY (service_id)
>         REFERENCES services(service_id)
>         ON DELETE RESTRICT
>         ON UPDATE CASCADE
> );
> ```

> [!NOTE]
> Room is a weak entity because it cannot exist independently of a Hotel. A room only makes sense inside a specific hotel, so its identity depends on the parent entity. Its primary key is therefore a composite key: `(hotel_id, room_number)`, where `hotel_id` comes from the owning hotel and `room_number` is the partial key that identifies the room within that hotel.
>

---

## Exercise 4: Data Type Selection

> [!NOTE]
> Fill in the **Your Data Type** and **Justification** columns in the table below.
For each column described below, choose the best PostgreSQL data type and write a brief justification (1–2 sentences). Do NOT just pick `VARCHAR` or `TEXT` for everything — think carefully about validation, storage, and query needs.

| # | Column Description | Your Data Type | Justification |
|---|---|---|---|
| 1 | Employee salary (exact, up to €999,999.99) | NUMERIC | Monetary values require precision, thats where NUMERIC(10,2) comes in. |
| 2 | Number of items in stock (never negative, max ~50,000) | SMALLINT | Its always non negative and maxes out around 50k, so it fits. |
| 3 | Whether a user's email is verified | BOOLEAN | True/False for the verification status, true if verified, false if not. Should suffice.|
| 4 | Customer's date of birth | DATE | I mean, a birthday is a date, so.. |
| 5 | Product description (variable length, could be several paragraphs) | TEXT | Text data type can be long and vary in length. No real limit as far as i know. |
| 6 | Country code (always exactly 2 letters, like "FI", "US") | CHAR | Country codes are exactly two letters, so CHAR(2) should enforce the fixed length. |
| 7 | IP address of a login attempt | INET | PostgresSQL apparently has a type specifically for IPV4/IPV6 addresses, so INET is the choice i'd take. |
| 8 | Order total (exact, up to €9,999,999.99) | NUMERIC | Order deals need to be precise and exact, so NUMERIC(9,2) avoids rounding errors. |
| 9 | GPS latitude of a store location | NUMERIC | Latitude needs to be exact aswell, but not have an unlimited range. NUMERIC(8,6) is the perfect fit. |
| 10 | A unique identifier for API tokens that must be globally unique across distributed systems | UUID | A globally unique distributed identifier is best represented by UUID, which is designed for uniqueness across systems. |
| 11 | Duration of a video in seconds (always a whole number) | INTEGER | Video duration in seconds is a whole number without decimals and it is small enough for INTEGER, which is simple. |
| 12 | Timestamp of when a record was last modified (users in multiple time zones) | TIMESTAMPZ | Because the users are in different timezones, TIMESTAMPZ shows that instant in time aswell as the timezone ( thats why TIMESTAMPZ instead of TIMESTAMP ) |
| 13 | A Finnish phone number like "+358 40 123 4567" | VARCHAR | Phone numbers have variable length and might include symbols such as + and spaces, so VARCHAR(20) is far more suitable than regular numeric types. |
| 14 | A percentage discount (0.00% to 100.00%) | NUMERIC | Percentage discounts need exact decimal precision, so NUMERIC avoids floating point errors when calculating. |
| 15 | A product's color options (e.g., a product comes in "red", "blue", "green") | TEXT[] | A product can have multiple color options, so an array type is a good fit for storing several values in one field.|

---

## Exercise 5: Constraint Design

For each business rule below, write the appropriate PostgreSQL constraint. Provide the constraint as it would appear inside a `CREATE TABLE` statement or as an `ALTER TABLE` statement.

### Part A: Single-Column Constraints

1. "A product's weight must be greater than zero (if provided)."

2. "Every customer must have an email address."

3. "Product names must be unique."

4. "An employee's hire date defaults to today if not specified."

5. "Order status can only be one of: 'new', 'confirmed', 'shipped', 'delivered', 'returned'."

> [!NOTE]
> ```sql
> -- 1
> weight NUMERIC(8,2) CHECK (weight IS NULL OR weight > 0),
> 
> -- 2
> email VARCHAR(255) NOT NULL,
> 
> -- 3
> name VARCHAR(255) NOT NULL UNIQUE,
> 
> -- 4
> hire_date DATE NOT NULL DEFAULT CURRENT_DATE,
> 
> -- 5
> status VARCHAR(20) NOT NULL
>     CHECK (status IN ('new', 'confirmed', 'shipped', 'delivered', 'returned')),
> ```

### Part B: Multi-Column Constraints

6. "A flight's arrival time must be after its departure time."

7. "In the `enrollments` table, the combination of `student_id` and `course_id` must be unique (a student can only enroll in a course once)."

8. "A discount percentage must be between 0 and 100, inclusive."

> [!NOTE]
> ```sql
> -- 6
> CHECK (arrival_time > departure_time)
> 
> -- 7
> UNIQUE (student_id, course_id)
> 
> -- 8
> CHECK (discount BETWEEN 0 AND 100)
> ```

### Part C: Foreign Key Constraints with Actions

9. "When a department is deleted, all employees in that department should have their `department_id` set to NULL (they become unassigned)."

10. "When a customer is deleted, prevent the deletion if the customer has any orders."

11. "When an author is deleted, all their blog posts should be deleted automatically."

12. "When a course is deleted, all enrollments for that course should be removed."

> [!NOTE]
> ```sql
> -- 9
> ALTER TABLE employees
> ADD CONSTRAINT fk_employees_department
> FOREIGN KEY (department_id)
> REFERENCES departments(department_id)
> ON DELETE SET NULL
> ON UPDATE CASCADE;
> 
> -- 10
> ALTER TABLE orders
> ADD CONSTRAINT fk_orders_customer
> FOREIGN KEY (customer_id)
> REFERENCES customers(customer_id)
> ON DELETE RESTRICT
> ON UPDATE CASCADE;
> 
> -- 11
> ALTER TABLE blog_posts
> ADD CONSTRAINT fk_posts_author
> FOREIGN KEY (author_id)
> REFERENCES authors(author_id)
> ON DELETE CASCADE
> ON UPDATE CASCADE;
> 
> -- 12
> ALTER TABLE enrollments
> ADD CONSTRAINT fk_enrollments_course
> FOREIGN KEY (course_id)
> REFERENCES courses(course_id)
> ON DELETE CASCADE
> ON UPDATE CASCADE;
> ```

---

## Submission Checklist

- [x] Exercise 1: `.sql` file with all CREATE TABLE statements + written justifications
- [x] Exercise 2: All 12 theory review answers
- [x] Exercise 3: Hotel booking schema with all tables and explanations
- [ ] Exercise 4: Data type selections with justifications for all 15 columns
- [ ] Exercise 5: All 12 constraints written in valid PostgreSQL syntax