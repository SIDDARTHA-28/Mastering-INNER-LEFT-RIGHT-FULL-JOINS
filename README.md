# Mastering-INNER-LEFT-RIGHT-FULL-JOINS
Intern can combine data across multiple tables efficiently
-- =============================================================
-- Mastering SQL Joins: INNER, LEFT, RIGHT, FULL OUTER
-- Sample scenario: Customers and their Orders
-- =============================================================

-- 1. Create two simple tables
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS orders;

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(50),
    city VARCHAR(50)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product VARCHAR(50),
    amount DECIMAL(10,2)
);

-- 2. Insert sample data (some customers have no orders, some orders have no matching customer)
INSERT INTO customers (customer_id, customer_name, city) VALUES
(1, 'Alice', 'Hyderabad'),
(2, 'Bob', 'Bangalore'),
(3, 'Charlie', 'Chennai'),
(4, 'David', 'Mumbai'),          -- no orders
(5, 'Eve', 'Pune');               -- no orders

INSERT INTO orders (order_id, customer_id, product, amount) VALUES
(101, 1, 'Laptop', 85000.00),
(102, 2, 'Phone', 45000.00),
(103, 1, 'Tablet', 32000.00),
(104, 3, 'Monitor', 18000.00),
(105, NULL, 'Keyboard', 2500.00),   -- orphan order (no customer)
(106, 99, 'Mouse', 1200.00);        -- orphan order (non-existing customer_id 99)

-- Quick look at the raw tables
SELECT 'Customers table' AS note;
SELECT * FROM customers ORDER BY customer_id;

SELECT 'Orders table' AS note;
SELECT * FROM orders ORDER BY order_id;

-- =============================================================
-- JOIN TYPES DEMONSTRATION
-- =============================================================

-- A. INNER JOIN
-- Only matching records from both tables
SELECT 
    c.customer_id,
    c.customer_name,
    c.city,
    o.order_id,
    o.product,
    o.amount
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_id;

-- Comment: Only customers who placed orders appear (Alice, Bob, Charlie)

-- B. LEFT JOIN (LEFT OUTER JOIN) — same thing
-- All records from LEFT table (customers) + matching from right
-- Customers without orders → NULL in order columns
SELECT 
    c.customer_id,
    c.customer_name,
    c.city,
    o.order_id,
    o.product,
    o.amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_id;

-- Comment: David and Eve appear with NULL orders

-- C. RIGHT JOIN (RIGHT OUTER JOIN)
-- All records from RIGHT table (orders) + matching from left
-- Orders without customers → NULL in customer columns
SELECT 
    c.customer_id,
    c.customer_name,
    c.city,
    o.order_id,
    o.product,
    o.amount
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id
ORDER BY o.order_id;

-- Comment: Orders 105 and 106 appear (with NULL customer info)

-- D. FULL OUTER JOIN
-- MySQL does NOT support FULL OUTER JOIN natively
-- Workaround: LEFT JOIN UNION RIGHT JOIN + remove duplicates if needed
-- (We use UNION to combine both directions)

SELECT 
    c.customer_id,
    c.customer_name,
    c.city,
    o.order_id,
    o.product,
    o.amount,
    'from LEFT JOIN' AS source
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id

UNION ALL

SELECT 
    c.customer_id,
    c.customer_name,
    c.city,
    o.order_id,
    o.product,
    o.amount,
    'from RIGHT JOIN' AS source
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id

ORDER BY customer_id, order_id;

-- Alternative cleaner version (most common pattern):
-- Exclude the pure inner matches from one side to avoid duplication
SELECT 
    c.customer_id,
    c.customer_name,
    c.city,
    o.order_id,
    o.product,
    o.amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id

UNION

SELECT 
    c.customer_id,
    c.customer_name,
    c.city,
    o.order_id,
    o.product,
    o.amount
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.customer_id IS NULL   -- ← this prevents duplicating the matched rows

ORDER BY customer_id, order_id;

-- Comment: You now see:
--   • All customers (even without orders)
--   • All orders (even without customers)
--   • Matching rows appear only once

-- =============================================================
-- Quick Summary Table (Venn diagram in words)
-- =============================================================
/*
INNER JOIN     → Only overlapping part
LEFT JOIN      → Left circle + overlap
RIGHT JOIN     → Right circle + overlap
FULL OUTER JOIN→ Both full circles (MySQL: LEFT + RIGHT with UNION)
*/
