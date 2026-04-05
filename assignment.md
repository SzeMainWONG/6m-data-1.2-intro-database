# **🚀 Advanced Challenge: Architecting "FoodFast"**

Topic: Complex Relationships & Data Integrity

Estimated Time: 45–60 Minutes

## 🎯 What You'll Practice

By completing this assignment you will have practised:
- Designing a multi-table schema from plain-English requirements
- Resolving a many-to-many relationship using a junction table
- Handling historical data integrity (a real-world engineering problem)
- Reading auto-generated SQL and asking informed questions about it

## **🌯 The Scenario**

You have been hired as the Lead Backend Engineer for "FoodFast," a new competitor to UberEats/DoorDash.

The CEO has given you a messy set of requirements on a napkin. Your job is to turn this into a robust Database Schema using dbdiagram.io.

### **The Requirements**

1. **Users:** We have customers who have a Name, Email, and *multiple* saved delivery addresses (Home, Work, etc.).  
2. **Restaurants:** Establishments have a Name, Cuisine Type, and Rating.  
3. **The Menu:** Each restaurant has many Menu Items (e.g., "Burger", "Fries"). Items have a Name and a Price.  
4. **Couriers:** Drivers have a Name and Vehicle Type (Bike, Car).  
5. **Orders:**  
   * An order belongs to **one** Customer.  
   * An order is picked up by **one** Courier.  
   * An order comes from **one** Restaurant.  
   * **CRITICAL:** An order can contain **many** Menu Items (e.g., 2 Burgers \+ 1 Fry).

## **🧠 Challenge 1: The "Many-to-Many" Problem**

In the basic lesson, we learned One-to-Many (One Customer \-\> Many Cars).

However, an Order contains Many Items, and a specific Item (like "Cheeseburger") appears in Many Orders.

Task:

You cannot simply put item\_id in the Orders table (because you might buy 5 items).

You cannot put order\_id in the Items table (because the item is sold to thousands of people).

Solution Required:

Create a Junction Table (often called order\_items or order\_details) to sit between Orders and Menu Items.

## **🕵️ Challenge 2: The "History" Problem (Normalization Trap)**

**Scenario:**

1. On Monday, John buys a "Big Mac" for **$5.00**.  
2. On Tuesday, the Restaurant raises the price of the "Big Mac" to **$6.00**.  
3. On Wednesday, John looks at his receipt from Monday.

The Question:

If your database schema just links to the menu\_items table for the price, John's receipt will now say he paid $6.00 on Monday. This is illegal (fraud).

Task:

Modify your schema to ensure that even if the Restaurant updates the Menu Item price, the historic Order Record remains accurate to what was actually paid at that moment.

<details>
<summary>💡 Hint (try for 5 minutes before opening this)</summary>

Think about **where in your schema a transaction is recorded**. When John places an order, his `order_items` row is created at that exact moment. What if you added an extra column to `order_items` that captured the price *right then*, independent of the `menu_items` table? That way, even if the menu price changes later, the transaction row still holds the original value.

</details>

## **🔮 Challenge 3: Looking Ahead to SQL (DDL)**

In our next lesson, we will write the code to actually create these tables.

dbdiagram.io has a magic feature.

1. Build your diagram.
2. Look for the **"Export"** function (often "Export to PostgreSQL" or "Export to MySQL").
3. Generate the SQL code.

> **If the Export button isn't available on your plan:** Don't worry — manually write a `CREATE TABLE` statement for the `users` table using what you already know. It might look wrong or incomplete, and that's fine. Lesson 1.3 (DDL) will teach you the correct syntax, and you can compare it then.

Task:

Paste the generated code for your Users table below. Try to read it, then open the answers below.

* What does NOT NULL mean?

<details>
<summary>Answer</summary>

`NOT NULL` is a constraint that tells the database this column **must always have a value** — you cannot insert a row and leave this field empty. For example, `email varchar NOT NULL` means every user must have an email address; the database will reject any insert that tries to leave it blank.

</details>

* What does DEFAULT mean?

<details>
<summary>Answer</summary>

`DEFAULT` sets an **automatic fallback value** for a column if you don't provide one when inserting a row. For example, `rating decimal DEFAULT 0` means if you add a new restaurant without specifying a rating, the database fills in `0` automatically. It's a way of making columns optional without allowing them to be `NULL`.

</details>

