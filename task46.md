# Week 46 — Transactions and Concurrency: Exercises

> [!IMPORTANT]
> **_How to Complete These Exercises_**
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

---

## 1. TrailShop Project Task

### Task 1.1: Write an Order Transaction

Write a complete transaction that processes a new order in TrailShop:

1. Insert a new order for an existing customer.
2. Insert 2–3 order items.
3. Decrease `stock_quantity` for each ordered product.
4. Include a check that stock doesn't go negative (if it would, ROLLBACK).
5. Use a SAVEPOINT before the stock update step.
   > [!NOTE]
   > **_Your SQL_**
   >
   > ```sql
   BEGIN;

   INSERT INTO orders (customer_id, order_date, status)
   VALUES (2, CURRENT_DATE, 'pending');

   INSERT INTO order_items (order_id, product_id, quantity, unit_price)
   VALUES
       (13, 6, 2, 200.00),
       (13, 7, 1, 2.00);

   SAVEPOINT sp1;
   
   DO $$
   BEGIN
       IF (SELECT stock_quantity FROM products WHERE product_id = 6) < 2
          OR (SELECT stock_quantity FROM products WHERE product_id = 7) < 1
       THEN
           RAISE EXCEPTION 'Not enough stock';
       END IF;
   END $$;

   UPDATE products
   SET stock_quantity = stock_quantity - 2
   WHERE product_id = 6;

   UPDATE products
   SET stock_quantity = stock_quantity - 1
   WHERE product_id = 7;
   >
   > ```

Test the transaction by:

- Running it successfully with sufficient stock.
- Running it where stock is insufficient — verify the ROLLBACK works.

### Task 1.2: Observe READ COMMITTED Behavior (Optional)

> [!TIP]
> **Recommended practice.** Do Tasks 1.2–1.4. They are not required to finish the TrailShop project. They prepare you for the exams.

Open two psql sessions (or two query windows). Demonstrate a non-repeatable read:

1. Session A: `BEGIN;` then `SELECT stock_quantity FROM products WHERE product_id = 101;`
2. Session B: `BEGIN;` then `UPDATE products SET stock_quantity = stock_quantity - 5 WHERE product_id = 101;` then `COMMIT;`
3. Session A: `SELECT stock_quantity FROM products WHERE product_id = 101;` (observe different value)
4. Session A: `COMMIT;`

Document what you observe and explain why it happens at READ COMMITTED.

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Task 1.3: Observe Locking (Optional)

Open two sessions and demonstrate row-level locking:

1. Session A: `BEGIN;` then `SELECT * FROM products WHERE product_id = 101 FOR UPDATE;`
2. Session B: Try `UPDATE products SET price = 99.99 WHERE product_id = 101;` — observe it waits.
3. Session A: `COMMIT;` — observe Session B proceeds.

Document the behavior.

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Task 1.4: Experiment with Isolation Levels (Optional)

1. Repeat Task 1.2, but set Session A to `REPEATABLE READ`. Does the non-repeatable read still occur?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_
>
> 2. In a REPEATABLE READ transaction, try to UPDATE a row that another transaction has already committed a change to. What error do you get?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_
>
> 3. Document the differences between READ COMMITTED and REPEATABLE READ behavior.
>
> _(Write your answer here.)_

---

## 2. Theory Review Questions

Answer in your own words (2–4 sentences each):

1. What is a transaction, and why is it important for data integrity?

> [!NOTE]
> **_Your Answer_**
>
> Transaction is a logical unit of work, where several SQL statements are done together as a full group. It is important for data integrity because if one of the statements fails, the changes from the other statements can be rolled back, so the database does not get partial or incorrect data.

2. Explain the Atomicity property using a real-world example (not from the lecture notes).

> [!NOTE]
> **_Your Answer_**
>
> Lets say there is an airport changing gates for a flight, removing the flight from one boarding gate and adding it to another. Atomicity means both changes would need to work, if the second change fails, the old gate assignment would also be rolled back. Otherwise passengers could be sent to a gate that is no longer assigned to the flight.
>
> 3. What does Isolation guarantee? Why is it needed in a multi-user system?

> [!NOTE]
> **_Your Answer_**
>
> Isolation means that transactions running at the same time should not interfere with each other or see each other's unfinished changes. It is needed in a system with several users because one user could place an order while another updates stock, and without isolation the system could give incorrect prices or stock counts.

4. What is the difference between COMMIT and ROLLBACK?

> [!NOTE]
> **_Your Answer_**
>
> COMMIT saves all the changes made in the current transaction and makes them permanent. ROLLBACK cancels the changes and brings the database back to the situation it had before the transaction, or before a chosen savepoint.

5. Explain what a SAVEPOINT is and give a scenario where it's useful.

> [!NOTE]
> **_Your Answer_**
>
> A SAVEPOINT is a named point inside a transaction that lets you cancel only the changes after that point. It could be useful when deleting one customer's order_items, orders and customer in separate steps, so if the customer deletion fails, we can roll back to the savepoint for that step instead of losing the earlier successful changes.
>
> 6. What is a dirty read? Why does PostgreSQL never allow it?

> [!NOTE]
> **_Your Answer_**
>
> A dirty read is when one transaction sees data that another transaction has changed but not committed yet. PostgreSQL never allows it because MVCC only gives transactions the committed version of a row, so uncommitted changes from another transaction are not visible.

7. Describe the lost update problem. How can SELECT FOR UPDATE prevent it?

