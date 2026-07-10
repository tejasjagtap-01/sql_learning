### Q1. Create a unified report combining two event types: delivered orders (label as 'Completed Sale') and cancelled orders (label as 'Lost Sale'). Show: customer name, product name, order date, order amount, and event label. Order by date.
```
SELECT
	c.name AS customer_name,
	p.name AS product_name,
	o.order_date,
	o.quantity * p.price AS order_amount,
	'Completed Sales' AS event_type
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE o.status = 'Delivered'

UNION

SELECT
	c.name AS customer_name,
	p.name AS product_name,
	o.order_date,
	o.quantity * p.price AS order_amount,
	'Lost Sale' AS event_type
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE o.status = 'Cancelled'

ORDER BY order_date
```


### Q2. Find customers who have ordered Electronics products but have NEVER ordered any non-Electronics product. In other words, customers who exclusively shop in the Electronics category.
```
SELECT DISTINCT customer_id FROM orders o
JOIN products p ON o.product_id = p.product_id
WHERE p.category = 'Electronics'

EXCEPT

SELECT DISTINCT customer_id FROM orders o
JOIN products p ON o.product_id = p.product_id
WHERE p.category != 'Electronics'
```


#### AlternateWay
```
WITH electronics_only AS(
	SELECT DISTINCT
		o.customer_id 
	FROM orders o 
	JOIN products p ON o.product_id = p.product_id
	WHERE p.category = 'Electronics'
EXCEPT
	SELECT DISTINCT
		o.customer_id 
	FROM orders o 
	JOIN products p ON o.product_id = p.product_id
	WHERE p.category != 'Electronics'
)
SELECT DISTINCT
	c.customer_id,
	c.name AS customer_name,
	c.city
FROM customers c
JOIN electronics_only eo ON c.customer_id = eo.customer_id
ORDER BY c.name
```


### Q3. Find "high-value repeat customers" — customers who appear in BOTH the top 3 spenders by total revenue AND have placed more than one order. Show their name, city, total spend, and order count.
```
WITH top_spenders AS(
SELECT
	customer_id,
	SUM(quantity * price) AS total_spend,
	DENSE_RANK() OVER(ORDER BY SUM(quantity * price) DESC) AS spend_ranking
FROM orders o
JOIN products p ON o.product_id = p.product_id
GROUP BY customer_id
),
top_3 AS (
SELECT 
	customer_id 
FROM top_spenders
WHERE spend_ranking <= 3
),
repeat_customers AS(
SELECT
	customer_id
FROM orders 
GROUP BY customer_id
HAVING COUNT(order_id) > 1
),
high_val_repeat AS (
SELECT 
	customer_id 
FROM top_3
INTERSECT 
SELECT
	customer_id 
FROM repeat_customers
)
SELECT 
	c.name AS customer_name,
	c.city,
	ts.total_spend,
	COUNT(o.order_id) AS order_count
FROM high_val_repeat hvr
JOIN customers c ON hvr.customer_id = c.customer_id
JOIN top_spenders ts On hvr.customer_id = ts.customer_id 
JOIN orders o ON hvr.customer_id = o.customer_id 
GROUP BY c.customer_id, c.name, c.city, ts.total_spend
ORDER BY ts.total_spend DESC
```

#### The Another approach 
```
SELECT
    c.name,
    c.city,
    SUM(o.quantity * p.price)  AS total_spend,
    COUNT(o.order_id)          AS order_count
FROM customers c
JOIN orders o   ON c.customer_id = o.customer_id
JOIN products p ON o.product_id  = p.product_id
GROUP BY c.customer_id, c.name, c.city
ORDER BY total_spend DESC
LIMIT 3;
```


### Q4. Create a single list of all people in the system — customers and employees. Show name, their associated location (city for customers, department for employees), and label each as 'Customer' or 'Employee'. Remove any name duplicates using UNION.
```
SELECT 
	name,
	city,
	'Customer' AS roles
FROM customers

UNION

SELECT 
	name,
	department,
	'Employee' AS roles
FROM employees
ORDER BY name, roles
```

### Q5. Find customers who have a Pending order but have never had a Delivered order.
```
SELECT 
	c.customer_id,
	c.name AS customer_name,
	c.city
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id 
WHERE o.status = 'Pending'

EXCEPT 

SELECT 
	c.customer_id,
	c.name AS customer_name,
	c.city
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id 
WHERE o.status = 'Delivered'
```
