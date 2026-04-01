# **🚀 Advanced Challenge: Architecting "FoodFast"**

Topic: Complex Relationships & Data Integrity

Estimated Time: 45–60 Minutes

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

## **🔮 Challenge 3: Looking Ahead to SQL (DDL)**

In our next lesson, we will write the code to actually create these tables.

dbdiagram.io has a magic feature.

1. Build your diagram.  
2. Look for the **"Export"** function (often "Export to PostgreSQL" or "Export to MySQL").  
3. Generate the SQL code.

Task:

Paste the generated code for your Users table below. Try to read it.

* What does NOT NULL mean?  
* What does DEFAULT mean?  
* Bring 2 questions about this syntax to the next class.


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
## **📖Post Class Reading**
### **3.3 Synthesis** 

### Instructor Prompt

> Our tables are now fully normalised. But let’s explore the following situations:
>
> **Step 1 – Historical Accuracy:**
> Imagine the iPhone’s price increases to $1200 next year.
>
> * What happens if we try to calculate last year’s revenue from our tables?
> * How does full normalisation affect historical reporting?
>
> **Step 2 – Query Complexity:**
> Assuming we **did store historical prices**, how easy is it to answer questions like ‘Total revenue by product last year’ using only fully normalised tables?
>
> * Consider the joins you would need and how this scales with large datasets.

### Learner Goal

Understand the trade-offs of full normalisation:

<details>
<summary>1. Structural correctness ✅</summary>

* Fully normalised tables store each fact in exactly one place.
* Example: No duplicate customers, consistent foreign key references, each order linked to the correct items.
* **Why it matters:** Ensures entity relationships remain consistent, and avoids wasting storage on redundant or repeated data.

</details>

<details>
<summary>2. Historical accuracy ⚠️</summary>

* Fully normalised tables reference **current descriptive data**.
* Updating `ItemPrice` in the `Items` table changes join results, so past revenues can appear wrong.
* **Example:** Joining `Orders → OrderLineItems → Items` today gives $1200 for last year’s iPhone, even though it was sold for $1000.

</details>

<details>
<summary>3. Query efficiency ⚠️</summary>

* Analytical questions often require joining multiple tables.
* **Example:** To calculate “total revenue by product last year” (assuming historical prices are stored), we need to join `Orders → OrderLineItems → Items` for every row.
* Large datasets + multiple joins → slower queries and more complex SQL.

</details>

<details>
<summary>4. Controlled denormalisation</summary>

* To solve these practical issues, systems often **store slowly changing descriptive data directly in transactions**:

  * Example: `sold_price`, `Product category`, `Brand` at time of sale.

* **Benefits:**

  * Preserves historical revenue and product context.
  * Reduces joins → faster queries.
  * Simplifies analytics and reporting.

</details>

### Key Takeaways

1. **Normalisation ensures structural correctness.**
2. **Controlled denormalisation preserves history and improves performance.**
3. **Good data design balances integrity with practical business needs.**
