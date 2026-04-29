# DBML Guide (Database Markup Language)

## Overview

DBML (Database Markup Language) is a simple, readable domain-specific language designed to define and document database structures. It was created by the team at Holistics to make database schema design more accessible and collaborative.

## DBML vs SQL

DBML serves the same purpose as SQL DDL (Data Definition Language) statements - defining database schemas - but with a simpler, more human-readable syntax.

**SQL DDL Example:**

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
    id INTEGER PRIMARY KEY,
    user_id INTEGER NOT NULL,
    title VARCHAR(200),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**Same Schema in DBML:**

```dbml
Table users {
  id integer [pk]
  username varchar(50) [not null, unique]
  email varchar(255) [not null, unique]
  created_at timestamp [default: `now()`]
}

Table posts {
  id integer [pk]
  user_id integer [not null]
  title varchar(200)
}

Ref: posts.user_id > users.id
```

**Key Differences:**

- **DBML** is a design and documentation tool - cleaner syntax, database-agnostic
- **SQL DDL** is executable code - database-specific, more verbose
- DBML can be **converted to SQL** for any database system (PostgreSQL, MySQL, SQL Server, etc.)
- DBML focuses on **schema design**, while SQL handles the actual database creation

**Workflow:**

1. Design your schema in DBML
2. Convert DBML to SQL for your target database
3. Execute the SQL to create your database

## Key Features

- **Human-readable syntax**: Easy to write and understand
- **Version control friendly**: Plain text format works well with Git
- **Cross-platform**: Can be converted to SQL for various database systems
- **Visualization**: Can generate ER diagrams automatically
- **Documentation**: Serves as living documentation for your database

## Basic Syntax

### Tables

Define tables using the `Table` keyword. The attributes are defined as `column_name data_type [settings]`.

```dbml
Table users {
  id integer [primary key]
  username varchar(50) [not null, unique]
  email varchar(255) [not null, unique]
  created_at timestamp [default: `now()`]
  status varchar(20) [default: 'active']

  Note: 'Users table to store user information'
}
```

- `id` is an integer and the primary key (PK) of the `users` table.
- `username` is a variable character string with a maximum length of 50 characters, and it cannot be null and must be unique.
- `email` is a variable character string with a maximum length of 255 characters, and it cannot be null and must be unique.
- `created_at` is a timestamp that defaults to the current time when a new record is created.
- `status` is a variable character string with a maximum length of 20 characters, and it defaults to 'active' if not specified.
- The `Note` provides additional documentation about the table, usually describing its purpose or any important details.
- You can add notes to individual columns as well like `username varchar(50) [not null, unique, note: 'Must be between 3 and 50 characters']`.

This would define a table like this:

| id  | username | email      | created_at          | status |
| --- | -------- | ---------- | ------------------- | ------ |
| 1   | johndoe  | john@a.com | 2024-01-01 10:00:00 | active |

### Data Types

Common data types supported in DBML:

| Data Type         | Description                          | Parameters                                                      | Example Values                     |
| ----------------- | ------------------------------------ | --------------------------------------------------------------- | ---------------------------------- |
| `integer`, `int`  | Whole numbers without decimals       | None                                                            | `42`, `-100`, `0`                  |
| `varchar(n)`      | Variable-length text with max length | `n` = max characters                                            | `'hello'`, `'user@email.com'`      |
| `text`            | Variable-length text, no limit       | None                                                            | Long paragraphs, articles          |
| `boolean`, `bool` | True or false values                 | None                                                            | `true`, `false`                    |
| `date`            | Calendar date only                   | None                                                            | `'2026-01-28'`                     |
| `time`            | Time of day only                     | None                                                            | `'14:30:00'`                       |
| `timestamp`       | Date and time combined               | None                                                            | `'2026-01-28 14:30:00'`            |
| `datetime`        | Alias for timestamp                  | None                                                            | `'2026-01-28 14:30:00'`            |
| `decimal(p,s)`    | Exact decimal numbers                | `p` = precision (total digits)<br/>`s` = scale (decimal places) | `decimal(10,2)` stores `12345.67`  |
| `numeric(p,s)`    | Alias for decimal                    | `p` = precision<br/>`s` = scale                                 | Same as decimal                    |
| `json`            | JSON formatted data                  | None                                                            | `'{"name": "Alice", "age": 30}'`   |
| `jsonb`           | Binary JSON (PostgreSQL)             | None                                                            | More efficient storage than `json` |

**Key Differences:**

- **`varchar(n)` vs `text`**: `varchar` has a length limit (good for controlled fields like usernames), `text` has no limit (good for long content like descriptions)
- **`integer` vs `decimal`**: `integer` for whole numbers, `decimal` for precise decimal values (like prices, measurements)
- **`timestamp` vs `date` vs `time`**: `timestamp` includes both date and time, `date` only stores calendar dates, `time` only stores time of day
- **`json` vs `jsonb`**: `jsonb` is PostgreSQL-specific and stores JSON in binary format for faster operations, `json` stores as plain text
- **`decimal` vs `numeric`**: They are functionally identical; both provide exact decimal precision

### Column Settings

Available column settings in brackets `[...]`:

- **Primary key**: `[primary key]` or `[pk]`
- **Not null**: `[not null]`
- **Unique**: `[unique]`
- **Default value**: `[default: value]`
- **Auto increment**: `[increment]`
- **Note**: `[note: 'description']`

```dbml
Table products {
  id integer [pk, increment]
  name varchar(100) [not null]
  price decimal(10,2) [not null, default: 0.00]
  sku varchar(50) [unique, note: 'Stock Keeping Unit']
}
```

### Relationships

Define relationships between tables using the `Ref` keyword.

Primary keys (PK) and foreign keys (FK) are used to establish relationships:

