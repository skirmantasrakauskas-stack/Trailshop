# Week 38 — Conceptual Data Modelling: Exercises

> [!IMPORTANT]
> ***How to Complete These Exercises***
> Write your answers directly in the highlighted **Your Answer** / **Your SQL** fields below each task. Replace the placeholder text with your own work before submitting.

These exercises accompany the Week 38 Theory material. Refer to the theory sections indicated in brackets when you need help.

---

## Exercise 1: TrailShop Project Task — Create the ER Diagram

**Goal:** Create a complete Entity-Relationship diagram for the TrailShop database using crow's foot notation.

> **From Week 37:** Last week each product had a single `category_id` (Category 1:N Product). That cannot store a product in two categories. This week's diagram must **not** put `category_id` on Product. Use **ProductCategory** as the junction that resolves Category M:N Product (see Theory Section 1.4).

### Instructions

Using the entity descriptions from Theory Section 12, create an ER diagram that includes:

1. **All six entities**: Category, Product, ProductCategory, Customer, Order, OrderItem
2. **All attributes** for each entity (as listed in Section 12.1)
3. **Primary keys** clearly marked (underline or "PK" label)
4. **Foreign keys** clearly marked (dashed underline or "FK" label)
5. **Relationships** between entities with:
   - Relationship name (verb)
   - Crow's foot notation showing cardinality and participation
6. **Identify weak / junction entities** — mark OrderItem as a weak entity, and mark ProductCategory as the junction that resolves Category M:N Product. Do **not** draw a direct M:N line between Category and Product.

### Requirements

- Use crow's foot notation (see Theory Section 9)
- You may use any tool: draw.io, Lucidchart, ERDPlus, dbdiagram.io, or even pen and paper (photograph and submit)
- The diagram must be readable — avoid crossing lines where possible
- Include a brief legend explaining your notation if using pen and paper

### Deliverables

- The ER diagram (image or link to online tool)
- A short written paragraph (3–5 sentences) that **must** explain why Week 37's 1:N `products.category_id` is being replaced by ProductCategory. You may also discuss another design decision (for example why OrderItem is a weak entity, or why `unit_price` is stored in OrderItem).

