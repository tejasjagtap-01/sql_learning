### Q1. For each order, show the customer name, order date, current order amount, previous order amount, and the change from the previous order. Customers who only have one order should show NULL for previous and change.
```
SELECT 
	c.name 	AS customer_name,
	o.order_date,
	o.quantity * p.price  AS order_amount, 
	LAG(o.quantity * p.price) 
		OVER (PARTITION BY o.customer_id ORDER BY o.order_date) AS previous_order_amount,
	o.quantity * p.price 
		- LAG(o.quantity * p.price) 
		OVER (PARTITION BY o.customer_id ORDER BY o.order_date) AS change_from_prev
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id
JOIN products p ON o.product_id = p.product_id
ORDER BY c.customer_id, o.order_date;
```


### Q2. For each customer order, calculate how many days passed before they ordered again. If it was their last order, show NULL. Show customer name, current order date, next order date, and days gap.
```
SELECT DISTINCT
	c.customer_id,
	c.name,
	o.order_date AS current_order,
	LEAD(o.order_date)
		OVER(PARTITION BY c.customer_id ORDER BY o.order_date) AS next_order,
	LEAD(o.order_date)
		OVER(PARTITION BY c.customer_id ORDER BY o.order_date) 
		-  o.order_date AS days_gap
FROM orders o 
JOIN customers c ON c.customer_id = o.customer_id
ORDER BY c.customer_id, o.order_date
```


### Q3. For each order, show: customer name, order date, current order amount, the amount of their VERY FIRST order ever, and the amount of their MOST RECENT order. All three columns on every row.
```
SELECT DISTINCT
	c.customer_id,
	c.name AS customer_name,
	o.order_date,
	o.quantity * p.price AS curr_order_amount,
	FIRST_VALUE(o.quantity * p.price) OVER (PARTITION BY c.customer_id ORDER BY o.order_id) AS first_order,
	FIRST_VALUE(o.quantity * p.price) OVER (PARTITION BY c.customer_id ORDER BY o.order_id DESC) AS latest_order,
	LAST_VALUE(o.quantity * p.price) OVER (PARTITION BY c.customer_id ORDER BY o.order_id 
		ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) AS recent_order_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
ORDER BY c.customer_id, o.order_date
```


### Q4. Find customers whose most recent order was lower in value than their previous order. These are "declining" customers. Show customer name, their latest order amount, their previous order amount, and the drop amount. Only show customers who have placed at least 2 orders.
```
WITH orders_comparison AS (
	SELECT DISTINCT
		o.customer_id,
		o.order_date,
		o.quantity * p.price AS order_amount,
		LAG(o.quantity * p.price) 
			OVER (PARTITION BY o.customer_id ORDER BY o.order_date) AS prev_amount,
		ROW_NUMBER() OVER(PARTITION BY o.customer_id ORDER BY o.order_date DESC) AS rn
	FROM orders o
	JOIN products p ON o.product_id = p.product_id
	ORDER BY rn
),
	latest_orders AS (
	SELECT *
		FROM orders_comparison
	WHERE rn = 1
		AND prev_amount IS NOT NULL 
	)
SELECT
	c.name AS customer_name,
	lo.order_date AS latest_order_date,
	lo.order_amount AS latest_spend,
	lo.prev_amount AS previous_spend,
	lo.order_amount - lo.prev_amount AS drop_amount
FROM latest_orders AS lo
JOIN customers c ON lo.customer_id = c.customer_id
WHERE lo.order_amount < lo.prev_amount
ORDER BY drop_amount
```


### Q5. Using the monthly_revenue table, calculate for each region: month, revenue, previous month revenue, growth percentage, and classify each month as 'Growth', 'Decline', or 'No Previous Data'. Order by region and month.
```
WITH monthly_comparison AS(
SELECT 
	region,
	month,
	revenue,
	LAG(revenue)
		OVER (PARTITION BY region ORDER BY month) AS prev_revenue,
	ROUND((revenue - LAG(revenue)OVER (PARTITION BY region ORDER BY month)) * 100
	/ NULLIF(LAG(revenue)OVER (PARTITION BY region ORDER BY month),0),
	2
	) AS grwth_pct
FROM monthly_revenue
)
SELECT 
	region,
	TO_CHAR(month,'Mon YYYY') AS month,
	revenue,
	prev_revenue,
	grwth_pct,
	CASE 
		WHEN grwth_pct > 0 THEN 'Growth'
		WHEN grwth_pct < 0 THEN 'Decline'
		WHEN grwth_pct = 0 THEN 'Flat'
		ELSE 'No Previous Data'
	END AS trend
FROM monthly_comparison
ORDER BY region,month 
```


### Q6.  Show each customer's order history with their order date, product name, current order amount, and the product they bought in their PREVIOUS order. If it was their first order, show 'First Order' instead of NULL
```
SELECT DISTINCT
	c.customer_id,
	o.order_id,
	c.name AS customer_name,
	o.order_date,
	p.name AS product_name,
	o.quantity * p.price AS order_amount,
	COALESCE (LAG(p.name) 
		OVER (PARTITION BY c.customer_id ORDER BY o.order_date),
		'First Order'
		)AS previous_order
FROM  customers c
JOIN orders o ON c.customer_id = o.customer_id 
JOIN products p ON o.product_id = p.product_id
ORDER BY c.customer_id, o.order_id
```