* Note down 2 questions about anything else in the SQL syntax that puzzles you — you'll get answers when you work through Lesson 1.3 (DDL), which covers exactly this code.


## **✅ Solution Key (Don't peek until you try\!)**
<details>

  <summary>DBML Code Solution</summary>
  
```dbml
Table users {
  id int [pk, increment]
  name varchar
  email varchar
}

// 1. One User has Many Addresses
Table addresses {
  id int [pk]
  user_id int
  street varchar
  city varchar
  type varchar // 'Home', 'Work'
}

Table restaurants {
  id int [pk]
  name varchar
  cuisine varchar
  rating decimal // Added rating as per requirements
}

Table menu_items {
  id int [pk]
  restaurant_id int
  name varchar
  current_price decimal // This is the price TODAY
}

Table couriers {
  id int [pk]
  name varchar
  vehicle varchar
}

Table orders {
  id int [pk]
  user_id int
  courier_id int
  restaurant_id int
  order_time datetime
  total_price decimal
}

// THE JUNCTION TABLE (Many-to-Many Resolver)
Table order_items {
  id int [pk]
  order_id int
  menu_item_id int
  quantity int      // How many did they buy?
  
  // THE FIX FOR CHALLENGE 2:
  // We copy the price at the moment of purchase into this table.
  price_at_purchase decimal 
}

// Relationships
Ref: addresses.user_id > users.id
Ref: menu_items.restaurant_id > restaurants.id
Ref: orders.user_id > users.id
Ref: orders.courier_id > couriers.id
Ref: orders.restaurant_id > restaurants.id

// Linking the Junction Table
Ref: order_items.order_id > orders.id
Ref: order_items.menu_item_id > menu_items.id
```
</details>

---
## **📖 Post-Class Reading: Normalisation Trade-offs**

You’ve normalised the FoodFast schema. But real-world databases aren’t always fully normalised — and for good reason. Work through the two scenarios below, think about your answer, then open the explanation.

**Scenario 1 – Historical Accuracy:**
Imagine the iPhone’s price increases to $1,200 next year. If your `order_line_items` table only stores a FK to `menu_items` (not the actual price paid), what happens when you try to calculate last year’s revenue?

**Scenario 2 – Query Complexity:**
Assuming you *did* store historical prices in `order_line_items`, how many table joins would you need to answer "Total revenue by product last year"? What might that mean for a database with millions of rows?

Understand the trade-offs of full normalisation:

<details>
<summary>Scenario 1 answer — What happens to last year’s revenue?</summary>

Your query joins `orders → order_line_items → menu_items` to get the price. But `menu_items.current_price` is now $1,200. The join returns $1,200 for every iPhone sold last year — even though customers actually paid $1,000. **Your revenue report is now wrong, and historic receipts are inaccurate.** This is why we added `price_at_purchase` to `order_items` in Challenge 2: transactions must snapshot the price at the moment of purchase, not reference a live price that can change.

Fully normalised tables store each fact in exactly one place, which is great for avoiding duplicates — but “current price” and “price paid” are two different facts that both deserve their own column.

</details>

<details>
<summary>Scenario 2 answer — How complex does the query get?</summary>

To answer “total revenue by product last year” you’d need something like:

`orders → order_line_items → menu_items`

That’s 3 tables joined. As your database grows to millions of rows, every join multiplies the work the database has to do. This is manageable for transactional queries (looking up one order), but slow for analytical queries that scan entire tables.

This is why analytics systems like data warehouses often **deliberately denormalise** — they store `product_name` and `price_at_purchase` directly in the transaction row, sacrificing some storage efficiency to make reporting queries fast and simple.

</details>

<details>
<summary>The underlying principle — Structural correctness vs. practical trade-offs</summary>

Fully normalised tables guarantee **structural correctness**: no duplicate customers, consistent foreign key references, each fact in one place. But real systems often accept controlled denormalisation for two reasons:

- **Historical accuracy:** Storing `sold_price` directly in the transaction row preserves what actually happened, regardless of future price changes.
- **Query efficiency:** Fewer joins means faster analytics, especially at scale.

Good data design means knowing *when* to normalise strictly and *when* to bend the rules intentionally.

</details>

### Key Takeaways

1. **Normalisation ensures structural correctness.**
2. **Controlled denormalisation preserves history and improves performance.**
3. **Good data design balances integrity with practical business needs.**