- **Primary Key (PK):** The unique identifier for a record in a table.
- **Foreign Key (FK):** The reference pointing to someone else's PK.

For example, if we have a `users` table and a `posts` table and we want to establish a relationship where each post belongs to a user, we will need a foreign key in the `posts` table that references the primary key of the `users` table i.e. `user_id` in `posts` references `id` in `users`.

The tables look like this:

**Users Table:**
| id | username | email | created_at | status |
| --- | -------- | ---------- | ------------------- | ------ |
| 1 | johndoe | john@a.com | 2024-01-01 10:00:00 | active |
| 2 | janedoe | jane@a.com| 2024-01-02 11:00:00 | active |

**Posts Table:**
| id | user_id | title |
| --- | ------- | ----- |
| 1 | 1 | "Hello World" |
| 2 | 1 | "My Second Post" |
| 3 | 2 | "Jane's First Post" |

#### One-to-Many / Many-to-One (1:n / n:1)

One-to-Many and Many-to-One are actually the **same relationship** viewed from different perspectives. The syntax is identical - what changes is how you describe it:

```dbml
Table users {
  id integer [pk]
  name varchar(50)
}

Table posts {
  id integer [pk]
  user_id integer [not null]
  title varchar(200)
}

// The arrow always points from FK to PK
Ref: posts.user_id > users.id

// This relationship can be described as:
// - One-to-Many: "One user has many posts" (from users' perspective)
// - Many-to-One: "Many posts belong to one user" (from posts' perspective)
```

**Note**: The `>` symbol means many-to-one - the left side is the "many" side (where the FK lives), and the right side is the "one" side (the PK being referenced). You can also write it as `<` from the PK side (e.g. `users.id < posts.user_id`), but using `>` from the FK side is the more common convention and easier to read.

Another example:

```dbml
Table customers {
  id integer [pk]
  name varchar(100)
}

Table orders {
  id integer [pk]
  customer_id integer [not null]
  total decimal(10,2)
}

// FK > PK (the side with the foreign key is the "many" side)
Ref: orders.customer_id > customers.id
// One customer has many orders / Many orders belong to one customer
```

#### One-to-One (1:1)

```dbml
Table users {
  id integer [pk]
}

Table user_profiles {
  id integer [pk]
  user_id integer [unique]  // The UNIQUE constraint enforces 1:1
}

Ref: user_profiles.user_id - users.id
// The '-' notation documents this as a 1:1 relationship
// But the actual enforcement comes from the UNIQUE constraint on user_id
```

**Important**: The `unique` constraint on `user_id` is what actually enforces the one-to-one relationship at the database level. The `-` symbol is just DBML notation to indicate the relationship type.

#### Many-to-Many (n:n)

Many-to-Many relationships **require a junction table** (also called a join table or associative table) to connect the two entities. This is because you cannot directly model a many-to-many relationship in a relational database.

A Many-to-Many relationship is actually **two Many-to-One relationships** connected through the junction table:

- Many records from the junction table point to one record in the first table
- Many records from the junction table point to one record in the second table

```dbml
Table students {
  id integer [pk]
  name varchar(100)
}

Table courses {
  id integer [pk]
  name varchar(100)
}

// Junction table to connect students and courses
Table enrollments {
  student_id integer [pk]
  course_id integer [pk]
  enrolled_at timestamp
}

// Two Many-to-One relationships:
Ref: enrollments.student_id > students.id  // Many enrollment records point to one student
Ref: enrollments.course_id > courses.id    // Many enrollment records point to one course

// Result: One student can enroll in many courses
//         One course can have many students
```

The junction table has composite primary keys (`student_id`, `course_id`) to ensure each student can enroll in a course only once. Attributes like `enrolled_at` can be added to capture additional information about the relationship.

### Inline Relationships

You can also define relationships inline:

```dbml
Table posts {
  id integer [pk]
  user_id integer [ref: > users.id]
  category_id integer [ref: > categories.id]
}
```

### Indexes

Define indexes for performance optimization:

```dbml
Table users {
  id integer [pk]
  email varchar(255)
  username varchar(50)
  created_at timestamp

  indexes {
    email [unique]
    (username, created_at) [name: 'idx_user_created']
    created_at [type: btree, name: 'idx_created']
  }
}
```

### Enums

Define enumerated types:

```dbml
enum order_status {
  pending
  processing
  shipped
  delivered
  cancelled
}

Table orders {
  id integer [pk]
  status order_status [default: 'pending']
}
```

### Comments and Notes

Add documentation using notes:

```dbml
Table users {
  id integer [pk]
  email varchar(255) [note: 'Must be a valid email format']

  Note: 'This table stores all user accounts.
  Email addresses must be verified before activation.'
}
```

## Tools and Resources

### Online Tools

1. **dbdiagram.io**: Online tool to create database diagrams using DBML
   - Website: https://dbdiagram.io
   - Features: Live preview, export to SQL, PNG, PDF
2. **DBML CLI**: Command-line tool for working with DBML

   ```bash
   npm install -g @dbml/cli

   # Convert DBML to SQL
   dbml2sql schema.dbml --postgres
   dbml2sql schema.dbml --mysql

   # Convert SQL to DBML
   sql2dbml dump.sql --postgres
   ```

3. **VS Code Extensions**:
   - DBML Language Support
   - Database Client

### Exporting to SQL

DBML can be converted to various SQL dialects:

**PostgreSQL:**

```bash
dbml2sql schema.dbml --postgres -o schema.sql
```

**MySQL:**

```bash
dbml2sql schema.dbml --mysql -o schema.sql
```

## References

- Official Documentation: https://dbml.dbdiagram.io/
- GitHub Repository: https://github.com/holistics/dbml
- dbdiagram.io: https://dbdiagram.io