> [!NOTE]
![Screenshot](https://i.ibb.co/hx8VLs4g/chrome-4-Shc-Zfs-G6-H.png)
The Week 37 design used a single products.category_id foreign key, which meant each product could belong to only one category. This is insufficient for TrailShop because a product may need to be assigned to multiple categories, so we replace the direct 1:N relationship with a junction entity called ProductCategory. ProductCategory links Product and Category together and resolves the many-to-many relationship in a normalized way.
>

---

## Exercise 2: Theory Review Questions

Answer each question in 2–4 sentences. Reference the relevant theory section. Question 11b is extra: it connects last week's 1:N category FK to this week's junction.

1. Why should you create a conceptual data model before writing SQL? Give two specific reasons. *(Section 1)*

> [!NOTE]
Because it would prevent problems by making me think of data relationships, also of the rules that govern it. It would prevent duplicate data over several talbes, prevent unnecessary collumns when it could've been fixed by a junction table.
>
>

2. What is the difference between the conceptual level and the logical level of a data model? *(Section 2)*

> [!NOTE]
The conceptual level is what the data and relationships are, the logical level is how that data is structured in the database.

3. Explain logical data independence with an example. *(Section 3)*

> [!NOTE]
Changing the conceptual schema doesn't change the user views with logical data independence. For example if we split item and item_description apart, the user view wouldn't change when using JOIN to combine their data.
>

4. Explain physical data independence with an example. *(Section 3)*

> [!NOTE]
Achieving physical data independence means you can change the internal schema without changing the conceptual or external schemas. For example if we moved the database to a faster server, nothing would change or break.
>

5. What is the difference between a strong entity and a weak entity? Give one example of each (not from TrailShop). *(Section 5)*

> [!NOTE]
A strong entity can be uniquely identified by its own attributes like a fish store has a table "Fish" and that table has fish_id, it would be uniquely identified because each type of fish is different. A weak entity couldn't be identified by its own attributes alone, such as Transaction, which would depend on BankAccount, because there could be transacion no.1 for every different BankAccount, therefore it's weak and depends on BankAccount
>

6. What is a composite attribute? How does it differ from a multivalued attribute? Give an example of each. *(Section 6)*

> [!NOTE]
A composite attribute means being able to be split into several smaller attributes, such as address being street, house, apartment, door etc. A multivalued attribute can hold several values for a single entity, such as boots being both waterproof and also at the same time durable, black, etc.
>

7. What is a derived attribute? Why is it usually not stored in the database? *(Section 6)*

> [!NOTE]
Derived attributes aren't stored because they are computed when it is needed. For example someone's age calculation changes day by day, so to avoid inconsistency, you calculate it when it is needed, apart from some instances where its required to be stored for performance reasons.
>

8. Explain the difference between a binary relationship and a unary (recursive) relationship. Give an example of each. *(Section 7)*
> [!NOTE]
A binary relationship is a relationship between two entity types, for example it would be Customer and Order, where a customer placed an order (Only two). A Unary relationship is where a single entity is related to itself, such as a Phone is a product, and so is a Phone case, they are both Products, but are related to eachother. 
>




9. What is the difference between an identifying relationship and a non-identifying relationship? How does this affect the child table's primary key? *(Section 7)*
> [!NOTE]
A strong (non identifying) relationship is where both entities can exist independently, neither needs the other for identification. A weak (identifying) relationship depends on eachother for its identification. The child's foreign key in a weak relationship is part of the primary key.
>




10. In crow's foot notation, what does the following endpoint mean: a circle followed by a crow's foot (fork)? *(Section 9)*

> [!NOTE]
It means zero or many. Optional to participate and there can be many. 
>




11. Why can't a many-to-many (M:N) relationship be directly implemented in a relational database? What is the solution? *(Section 10)*

> [!NOTE]
A many-to-many relationship cannot be directly implemented in a relational database because one row cannot hold multiple foreign keys in a single attribute. The solution is to create a junction table, such as ProductCategory, which stores the relationship and usually uses a composite primary key made from both foreign keys.
>

11b. Last week TrailShop used `products.category_id` so each product belonged to exactly one category. Why is that insufficient, and what ER construct replaces it? *(Section 1.4)*

> [!NOTE]
This is insufficient because a product can belong to more than one category, but one foreign key can only store one category_id. The ER construct that replaces it is a junction/associative entity such as ProductCategory, which links Product and Category and represents the many-to-many relationship.
>

12. A business rule states: "Every employee must belong to exactly one department, and every department must have at least one employee." Express this using min-max notation for both sides. *(Section 8)*

> [!NOTE]
Department (1, N) -- Employee (1;1)
>

---

## Exercise 3: ER Diagram Reading Exercise

### Diagram A: Library System

Study the following ER description and answer the questions below.

```
┌──────────┐                        ┌──────────┐
│  AUTHOR  │──||──────O<────────────│   BOOK   │
└──────────┘                        └─────┬────┘
                                          │
                                    ||    │
                                          │
                                    O<    │
                                          │
                                   ┌──────┴─────┐
                                   │    LOAN     │
                                   └──────┬──────┘
                                          │
                                    ||    │
                                          │
                                    O<    │
                                          │
                                   ┌──────┴──────┐
                                   │   MEMBER    │
                                   └─────────────┘
```

Relationships (in crow's foot):
- Author `──||──────O<──` Book
- Book `──||──────O<──` Loan
- Member `──||──────O<──` Loan

**Questions:**

a) Can an author exist without having written any books? Explain using the notation.
> [!NOTE]
Yes, an author can exist without having written any books, but a book must have an author. 0< means zero or many, -||- means exactly one.
>

b) Can a book exist without being loaned? Explain using the notation.
> [!NOTE]
Yes a book can exist without being loaned, but a loan needs a book. 0< means zero or many, -||- means exactly one.
>

c) What type of entity is Loan in this diagram? Is it a junction/associative entity? Why?


> [!NOTE]
Loan is a weak, identifying entity, its also a junction/associative entity, it depends on both book and member and stores their relationship details between them.
>

