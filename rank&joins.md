# Joins & Rank() Functions

---

### Q1. Show each order with the customer name and product name. Include only orders where both customer and product exist.
```
SELECT 
	o.order_id,
	c.name AS Customer_Name,
	p.name AS Product_Name,
	o.quantity,
	o.status
FROM orders AS o
INNER JOIN customers AS c ON o.customer_id = c.customer_id
INNER JOIN products AS p ON o.product_id = p.product_id
ORDER BY o.order_id;
```


### Q2. Show ALL customers and their orders. If a customer has no orders, still show them with NULL in the order columns.
```
SELECT 
	c.customer_id,
	c.name AS Customer_Name,
	c.city,
	o.order_id,
	o.product_id,
	o.quantity,
	o.order_date,
	o.status
FROM customers AS c
LEFT JOIN orders AS o ON c.customer_id = o.customer_id
--WHERE o.customer_id IS NULL
ORDER BY c.customer_id
```

### Q3. Find customers who have NEVER placed an order.
```
SELECT 
	c.customer_id,
	c.name AS Customer_Name,
	c.city,
	o.order_id,
	o.product_id,
	o.quantity,
	o.order_date,
	o.status
FROM customers AS c
LEFT JOIN orders AS o 
ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL
ORDER BY c.customer_id
```


### Q4. Find products that have NEVER been ordered.
```
SELECT 
	p.product_id,
	p.name AS Product_Name,
	p.price,
	p.category,
	o.order_id,
	o.customer_id
FROM orders AS o
RIGHT JOIN products AS p 
ON o.product_id = p.product_id
WHERE o.product_id IS NULL
```

### Q5. Show ALL customers and ALL products in one result, even if there is no relationship between them. Show customer name and product name side by side. NULLs are expected where no match exists.
```
SELECT 
	c.name AS customer_name,
	p.name AS product_name
FROM customers AS c
FULL JOIN products AS p
ON c.customer_id = p.product_id
```


### Q6. For each customer, show their total spend (quantity × price) on Delivered orders only. Include customers with zero delivered orders as 0, not NULL.
```
SELECT 
	c.name AS customer_name,
	COALESCE(SUM(o.quantity * p.price),0) AS total_spend
FROM customers AS c
LEFT JOIN orders AS o 
ON c.customer_id = o.customer_id
AND o.status = 'Delivered'
LEFT JOIN products AS p 
ON o.product_id = p.product_id
GROUP BY c.name,c.customer_id
ORDER BY total_spend DESC;
```


### Q7. Show each employee with their manager's name. The CEO (no manager) should still appear with NULL in the manager column.

```
SELECT 
	e.name AS employee_name,
	e.department,
	e.salary,
	m.name AS manager_name 
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id
ORDER BY e.emp_id;
```


### Q8. Generate all possible combinations of cities and product categories — useful for building a reporting template where you need every city × category combination, even if no sales exist yet.
```
SELECT 
	DISTINCT c.city,
	p.category
FROM customers c
CROSS JOIN products p
ORDER BY c.city, p.category
```


### Q9. For each customer, show every order they placed AND rank their orders by spend (quantity × price) from highest to lowest. Show customer name, product name, order amount, and the rank.
```
SELECT 
	c.name AS customer_name,
	p.name AS product_name,
	o.order_date,
	o.quantity * p.price AS order_amount,
	RANK() OVER (PARTITION BY c.customer_id ORDER BY o.quantity * p.price DESC ) AS spend_rank
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN products p ON o.product_id = p.product_id
ORDER BY  c.customer_id, spend_rank
```


### Q10.
#### i. Calculate total spend per customer (Delivered orders only) 
#### ii. Rank customers by their total spend 
#### iii. Return only the Top 3 spenders with their city and total spend
```
WITH customer_spend AS(
 SELECT
 	c.customer_id, 
	c.name AS customer_name,
	c.city,
	COALESCE (SUM(o.quantity * p.price),0) AS total_spend
 FROM customers c 
 LEFT JOIN orders o ON c.customer_id = o.customer_id
 	AND o.status = 'Delivered'
 LEFT JOIN products p ON o.product_id = p.product_id
 GROUP BY c.customer_id , c.name, c.city
), 
ranked_customers AS (
	SELECT *,
	  DENSE_RANK() OVER (ORDER BY total_spend DESC) AS spend_rank
	FROM customer_spend
)
SELECT customer_name, city, total_spend, spend_rank
FROM ranked_customers
WHERE spend_rank <= 3
ORDER BY spend_rank;
```