> [!NOTE]
> **_Your Answer_**
>
> A lost update happens when two transactions read the same value and update it, while one update overwrites the other one. For example, both workers read stock 10 and update it separately instead of combining both shipped amounts. SELECT FOR UPDATE locks the rows while the first transaction uses them, so the second transaction has to wait and can not change the rows based on the old value.

8. What is the default isolation level in PostgreSQL? What concurrency problems does it still allow?

> [!NOTE]
> **_Your Answer_**
>
> The default isolation level in PostgreSQL is READ COMMITTED. It does not allow dirty reads, but it can still allow non-repeatable reads and phantom reads because each statement gets a new view of the committed data.

9. Explain MVCC in one paragraph: what is it, and what advantage does it provide?

> [!NOTE]
> **_Your Answer_**
>
> MVCC means multi-version concurrency control, basically PostgreSQL keeps several versions of a row instead of replacing the old version immediately. It uses transaction rules to decide which version each transaction can see, so readers and writers can work at the same time without blocking each other for every operation.

10. What is a deadlock, and how does PostgreSQL handle it when one is detected?

> [!NOTE]
> **_Your Answer_**
>
> A deadlock happens when two or more transactions are waiting for each other's locks, for example transaction A is holding row 1 and waiting for row 2, while transaction B holds row 2 and waits for row 1. PostgreSQL detects this after the deadlock timeout and aborts one transaction, which releases its locks and lets the other transaction continue.

---

## 3. Scenario Analysis (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

For each scenario, identify: (a) which concurrency problem occurs, and (b) at which isolation level(s) it can happen.

### Scenario A

Transaction 1 reads customer 5's order count (3 orders). Transaction 2 inserts a new order for customer 5 and commits. Transaction 1 reads customer 5's order count again and gets 4.

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Scenario B

Transaction 1 reads product 101's price (€89.99). Transaction 2 updates product 101's price to €79.99 but then rolls back. If Transaction 1 had somehow seen €79.99, what problem would that be?

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

### Scenario C

Two warehouse workers simultaneously check stock for product 101 (both see 10). Worker A ships 3 units and sets stock to 7. Worker B ships 4 units and sets stock to 6. The correct stock should be 3.

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

---

## 4. Transaction Writing (Optional)

> [!TIP]
> **Recommended practice.** Do this section. It is not required to finish the TrailShop project. It prepares you for the exams.

Write complete SQL transactions for the following TrailShop scenarios. Include BEGIN, appropriate operations, error handling (SAVEPOINT where useful), and COMMIT.

### Transaction 1: Bulk Price Update

Update the price of all products in the "Footwear" category by +10%. If any product's new price would exceed €200, rollback the entire update.

### Transaction 2: Customer Account Deletion

Delete a customer and all their associated data:

1. Delete their order_items (for all their orders)

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_
>
> 2. Delete their orders

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_
>
> 3. Delete the customer record

> [!NOTE]
> **_Your Answer_**
>
> _(Write your answer here.)_

Use SAVEPOINTs between each step in case of errors.

### Transaction 3: Inventory Restock

Restock 3 products (add specific quantities to stock). Use SELECT FOR UPDATE to lock the rows first, preventing lost updates. Include a check that the new stock doesn't exceed a maximum (e.g., 200).

---

## 5. True/False

For each statement, write TRUE or FALSE and briefly explain why.

1. In PostgreSQL, a single SQL statement outside of an explicit BEGIN/COMMIT is not a transaction.
   - FALSE. PostgreSQL treats each standalone SQL statement as its own implicit transaction and automatically commits it if it succeeds.
2. ROLLBACK undoes all changes made since the last SAVEPOINT.
   - FALSE. Plain ROLLBACK cancels the entire transaction. ROLLBACK TO SAVEPOINT is the command that undoes changes made after a named savepoint.
3. At READ COMMITTED isolation level, a transaction can never see another transaction's uncommitted changes.
   - TRUE. READ COMMITTED does not show another transaction's uncommitted changes.
4. REPEATABLE READ in PostgreSQL prevents phantom reads.
   - TRUE. PostgreSQL REPEATABLE READ uses a transaction snapshot, so rows committed after the snapshot should not appear in later reads.
5. SELECT FOR UPDATE blocks other SELECT statements on the same rows.
   - FALSE. A normal SELECT can still read committed row versions. FOR UPDATE blocks operations that need a conflicting lock on the selected rows.
6. If a deadlock occurs, PostgreSQL will wait indefinitely for it to resolve.
   - FALSE. PostgreSQL detects the deadlock and cancels one of the transactions, letting the rest continue.
7. MVCC means PostgreSQL keeps multiple versions of each row, allowing readers and writers to work concurrently without blocking.
   - TRUE. MVCC stores several row versions and lets readers use a committed version while a writer updates a newer version.
8. Once a transaction is committed, its changes can be undone with ROLLBACK.
   - FALSE. ROLLBACK works only inside an active transaction. After COMMIT, another transaction or compensating SQL statement is needed to undo changes.

---

## Submission Checklist

**Required**

- [x] Order transaction written and tested, including the insufficient-stock rollback (Task 1.1)
- [x] Theory review questions answered
- [x] True/False questions answered with a short explanation

**Recommended practice**

- [ ] READ COMMITTED, locking, and isolation-level experiments documented (Tasks 1.2–1.4)
- [ ] Scenario analysis completed
- [ ] Extra transactions written