d) What is the cardinality of the Author-Book relationship? Is this realistic? What might be a more accurate model?


> [!NOTE]
It is 1:N, one to many, no it's not realistic, because a book can be written by multiple authors and an author can wrie multiple books. A more accurate model would include AuthorBook, where it would work as a many to many relationship junction table.
>

e) What attributes would you add to the Loan entity?


> [!NOTE]
I would add:
loan_id(PK)
member_id(FK)
book_id(FK)
loan_date
due_date
return_date
>

### Diagram B: School System

```
STUDENT ──O|──────O<── ENROLLMENT ──>|──||── COURSE
                                        │
                                    ||  │
                                        │
                                    O<  │
                                        │
                                   TEACHER
```

Relationships:
- Student `──O|──────O<──` Enrollment (a student may have zero or many enrollments)
- Enrollment `──||──────||──` Course (each enrollment is for exactly one course)
- Teacher `──||──────O<──` Course (each course has zero or many sections, each taught by exactly one teacher)

**Questions:**

a) Can a student exist without being enrolled in any course?


> [!NOTE]
Yes, he/she can. You already explained the relations, so i won't explain it another time for redundancy reasons.
>

b) Can a course exist without having any enrolled students?


> [!NOTE]
Yes, it can.
>

c) What is the cardinality between Student and Course (through Enrollment)?


> [!NOTE]
It's many to many, a student can enroll in many courses, enrollment resolves this relationship.
>

d) Can a teacher exist without teaching any courses?


> [!NOTE]
Yes, the teacher side allows zero courses, so it can exist without being assigned to a course.
>

e) Is the Teacher-Course relationship 1:1 or 1:N? What does this imply about team teaching?


> [!NOTE]
It is 1:N, because a teacher can teach many courses, but each course is taught by EXACTLY ONE teacher.
>

---

## Exercise 4: ER Diagram Creation — Gym/Fitness Center

### Scenario

FitZone is a local gym and fitness center. They need a database to manage their operations. Here are the business rules:

1. The gym has **members**. Each member has an ID, first name, last name, email, phone, date of birth, and membership start date.

2. The gym offers **membership plans** (e.g., "Basic", "Premium", "Student"). Each plan has a plan ID, name, monthly price, and description. Each member subscribes to exactly one plan. A plan can have many members.

3. The gym has **trainers** (employees who lead classes). Each trainer has an ID, first name, last name, specialization (e.g., "Yoga", "CrossFit"), and hire date.

4. The gym offers **classes** (e.g., "Morning Yoga", "HIIT Blast"). Each class has an ID, name, day of the week, start time, end time, and maximum capacity. Each class is led by exactly one trainer, but a trainer can lead many classes.

5. Members can **register** for classes. A member can register for many classes, and a class can have many registered members. The registration records the registration date.

6. The gym has **equipment** (treadmills, dumbbells, etc.). Each piece of equipment has an ID, name, type, purchase date, and status ("working", "maintenance", "retired").

7. When equipment breaks, a **maintenance request** is created. Each request has an ID, request date, description of the problem, status ("open", "in progress", "closed"), and resolution date. Each request is for exactly one piece of equipment. One piece of equipment can have many maintenance requests over time.

### Task

1. Identify all entities and their attributes (including key attributes).

