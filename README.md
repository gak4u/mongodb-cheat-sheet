
# MongoDB Aggregations Cheat Sheet

## Basic Commands

- **Database and Collection:**
  - **MySQL:** `USE database;`
  - **MongoDB:** `use database;`

  - **MySQL:** `SELECT * FROM table;`
  - **MongoDB:** `db.collection.find({});`

## SELECT and WHERE

- **MySQL:**
  ```sql
  SELECT * FROM users WHERE age > 30;
  ```
  **MongoDB:**
  ```javascript
  db.users.find({ age: { $gt: 30 } });
  ```

## SELECT with LIMIT and SORT

- **MySQL:**
  ```sql
  SELECT * FROM users WHERE age > 30 ORDER BY name ASC LIMIT 10;
  ```
  **MongoDB:**
  ```javascript
  db.users.find({ age: { $gt: 30 } }).sort({ name: 1 }).limit(10);
  ```

## Aggregations

- **COUNT**

  - **MySQL:**
    ```sql
    SELECT COUNT(*) FROM users;
    ```
  - **MongoDB:**
    ```javascript
    db.users.aggregate([{ $count: "total" }]);
    ```

- **SUM**

  - **MySQL:**
    ```sql
    SELECT SUM(amount) FROM orders;
    ```
  - **MongoDB:**
    ```javascript
    db.orders.aggregate([{ $group: { _id: null, totalAmount: { $sum: "$amount" } } }]);
    ```

- **AVG**

  - **MySQL:**
    ```sql
    SELECT AVG(age) FROM users;
    ```
  - **MongoDB:**
    ```javascript
    db.users.aggregate([{ $group: { _id: null, averageAge: { $avg: "$age" } } }]);
    ```

- **MIN and MAX**

  - **MySQL:**
    ```sql
    SELECT MIN(age), MAX(age) FROM users;
    ```
  - **MongoDB:**
    ```javascript
    db.users.aggregate([{ $group: { _id: null, minAge: { $min: "$age" }, maxAge: { $max: "$age" } } }]);
    ```

## GROUP BY

- **MySQL:**
  ```sql
  SELECT country, COUNT(*) FROM users GROUP BY country;
  ```
  **MongoDB:**
  ```javascript
  db.users.aggregate([
    { $group: { _id: "$country", count: { $sum: 1 } } }
  ]);
  ```

## JOIN (Lookup)

- **MySQL:**
  ```sql
  SELECT u.name, o.amount FROM users u JOIN orders o ON u.id = o.user_id;
  ```
  **MongoDB:**
  ```javascript
  db.users.aggregate([
    {
      $lookup: {
        from: "orders",
        localField: "_id",
        foreignField: "user_id",
        as: "user_orders"
      }
    },
    { $unwind: "$user_orders" },
    { $project: { name: 1, "user_orders.amount": 1 } }
  ]);
  ```

## HAVING

- **MySQL:**
  ```sql
  SELECT country, COUNT(*) as count FROM users GROUP BY country HAVING count > 5;
  ```
  **MongoDB:**
  ```javascript
  db.users.aggregate([
    { $group: { _id: "$country", count: { $sum: 1 } } },
    { $match: { count: { $gt: 5 } } }
  ]);
  ```

## Complex Aggregations

- **MySQL:**
  ```sql
  SELECT department, AVG(salary) as avg_salary FROM employees WHERE age > 30 GROUP BY department ORDER BY avg_salary DESC LIMIT 5;
  ```
  **MongoDB:**
  ```javascript
  db.employees.aggregate([
    { $match: { age: { $gt: 30 } } },
    { $group: { _id: "$department", avg_salary: { $avg: "$salary" } } },
    { $sort: { avg_salary: -1 } },
    { $limit: 5 }
  ]);
  ```

## Pipeline Stages

- **$match:** Filters the documents to pass only those that match the specified condition(s).
- **$group:** Groups input documents by a specified identifier expression and applies the accumulator expression(s) to each group.
- **$project:** Passes along the documents with only the specified fields or newly computed fields.
- **$sort:** Sorts all input documents and returns them to the pipeline in order.
- **$limit:** Limits the number of documents passed to the next stage in the pipeline.
- **$lookup:** Performs a left outer join to another collection in the same database to filter in documents from the “joined” collection for processing.

## Example Aggregation Pipeline

- **Scenario:** Calculate the total sales per product and sort the result in descending order of total sales.

  - **MySQL:**
    ```sql
    SELECT product_id, SUM(sales) as total_sales
    FROM orders
    GROUP BY product_id
    ORDER BY total_sales DESC;
    ```
  - **MongoDB:**
    ```javascript
    db.orders.aggregate([
      { $group: { _id: "$product_id", total_sales: { $sum: "$sales" } } },
      { $sort: { total_sales: -1 } }
    ]);
    ```
