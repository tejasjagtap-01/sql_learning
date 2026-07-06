### Q1. Find all orders where the order amount (quantity × price) is greater than the average order amount across all orders
```
SELECT DISTINCT
	o.order_id,
	c.name AS customer_Name,
	p.name AS product_Name,
	o.quantity * p.price AS order_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE o.quantity * p.price > (
				SELECT AVG(o2.quantity * p2.price)
				FROM orders o2
				JOIN products p2 ON o2.product_id = p2.product_id
)
ORDER BY order_amount DESC
```


### Q2. Find all customers who have ordered at least one Electronics product.
```
SELECT DISTINCT
	c.name AS customer_name,
	c.customer_id AS customer_id,
	c.city
FROM customers c
WHERE c.customer_id IN (
		SELECT DISTINCT o.customer_id 
		FROM orders o
		JOIN products p ON o.product_id = p.product_id
		WHERE category ='Electronics'
)
```


### Q3. Show each product with its price, the average product price, and whether it is above or below average.
```
SELECT 
	name AS product_name,
	category,
	price,
	ROUND((SELECT AVG(price) FROM products),2) AS avg_price,
	ROUND(price - (SELECT AVG(price) FROM products),2) AS diff_from_avg,
	CASE 
		WHEN 
		price > (SELECT AVG(price) FROM products) THEN 'Above Average'
		ELSE 'Below Average'
	END AS price_position
FROM products
ORDER BY diff_from_avg DESC;
```


### Q4. Find which product categories generated above-average revenue. Show category and total revenue.
```
SELECT 
	category,
	category_revenue
FROM
(SELECT 
	p.category,
	SUM(o.quantity * p.price) AS category_revenue
FROM orders o
JOIN products p ON o.product_id = p.product_id
GROUP BY p.category)
AS category_totals
WHERE category_revenue > (
	SELECT AVG(cat_rev)
		FROM(
		SELECT SUM(o2.quantity * p2.price) AS cat_rev
		FROM orders o2
		JOIN products p2 ON o2.product_id = p2.product_id
		GROUP BY p2.category)
	AS base_avg
)
ORDER BY category_revenue DESC;
```


### Q5. Find customers who placed more orders than the average number of orders per customer.
```
SELECT * FROM customers
SELECT * FROM orders

SELECT 
	c.name AS customer_name,
	COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id , c.name
HAVING COUNT (o.order_id) > (
SELECT AVG(order_count)
FROM(
SELECT 
	customer_id,
	COUNT(order_id) AS order_count 
FROM orders
GROUP BY customer_id
) AS count_per_customer
)
ORDER BY order_count DESC
```


### Q6. For each order, classify it as 'Above Personal Average' or 'Below Personal Average' based on that customer's own average spend.
```
SELECT * FROM customers
SELECT * FROM orders

SELECT 
	o.order_id,
	c.name AS customer_name,
	p.name AS product_name,
	o.quantity * p.price AS order_amount,
	CASE
		WHEN o.quantity * p.price > (
			SELECT AVG(o2.quantity * p2.price)
			FROM orders o2
			JOIN products p2 ON o2.product_id = p2.product_id
			WHERE o2.customer_id = o.customer_id
		)THEN 'Above Personal Average'
		ELSE 'Below Personal Average'
	END AS personal_avg
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
ORDER BY c.customer_id, order_amount DESC
```


### Q7. Find customers who have NEVER ordered any Electronics product.
```
SELECT DISTINCT
	c.name AS customer_name,
	c.customer_id AS customer_id,
	c.city
FROM customers c
WHERE NOT EXISTS (
		SELECT 1
		FROM orders o
		JOIN products p ON o.product_id = p.product_id
		WHERE o.customer_id = c.customer_id 
		AND category ='Electronics'
)
```

#### Alternate way
```
SELECT DISTINCT
	c.name AS customer_name,
	c.customer_id AS customer_id,
	c.city
FROM customers c
WHERE customer_id NOT IN  (
		SELECT DISTINCT customer_id
		FROM orders o
		JOIN products p ON o.product_id = p.product_id
		WHERE o.customer_id = c.customer_id 
		AND category ='Electronics'
)
```


### Q8. Find the most expensive product in each category — without using window functions.
```
SELECT 
	p.category,
	p.name AS product_name,
	p.price
FROM products p
WHERE p.price = (
	SELECT MAX(p2.price) 
	FROM products p2
	WHERE p2.category = p.category
)
ORDER BY p.price DESC
```


### Q9. For each customer, show their total spend and what percentage of total company revenue they represent
```
SELECT 
	c.name AS customer_name,
	COALESCE (SUM(o.quantity * p.price),0) AS customer_spend ,
	ROUND( 
	COALESCE (SUM(o.quantity * p.price),0) * 100
	/
	(SELECT SUM(o2.quantity * p2.price)
	FROM orders o2
	JOIN products p2 ON o2.product_id = p2.product_id
	),
	2
	) AS revenue_pct
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN products p ON o.product_id = p.product_id 
GROUP BY c.customer_id, c.name
ORDER BY customer_spend DESC;
```


### Q10. Classic question asked at Amazon, Google, Microsoft. For each department, find the second highest salary. If only one employee exists in a department, return NULL
```
SELECT DISTINCT 
	e1.department,
	(
		SELECT MAX(e2.salary)
		FROM employees e2
		WHERE e2.department = e1.department
		AND e2.salary < (
			SELECT MAX(e3.salary)
			FROM employees e3
			WHERE e3.department = e1.department
		)
	)AS second_high_salary
FROM employees e1
ORDER BY e1.department; 
```

#### Alternate way
```
WITH ranked AS(
	SELECT 
		department,
		salary,
		DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
		FROM employees
)
SELECT DISTINCT 
	department,
	MAX(CASE WHEN rnk = 2 THEN salary END) AS second_highest_salary
FROM ranked
GROUP BY department
ORDER BY department;
```

---

### Interview Based Question

### Q11. Find the total revenue per city. Show city and total revenue, ordered highest first.
```
SELECT 
	c.city AS cities,
	SUM(o.quantity * p.price) AS total_revenue
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN products p ON p.product_id = o.product_id
GROUP BY cities
ORDER BY total_revenue DESC
```


### Q12. Show all orders placed by customers from Mumbai. Display customer name, product name, order date, and order status.
```
SELECT DISTINCT
	c.name AS customer_name,
	p.name AS product_name,
	o.order_date,
	o.status
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE c.city = 'Mumbai'
ORDER BY o.order_date
```


### Q13. Find all products that have been ordered more than once in total. Show product name and how many times it was ordered.
```
SELECT 
	p.name AS product_name,
	COUNT(o.order_id) AS times_ordered
FROM products p
JOIN orders o ON p.product_id = o.product_id
GROUP BY p.name
HAVING COUNT(o.order_id) > 1
ORDER BY times_ordered DESC
```


----
