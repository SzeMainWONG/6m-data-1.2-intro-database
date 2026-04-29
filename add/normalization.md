# What is Database Normalization?

Normalization is the process of structuring relational tables to reduce redundancy and prevent data anomalies. It involves decomposing tables into smaller, well-structured tables and defining relationships between them.

---

## Why Normalize?

Without normalization, the following data anomalies can occur:

- **Update Anomaly:** If data is duplicated across multiple rows, updating one instance may lead to inconsistencies if other instances are not updated.
- **Insert Anomaly:** Inability to add data due to missing related data.
- **Delete Anomaly:** Deleting a record may inadvertently remove other valuable data.

Consider the following unnormalized table `Orders`:

| order_id | customer_name | customer_email | product_name | product_price | quantity |
| -------- | ------------- | -------------- | ------------ | ------------- | -------- |
| 1        | Alice         | alice@a.com    | Orange       | 3.00          | 1        |
| 2        | Bob           | bob@a.com      | Banana       | 2.00          | 2        |
| 3        | Alice         | alice@a.com    | Apple        | 4.00          | 1        |
| 4        | Alice         | alice@a.com    | Grape        | 5.00          | 3        |

---

## Update Anomaly Example

An update anomaly occurs when we need to update the same data in multiple places otherwise the database becomes inconsistent.

If Alice changes her email, we must update it in multiple rows:

| order_id | customer_name | customer_email     | product_name | product_price | quantity |
| -------- | ------------- | ------------------ | ------------ | ------------- | -------- |
| 1        | Alice         | alice_new@a.com 🆕 | Orange       | 3.00          | 1        |
| 2        | Bob           | bob@a.com          | Banana       | 2.00          | 2        |
| 3        | Alice         | alice_new@a.com 🆕 | Apple        | 4.00          | 1        |
| 4        | Alice         | alice@a.com 🚨     | Grape        | 5.00          | 3        |

This can lead to inconsistencies if we forget to update one of the rows.

---

## Insertion Anomaly Example

An insertion anomaly occurs when we cannot add data due to missing related data.

We cannot add a new customer without placing an order:

| order_id   | customer_name | customer_email | product_name | product_price | quantity   |
| ---------- | ------------- | -------------- | ------------ | ------------- | ---------- |
| no data 🚨 | Charlie       | charlie@a.com  | no data 🚨   | no data 🚨    | no data 🚨 |

Also we cannot add a new product without an order:

| order_id   | customer_name | customer_email | product_name | product_price | quantity   |
| ---------- | ------------- | -------------- | ------------ | ------------- | ---------- |
| no data 🚨 | no data 🚨    | no data 🚨     | Mango        | no data 🚨    | no data 🚨 |

Even if we could set those fields to `NULL`, it leads to meaningless records. The logical correctness would be compromised e.g. a customer exists independently of orders in an `Order` table. Furthermore, primary key constraints may prevent `NULL` values.

Hence an insertion anomaly still exists even if we allow `NULL` values, because the schema does not accurately represent the real-world entities and their relationships.

---

## Deletion Anomaly Example

A deletion anomaly occurs when deleting a record inadvertently removes other valuable data.

If Bob cancels his order and we delete his record:

| order_id   | customer_name | customer_email | product_name | product_price | quantity   |
| ---------- | ------------- | -------------- | ------------ | ------------- | ---------- |
| no data 🚨 | no data 🚨    | no data 🚨     | no data 🚨   | no data 🚨    | no data 🚨 |

Then we lose all information about Bob, including his email, which may be needed for future marketing or communication. The product information is also lost, which may be relevant for inventory or sales analysis.

---

## The Normal Forms

Normalization of data is a process of analyzing the given relation schemas to achieve:

1. Minimal redundancy - ensuring that each piece of information is stored in only one place.
1. Minimal insertion, deletion, and update anomalies - ensuring that changes to the data do not lead to inconsistencies or loss of information.

The process involves applying a series of rules called "normal forms". The most commonly used normal forms are:

- **First Normal Form (1NF):**
  - Each cell contains only one atomic value (no multi-valued cells)
  - Each row is unique
- **Second Normal Form (2NF):**
  - Be in 1NF
  - Every non-key attribute is fully functionally dependent on the entire primary key (no partial dependencies)
- **Third Normal Form (3NF):**
  - Be in 2NF
  - No transitive dependencies — non-key attributes must depend only on the primary key, not on other non-key attributes

By applying these normal forms, we can create a well-structured database that minimizes redundancy and maintains data integrity.

### 1NF

**Rule**

- Each cell contains only one atomic value (no multi-valued cells)
- Each row is unique.

Assume the following is our initial raw data table, where each order can contain multiple products stored as comma-separated values:

| order_id | customer_name | customer_email | products             | product_prices   | quantities |
| -------- | ------------- | -------------- | -------------------- | ---------------- | ---------- |
| 1        | Alice         | alice@a.com    | Orange, Apple, Grape | 3.00, 4.00, 5.00 | 1, 1, 3    |
| 2        | Bob           | bob@a.com      | Banana               | 2.00             | 2          |

The `Orders` table is currently not in 1NF. The `products`, `product_prices`, and `quantities` columns each contain multiple comma-separated values in a single cell - violating the rule that each cell must hold only one atomic value.

To achieve 1NF, we expand each product into its own row:

