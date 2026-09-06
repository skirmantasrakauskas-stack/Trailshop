# Week 36 — Exercises & Project Task

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 36 Theory material. Complete all sections.

---

## Part 1: TrailShop Project Task

### Task 1: Set Up PostgreSQL

Goal: install a working PostgreSQL server on your computer and confirm you can start `psql` (or an equivalent client).

---

### Local installation

**Step 1 — Download the installer**

1. Open [https://www.postgresql.org/download/windows/](https://www.postgresql.org/download/windows/).
2. Click **Download the installer** (EDB installer is fine).
3. Choose a recent stable version (e.g. 16 or 17) for **Windows x86-64**.
4. Save the `.exe` file and run it.

_(macOS / Linux: start from [https://www.postgresql.org/download/](https://www.postgresql.org/download/) and follow the OS-specific guide.)_

**Step 2 — Run the setup wizard**

Work through the wizard. Suggested choices for this course:

1. **Installation directory** — leave the default.
2. **Select components** — keep at least **PostgreSQL Server**, **pgAdmin 4**, **Command Line Tools**, and **Stack Builder** (Stack Builder can be skipped later).
3. **Data directory** — leave the default.
4. **Password** — set a password for the superuser account `postgres`.
   **Write it down.** You will need it every time you connect.
5. **Port** — leave **5432** (default) unless that port is already in use.
6. **Locale** — leave the default.
7. Finish the install. You can decline launching Stack Builder if prompted.

**Step 3 — Confirm the service is running (Windows)**

1. Press `Win`, type **Services**, open **Services**.
2. Find a service named like `postgresql-x64-16` (version number may differ).
3. Status should be **Running**. If not, right-click → **Start**.

**Step 4 — Open a terminal where** `psql` **is available**

On Windows, the easiest reliable options are:

- **Option 4a:** Start Menu → **SQL Shell (psql)** (installed with PostgreSQL), **or**
- **Option 4b:** Open **PowerShell** or **Command Prompt**.

If `psql` is not found in PowerShell, either use **SQL Shell (psql)** or add the PostgreSQL `bin` folder to your PATH, for example:

```
C:\Program Files\PostgreSQL\16\bin
```

(Adjust `16` to match your installed version.)

**Step 5 — Verify the installation**

In PowerShell / Command Prompt, run:

```
psql --version
```

You should see something like `psql (PostgreSQL) 16.x`.

If you used **SQL Shell (psql)** instead, you can skip `--version` and go straight to connecting in Task 2 — successful connection also proves the install works.

**Step 6 — Capture evidence for submission**

Take a screenshot of `psql --version` output (or of a successful `psql` connection prompt).

---

### Task 2: Create the TrailShop Database

Goal: create an empty database named `trailshop` and confirm you are connected to it.

Complete these steps after Task 1 succeeds.

**Step 1 — Connect to the PostgreSQL server as** `postgres`

**If you use SQL Shell (psql) on Windows**, press Enter to accept defaults for Server, Database, Port, and Username (`postgres`), then type the password you set during install when prompted.

**If you use PowerShell / Command Prompt**, run:

```
psql -U postgres -h localhost -p 5432
```

Enter the `postgres` password when asked.

You should see a prompt similar to:

```
postgres=#
```

That means you are connected to the default `postgres` maintenance database — which is normal before creating `trailshop`.

**Step 2 — List existing databases (optional but useful)**

At the `postgres=#` prompt, run:

```
\l
```

Scan the list. If `trailshop` already exists from an earlier attempt, skip Step 3 and go to Step 4.

**Step 3 — Create the database**

Still at the `postgres=#` prompt, run exactly:

```sql
CREATE DATABASE trailshop;
```

Expected result:

```
CREATE DATABASE
```

If you see `ERROR: database "trailshop" already exists`, the database is already there — continue to Step 4.

**Step 4 — Connect to** `trailshop`

In `psql`, run:

```
\c trailshop
```

Expected result (wording may vary slightly):

```
You are now connected to database "trailshop" as user "postgres".
```

**Step 5 — Confirm the prompt and that the database is empty**

1. Check that your prompt shows `trailshop`, for example:

```
trailshop=#
```

1. List tables:

```
\dt
```

You should see **no tables** (or a message that no relations were found). That is expected in Week 36 — tables come later.

1. Confirm the current database name with SQL:

```sql
SELECT current_database();
```

Expected: one row with `trailshop`.

**Step 6 — Quit psql (when finished)**

```
\q
```

**Step 7 — Capture evidence for submission**

Take a screenshot showing:

- successful `\c trailshop` (or the `trailshop=#` prompt), **and**
- `\dt` with an empty result / “Did not find any relations”
![Screenshot](https://i.ibb.co/93b8Rd12/Windows-Terminal-JVr-YWKMQTN.png)
![Screenshot](https://i.ibb.co/Y71c77F0/Windows-Terminal-OPSx-KE8-Xce.png)
---

### Troubleshooting (Tasks 1–2)

| Problem                                | What to try                                                                                                                  |
| -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `psql` is not recognized               | Use **SQL Shell (psql)**, or add PostgreSQL’s `bin` folder to PATH and open a **new** terminal                               |
| Password authentication failed         | Use the password set for user `postgres` during install; check Caps Lock                                                     |
| Connection refused / could not connect | Confirm the PostgreSQL Windows service is **Running**; confirm port **5432**                                                 |
| Port already in use                    | Either stop the other service using 5432, or reinstall/reconfigure PostgreSQL to another port and always pass `-p` that port |
| Permission denied to create database   | Connect as `postgres` (superuser), not as a limited role                                                                     |

---

### Task 3: Reflection Worksheet

Answer the following in your own words (write 2–3 sentences per point):

1. List **3 specific problems** TrailShop would face if they kept using spreadsheets as their product catalog grows to 5,000+ items with 10 staff members.

> [!NOTE]
> The first problem would definetly be managing a system that large with just 10 members, the scalability is trash on spreadsheets, because each cell record would have to be individually found and changed on seasonal, occasional sales and could accidentally be forgotten and overwritten by other members. Spreadsheets are notorious for problems relating to human error and have no concept of mandatory fields, a cell with slightly incorrect information ( such as an accidental empty space, negative symbol or a null value ) could lead to weird issues website, customer wise. Separate staff member departments ( procurement, warehouse, marketing, product managing, financial, etc.) would have unneccessary access to other confidential company information, due to the lack of security and role management in spreadsheets.

2. List **3 benefits** of switching to a database system, explaining how each one solves a problem from your list above.

> [!NOTE]
> Databases have MVCC, so the overwriting issues are fixed, several people can work on the system at once, its also much easier to control large amounts of information and change the values at once. Databases can have mandatory fields and rules, where null values or empty fields are instantly not accepted, its impossible to add records without satisfying the rules first. Security is far better, where certain user roles can only access certain information without having access to a different department's work.

3. Explain the three-schema architecture in your own words. Why is the separation into three levels useful?

> [!NOTE]
> The different layers are for different purposes, the external layer displays relevant information on the very surface, at the same time separating information based on roles. The conceptual level by my understanding is the level where most database developers work, create, modify, delete, etc. on the database information and inner structures. The internal level is basically where and how the information is stored, file formats, data compression etc., this is mainly used and edited by the DBMS. The separation is useful because it sections off areas depending on their purpose, its not one big melting pot. By my understanding it's separated to not have very large learning curves and be way more understandable, workable to the average person.
---

## Part 2: Theory Review Questions

Answer each question in 2–4 sentences. Reference the Theory material sections as needed.

### Short-Answer Questions

**Q1.** What is the difference between data and information? Give a concrete example using TrailShop data.
_(See Section 1 of this week's Theory material.)_

> [!NOTE]
> Data is raw facts, values, words, etc. information is like processed, organized and presentable data, it has a context and a concrete meaning, whereas data could just be several numbers in a line, where not having context could be of little value to someone, lets say, in a board meeting. Information is presented for example as: "Trailmaster X4 costs 149.99 euros and has 12 units in stock. Raw data for this exact same product would be : "Trailmaster X4, 149.99, 12".

**Q2.** List and explain three disadvantages of file-based data management systems. For each, describe how it would affect TrailShop specifically.
_(See Section 2 of this week's Theory material.)_

> [!NOTE]
>Concurrency access issues would involve different versions of the same file being edited, and accidentally overwritten. It would affect trailshop by incorrectly changing the values of pricing, there could also be different meanings for the same product. Security issues could cause large company wide incidents, due to lack of access control, a random angry intern could delete the entire system out of spite or just access information that he should not be able to access in his line of work. Several files that share some of the same data could have different not updated meanings, this would be bad, because it would be a consistency issue, it would affect any company, including TrailShop.

**Q3.** What is a DBMS? List four of its core functions.
_(See Sections 3 and 4 of this week's Theory material.)_

> [!NOTE]
>DBMS stands for database management system. 1. Data definition 2. Data manipulation 3. Data dictionary 4. Backup and recovery
>
**Q4.** Explain program-data independence with a concrete example. Why is it important?
_(See Section 5.2 of this week's Theory material.)_

> [!NOTE]
> Program-data independence means that changes to the structure of data do not require changes to existing programs. If lets say products table contains only name and price, a website might use SELECT name, price FROM products; and it would still work perfectly even if for example a collumn weight_kg was added. It is important, because it reduces the need to modify and test applications after a database structure modification occurs. This also means that it would be easier to maintain and update the systems.

**Q5.** What is metadata? Give two examples of metadata for a `products` table.
_(See Section 8 of this week's Theory material.)_

> [!NOTE]
>Metadata is "data about data", it explains the value of a collumn, for example a product_id is a numeric collumn, name is a text collumn.

**Q6.** What is the three-schema architecture? Name and briefly describe each level.
_(See Section 3.3 of this week's Theory material.)_

> [!NOTE]
> 1. External level - what users/applications see.
2. Conceptual level - The logical structure of the whole database.
3. Internal level - how data is physically stored on disk.

**Q7.** Explain the difference between logical data independence and physical data independence.
_(See Section 3.4 of this week's Theory material.)_

> [!NOTE]
> The main difference is what you change, logical data independence would be which and how data is organized in the database without breaking applications, for example, adding a new weight_kg collumn. Physical data independence would be changing how the data is stored on the computer/server without changing the dataabase structure, for example, adding an index to make searching faster.

**Q8.** What is a transaction? Why is atomicity important? Give a TrailShop example.
_(See Section 5.5 of this week's Theory material.)_

> [!NOTE]
> Transaction is a logical unit of work, which either completes full or doesn't at all. It is important in cases where one of the steps for a purchase (on trailshop example) orders addition> order_items addition > stock_quantity in products decrease > charging the customer for the purchase does not go through due to one of the steps failing. If one of the steps fail, all of them do, to not cause data consistency issues. 

### True/False

For each statement, write **True** or **False** and correct any false statements.

1. A DBMS stores only data, not information about the data's structure.
2. In a file-based system, changing the format of a data file requires updating every program that reads it.
3. Data redundancy means the same data is stored in multiple places.
4. PostgreSQL is a commercial, closed-source database system.
5. The conceptual level of the three-schema architecture describes how data is physically stored on disk.

> [!NOTE]
>False (A DBMS stores data and information about the data's structure, such as tables, columns, relationships, and constraints).
>True.
>True.
>False(PostgreSQL is a free and open-source database system).
>False(The conceptual level describes the logical structure of the database. The internal level describes how data is physically stored on disk).
### Matching Exercise

Match each term (1–10) with its definition (A–J).

| #   | Term              |
| --- | ----------------- |
| 1   | Data dictionary   |
| 2   | RDBMS             |
| 3   | Concurrency       |
| 4   | View              |
| 5   | Schema            |
| 6   | Data isolation    |
| 7   | Transaction       |
| 8   | SQL               |
| 9   | Data independence |
| 10  | ACID              |

| Letter | Definition                                                                          |
| ------ | ----------------------------------------------------------------------------------- |
| A      | A virtual table defined by a query, showing a subset of data                        |
| B      | Multiple users accessing data at the same time                                      |
| C      | The formal definition of a database's structure (tables, columns, types)            |
| D      | A logical unit of work that must complete fully or not at all                       |
| E      | The standard language for querying and managing relational databases                |
| F      | The system catalog storing metadata about the database                              |
| G      | Data trapped in separate files/formats that are hard to combine                     |
| H      | A DBMS based on the relational model, using tables and SQL                          |
| I      | The ability to change storage or structure without affecting applications           |
| J      | Atomicity, Consistency, Isolation, Durability — properties of reliable transactions |

> [!NOTE]
> ***Your Answers***
>
> | #   | Your Match |
> | --- | ---------- |
> | 1   |     F      |
> | 2   |     H      |
> | 3   |     B      |
> | 4   |     A      |
> | 5   |     C      |
> | 6   |     G      |
> | 7   |     D      |
> | 8   |     E      |
> | 9   |     I      |
> | 10  |     J      |

---

## Part 3: Practical Exercises

These exercises require a working PostgreSQL installation. See Part 1, Task 1 if you haven't set it up yet.

### Exercise 3.1: Explore psql

Connect to PostgreSQL using psql and complete the following. Write down the command you used and the output (or a summary of it).

1. List all databases on your server.
2. Connect to the `trailshop` database.
3. List all tables in the `trailshop` database.
4. Use `\?` to display the list of psql meta-commands. Find and write down the commands for:

- Describing a specific table's structure
- Listing all users/roles
- Showing help for a specific SQL command

5. Quit psql.

> [!NOTE]
> 1,2,3.
![Screenshot](https://i.ibb.co/yccCWYkw/Windows-Terminal-F8v7sp-Fek-D.png)
> 4.1. \d **table_name**
> 4.2. \du
> 4.3. \h **command**
> ctrl+c, Y
### Exercise 3.2: Explore the System Catalog

While connected to `trailshop`, run the following queries and write down what they return:

```sql
SELECT current_database();
```

```sql
SELECT version();
```

```sql
SELECT table_name FROM information_schema.tables
WHERE table_schema = 'public';
```

Why does the last query return no rows? What would you expect to see after creating tables in future weeks?

> [!NOTE]
> There are no tables created.
![Screenshot](https://i.ibb.co/CKbxynfm/Windows-Terminal-02-K2c8m-Kr6.png)
> It would return publically available tables
### Exercise 3.3: Create and Drop a Test Database

Practice database creation and deletion:

```sql
-- Create a test database
CREATE DATABASE test_playground;

-- List databases to confirm it exists
\l

-- Drop (delete) the test database
DROP DATABASE test_playground;

-- List databases again to confirm it's gone
\l
```

![Screenshot] (https://i.ibb.co/jPrk788c/Windows-Terminal-Qydqm-Hjfg9.png)


**Warning:** `DROP DATABASE` permanently deletes a database and all its data. Always double-check the database name before running this command.

---

## Submission Checklist

- [ ] PostgreSQL installed and working (screenshot of `psql --version` or equivalent)
- [ ] `trailshop` database created (screenshot of `\c trailshop` showing successful connection)
- [ ] Reflection Worksheet answers (Part 1, Task 3)
- [ ] Theory Review Questions answered (Part 2)
- [ ] Practical Exercise outputs documented (Part 3)