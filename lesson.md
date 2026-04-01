# 🎓 Lesson 1.2: Intro to Databases — Instructor Guide

## Session Overview

| Item | Detail |
|------|--------|
| **Duration** | 3 hours |
| **Format** | Flipped Classroom + Problem-Based Learning |
| **Prerequisites** | Pre-class reading on database types, keys, and normalisation; dbdiagram.io account |
| **Tools** | Browser + [dbdiagram.io](https://dbdiagram.io) — no installation required |

### Agenda

| Time | Section | Focus |
|------|---------|-------|
| 0:00 – 0:55 | Part 1: The Data Landscape | Relational vs. NoSQL vs. Vector databases |
| 0:55 – 1:00 | Break | — |
| 1:00 – 1:55 | Part 2: Building the Blueprint | Primary Keys, Foreign Keys, ERD design |
| 1:55 – 2:00 | Break | — |
| 2:00 – 2:55 | Part 3: Organising Data | Normalisation — 1NF, 2NF, 3NF |
| 2:55 – 3:00 | Wrap-Up | Key Takeaways & Post-Class Assignment Briefing |

---

## 🏃 Part 1: The Data Landscape (55 min)

### 🎯 Learning Objective
Analyse the differences between Relational, NoSQL, and Vector databases and select the appropriate type for a given business use case.

### 📖 Theory Recap (10 min)

**Analogy:** Think of three types of storage systems:
- **Relational (SQL):** A filing cabinet — rigid drawers, labelled folders, fixed forms. Perfect for structured, consistent data.
- **NoSQL:** A bag of zip-lock bags — flexible, can hold anything, easy to add new types. Great for fast-changing or varied data.
- **Vector:** A library organised by topic similarity, not alphabetical order. Built for semantic search and AI applications.

| Database Type | Best For | Examples |
|---------------|---------|----------|
| **Relational (SQL)** | Financial records, inventory, accounts | PostgreSQL, MySQL, DuckDB |
| **NoSQL** | Social feeds, chat logs, IoT data | MongoDB, Redis, Firestore |
| **Vector** | Image search, LLM memory, recommendations | Pinecone, pgvector, Weaviate |

Common SQL data types: `INT`, `VARCHAR`, `TEXT`, `DATE`, `BOOLEAN`, `DECIMAL`

### 🛠️ Hands-On Activity: "The Sorting Game" (35 min)

**Scenario:** A startup is building a super-app with multiple features. Each feature team is arguing over which database to use.

**Task:** For each business feature below, vote on which database type fits best and explain why:

| Feature | Best DB Type | Why? |
|---------|-------------|------|
| User payment history | Relational | Structured, needs ACID compliance |
| User-uploaded photos | NoSQL or object store | Unstructured; schema-free |
| "Find similar products" search | Vector | Semantic similarity matching |
| Live chat messages | NoSQL | High velocity, flexible schema |
| Monthly financial reports | Relational | Joins, aggregations, integrity |

**Discussion Questions:**
- "Could you build the entire app with just one database type? What would break?"
- "Why would a bank never use a NoSQL database for transaction records?"

### 💬 Q&A & Reflection (10 min)

- **Common Misconception:** "SQL databases are old and being replaced by NoSQL." → SQL databases handle the vast majority of the world's structured business data. NoSQL and Vector DBs are additive, not replacements.
- **Business Case:** Uber uses a combination: PostgreSQL for ride billing (Relational), Cassandra for real-time location tracking (NoSQL), and vector databases for driver-rider matching algorithms.

---

## 🏃 Part 2: Building the Blueprint (55 min)

### 🎯 Learning Objective
Apply Primary Keys and Foreign Keys to design an Entity-Relationship Diagram (ERD) using DBML syntax.

### 📖 Theory Recap (10 min)

**Analogy:**
- **Primary Key (PK):** Your national ID number — unique, non-null, cannot change.
- **Foreign Key (FK):** A reference to someone else's ID — it proves the relationship exists but lives in the other table.

Key rules:
- Every table must have exactly one PK.
- FKs reference a PK in another table.
- Deleting a parent record with dependent child records causes a **referential integrity error** unless handled.

DBML syntax for relationships: `Ref: child_table.fk_column > parent_table.pk_column`

### 🛠️ Code-Along: "Car Insurance Schema" (20 min)

Walk students through building this schema in dbdiagram.io:

```dbml
Table customers {
  id INT [pk, increment]
  name VARCHAR
  email VARCHAR [unique]
  created_at DATETIME
}

Table cars {
  id INT [pk, increment]
  customer_id INT [ref: > customers.id]
  make VARCHAR
  model VARCHAR
  year INT
}

Table accidents {
  id INT [pk, increment]
  car_id INT [ref: > cars.id]
  date DATE
  claim_amount DECIMAL
}
```

**Discussion:** "How many accidents can one car have? How many cars can one customer have? Draw the cardinality."

### 🛠️ Workshop: "The School System ERD" (25 min)

**Task:** In dbdiagram.io, build a schema where:
- Students belong to exactly one class.
- Teachers teach multiple classes.
- Classes have one teacher but many students.

**Discussion Questions:**
- "What happens if you try to delete a teacher who still has classes assigned?"
- "Why do we use `teacher_id` in the classes table rather than storing the teacher name directly?"

### 💬 Q&A & Reflection (10 min)

- **Common Misconception:** "I can just store everything in one table to keep it simple." → This leads to update anomalies — change one piece of information and you must update dozens of rows. One misses = data inconsistency.
- **Business Case:** Every e-commerce platform (Amazon, Shopify) uses this exact PK/FK pattern to link users → orders → order_items → products. A poorly designed schema at launch can cost millions to refactor.

---

## 🏃 Part 3: Organising Data — Normalisation (55 min)

### 🎯 Learning Objective
Evaluate an unnormalised dataset and decompose it step-by-step into 3rd Normal Form (3NF).

### 📖 Theory Recap (10 min)

**Analogy:** Imagine a coffee shop recording orders like this: `"Alice, 123 Main St, Oat Latte £4, Croissant £3.50"` — all in one cell. If Alice moves, you update 47 rows. That's the problem normalisation solves.

**The Three Rules:**

| Normal Form | The Rule | The Fix |
|-------------|---------|---------|
| **1NF** | No lists or repeating groups in cells | One value per cell; separate rows for repeating items |
| **2NF** | Every non-key column depends on the *whole* primary key | Split tables so each piece of data belongs to its natural entity |
| **3NF** | No transitive dependencies (A → B → C) | If C depends on B and B depends on A, move C to B's table |

**Key insight:** "Every fact should be about the key, the whole key, and nothing but the key."

### 🛠️ Workshop: "E-Commerce Deconstruction" (35 min)

**Start with this unnormalised orders table:**

| OrderID | CustomerName | CustomerEmail | ProductName | ProductCategory | UnitPrice | Qty |
|---------|-------------|--------------|------------|----------------|-----------|-----|
| 1 | Alice | alice@x.com | USB Hub | Electronics | 25.00 | 2 |
| 1 | Alice | alice@x.com | HDMI Cable | Electronics | 12.00 | 1 |
| 2 | Bob | bob@x.com | USB Hub | Electronics | 25.00 | 1 |

**Step 1 — 1NF:** Already met (one value per cell). Add `order_item_id` as the new composite PK.

**Step 2 — 2NF:** `CustomerName` and `CustomerEmail` depend only on `OrderID`, not on the item. Split:
- **orders:** `(order_id, customer_id, order_date)`
- **customers:** `(customer_id, name, email)`
- **order_items:** `(order_item_id, order_id, product_id, qty)`

**Step 3 — 3NF:** `ProductCategory` depends on `ProductName`, not directly on the order. Split:
- **products:** `(product_id, product_name, category_id, unit_price)`
- **categories:** `(category_id, category_name)`

**Discussion:** "Why store `unit_price` in `order_items` instead of just referencing it from `products`?"

*(Answer: Historical accuracy — if the product price changes, past order amounts must remain correct.)*

### 💬 Q&A & Reflection (10 min)

- **Common Misconception:** "More normalisation is always better." → Over-normalisation creates complex JOIN queries that slow down performance. Large-scale analytical databases sometimes deliberately *denormalise* for speed.
- **Business Case:** Shopify's order database uses a 3NF-compliant schema for transactional integrity, but their analytics warehouse uses a denormalised star schema for fast reporting — two different tools for two different jobs.

---

## 🎯 Wrap-Up (5 min)

### Key Takeaways
1. **Database type selection** is a design decision, not a preference — match the tool to the data shape and query pattern.
2. **Keys are the skeleton of your database** — every table needs a PK; FKs define relationships.
3. **Normalisation removes redundancy** — but know when to stop. Production databases often balance between 3NF and performance.

### Next Steps
- **Post-Class:** Complete the [FoodFast Database Design Challenge](./assignment.md) — design and export a 7-table schema (45–60 min).
- **Next Lesson:** Lesson 1.3 is where you start *building* these schemas with real SQL DDL commands.