> [!NOTE]
![Screenshot](https://i.ibb.co/xK2FpWGS/chrome-V497-Wznhd-D.png)
membershipPlan(plan_id PK, name, monthly_price, description)
members(member_id PK, first_name, last_name, email, phone, date_of_birth, membership_start_date, plan_id FK)
trainers(trainer_id PK, first_name, last_name, specialization, hire_date)
classes(class_id PK, name, day_of_week, start_time, end_time, max_capacity, trainer_id FK)
registration(registration_id PK, member_id FK, class_id FK, registration_date)
equipment(equipment_id PK, name, type, purchase_date, status)
maintenanceRequest(request_id PK, equipment_id FK, request_date, problem_description, status, resolution_date)

>

2. Identify all relationships with their cardinality and participation constraints.

> [!NOTE]
membershipPlan -> members: 1:N, one plan can have many members, each member belongs to exactly one plan
trainers -> classes: 1:N, one trainer can lead many classes, each class is led by exactly one trainer
members <-> classes via registration: M:N, one member can register for many classes and one class can have many members, registration resolves this relationship and stores registration_date
members -> registration: 1:N, one member can have many registrations, each registration belongs to exactly one member
classes -> registration: 1:N, one class can have many registrations, each registration belongs to exactly one class
equipment -> maintenanceRequest: 1:N, one equipment item can have many maintenance requests, each maintenance request belongs to exactly one equipment item
>

3. Draw a complete ER diagram using crow's foot notation.

> [!NOTE]
> ![Screenshot](https://i.ibb.co/qLfq3ZP9/mspaint-Uabsds-Qt-J8.png)
>

4. Identify any entity that might be considered a weak entity or a junction/associative entity. Justify your answer.

> [!NOTE]
Registration is a junction/associative entiyt, because it resolves the M:N relationshop between members and classes, it is not a weak entity, as it has its own registration_id primary key. MaintenanceRequest is a strong entity, as it has its own request_id primary key, but it depeends on equipment for its relationship.
>

5. Are there any M:N relationships? If so, what junction entity resolves them?

> [!NOTE]
There is one M:N relationship which is members and classes. Registration junction entity resolves this by storing member_id, class_id and registration_date.
>
---

## Exercise 5: Find and Correct the Errors

The following ER diagram description contains **four errors**. Find each error, explain why it's wrong, and provide the correction.

### Scenario: Online Bookstore

**Entities and attributes:**

1. **Books**
   - book_id (PK)
   - title
   - author_name
   - price
   - genres (stores "Fiction, Mystery, Thriller" as a comma-separated string)

2. **Customer**
   - customer_id (PK)
   - full_name
   - address

3. **Purchase**
   - purchase_id (PK)
   - purchase_date
   - total_amount

**Relationships:**
- Books to Customer: M:N (implemented directly — no junction table)
- Customer to Purchase: 1:N (one customer, many purchases)
- Books to Purchase: no relationship defined

### Your Task

Find the four errors in this design and for each one:

a) State what the error is
[!NOTE]
1. Books to Customer is modeled as a direct M:N relationship.
2. Genres is stored as a comma-separated string.
3. Author name is stored directly in books.
4. Books to purchase has no relationship defined.
>

b) Explain why it's a problem (reference the relevant theory section)

[!NOTE]
1. This is a many-to-many relationship, and Chapter 10 explains that it cannot be directly implemented in a relational database without breaking the relational model.
2. This is a multivalued attribute, and Chapter 6 explains that multiple values should not be stored in one field because it creates redundancy and makes the data harder to query.
3. This is another M:N relationship, because a book can have many authors and an author can write many books. Chapter 7 and Chapter 10 explain that this should not be represented as a single attribute in one entity.
4. There is no relationship between Books and Purchase, which is a missing relationship issue discussed in Chapter 7 and Chapter 8. Without it, the database cannot show which books are included in each purchase or store details like quantity and price.
>

c) Describe how to fix it

> [!NOTE]
1. Create a junction/associative entity such as PurchaseItem or BookCustomer to resolve the M:N relationship properly.
2. Replace the genres field with a separate Genre table or a BookGenre junction table so each genre is stored as its own value.
3. Create an Author entity and a BookAuthor junction table so books and authors are linked correctly.
4. Add a PurchaseItem entity that links Purchase to Book and stores details such as quantity and unit price.
>

**Hints:** Think about multivalued attributes, M:N relationships, entity naming conventions, and missing relationships.

---

## Submission Checklist

- [x] Exercise 1: ER diagram + design decision paragraph (including why Week 37's category FK is replaced)
- [x] Exercise 2: All 12 theory review answers, plus 11b
- [x] Exercise 3: All questions answered for both Diagram A and Diagram B
- [x] Exercise 4: Entity list, relationship list, ER diagram, and justifications
- [x] Exercise 5: Four errors identified with explanations and corrections