| order_id | customer_name | customer_email | product_name | product_price | quantity |
| -------- | ------------- | -------------- | ------------ | ------------- | -------- |
| 1        | Alice         | alice@a.com    | Orange       | 3.00          | 1        |
| 1        | Alice         | alice@a.com    | Apple        | 4.00          | 1        |
| 1        | Alice         | alice@a.com    | Grape        | 5.00          | 3        |
| 2        | Bob           | bob@a.com      | Banana       | 2.00          | 2        |

Now the table is in 1NF because each cell contains atomic values and each row is unique, identified by the composite primary key `(order_id, product_name)` because an order can have multiple products. A composite primary key is a primary key that consists of more than one column. In this case, the combination of `order_id` and `product_name` uniquely identifies each row in the table.

### 2NF

The table has potential anomalies still because e.g. if the `product_price` of "Apple" changes, we may have to update multiple rows. If we miss one, we get inconsistencies.

**Rule**

- Be in 1NF.
- Every non-key attribute is fully functionally dependent on the entire primary key (no partial dependencies).

A **functional dependency** is when one attribute uniquely determines another attribute e.g. `order_id` -> `customer_name` means that the order ID uniquely determines the customer name because each order is placed by a single customer. Knowing the order ID tells us who the customer is. We say that `customer_name` is functionally dependent on `order_id`.

Similarly, `product_name` -> `product_price` means that the product name uniquely determines the product price because each product has a fixed price. Knowing the product name tells us its price. We say that `product_price` is functionally dependent on `product_name`.

A **partial dependency** occurs when a non-key attribute depends on only part of a composite primary key.

In our table, `quantity` depends on the entire primary key `(order_id, product_name)`, but

- `product_price` depends only on `product_name`.

This is a partial dependency because `product_price` does not depend on the full composite key — knowing only the product name is sufficient to determine its price, regardless of which order it belongs to.

For 2NF, each table should store facts that depend on the entire primary key only.

So we must:

- Move product-related data to a separate `Products` table.

| product_name | product_price |
| ------------ | ------------- |
| Orange       | 3.00          |
| Apple        | 4.00          |
| Grape        | 5.00          |
| Banana       | 2.00          |

Then our `Orders` table becomes:

| order_id | customer_name | customer_email | product_name | quantity |
| -------- | ------------- | -------------- | ------------ | -------- |
| 1        | Alice         | alice@a.com    | Orange       | 1        |
| 1        | Alice         | alice@a.com    | Apple        | 1        |
| 1        | Alice         | alice@a.com    | Grape        | 3        |
| 2        | Bob           | bob@a.com      | Banana       | 2        |

Now both tables are in 2NF because all non-key attributes depend on the entire primary key.

### 3NF

**Rule**

- Be in 2NF.
- No transitive dependencies — non-key attributes must depend only on the primary key, not on other non-key attributes.

A **transitive dependency** occurs when a non-key attribute depends on another non-key attribute. In our `Orders` table, `customer_email` depends on `customer_name`, which is a non-key attribute. This creates a transitive dependency because `customer_email` does not depend directly on the primary key `order_id`.

`order_id` -> `customer_name` -> `customer_email`

To achieve 3NF, we must remove this transitive dependency by creating a separate `Customers` table:

| customer_name | customer_email |
| ------------- | -------------- |
| Alice         | alice@a.com    |
| Bob           | bob@a.com      |

Then our `Orders` table becomes:

| order_id | customer_name | product_name | quantity |
| -------- | ------------- | ------------ | -------- |
| 1        | Alice         | Orange       | 1        |
| 1        | Alice         | Apple        | 1        |
| 1        | Alice         | Grape        | 3        |
| 2        | Bob           | Banana       | 2        |

Now all three tables are in 3NF because all non-key attributes depend only on the primary key.

> **Note:** Strictly speaking, `customer_name` and `customer_email` are also partial dependencies (they depend only on `order_id`, not the full composite key `(order_id, product_name)`), so they should have been resolved during the 2NF step. The transitive dependency `order_id → customer_name → customer_email` is real, but it only becomes the *primary* violation once the table has a single-column primary key. This example intentionally defers the customer data to the 3NF step to illustrate the concept of transitive dependencies — the underlying idea is correct, but the example will be improved later.

---

# A Note on Normalization for App Development vs Data Science

In application development, when we model entities and relationships correctly (ER modeling), we will almost always end up with normalized tables that is close to 3NF.

This happens because ER modeling encourages us to think about entities (tables) and their attributes (columns) separately, leading to a natural separation of concerns. In ER modeling, we ask what thing does an attribute describe? If it describes another entity, it should be in a separate table. This eliminates redundancy and ensures that each fact is stored in exactly one place. The _fact_ refers to a piece of information stored in the database.

Normalization may still be necessary when after designing the ER model, we find that some tables still contain partial or transitive dependencies. However, this is less common if the ER model is well-designed from the start.

But for Data Science, the source data we get may be aggregated or denormalized for performance reasons. Such flat tables compress multiple entities and relationships into a single table e.g.

| order_id | order_date | user_id | user_name | product_id | product_name | quantity | price |
| -------- | ---------- | ------- | --------- | ---------- | ------------ | -------- | ----- |
| 1        | 2023-10-01 | 101     | Alice     | 201        | Apple        | 2        | 4.00  |

We learn normalization not to design databases, but to understand, reason about and reshape flat data for analysis when necessary.
