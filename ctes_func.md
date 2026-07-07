### Q1. Using a CTE, find all customers who have placed at least one order. Show customer name and city.
```
WITH customers_with_order AS
(
SELECT DISTINCT
	o.customer_id
FROM orders o 
)
SELECT DISTINCT
	c.name AS customer_name,
	c.city 
FROM customers c
JOIN customers_with_order cwo ON c.customer_id = cwo.customer_id
ORDER BY c.name
```


### Q2. Using a CTE, calculate total revenue per product. Then in the main query show only the product name and revenue.
```
WITH rev AS
(
SELECT 
	p.product_id,
	p.name AS product_name,
	SUM(o.quantity * p.price) AS total_revenue
FROM orders o
JOIN products p ON o.product_id = p.product_id 
GROUP BY p.product_id, p.name
)
SELECT 
	product_name,
	total_revenue
FROM rev
ORDER BY total_revenue DESC
```


### Q3. Earlier you wrote a subquery to find orders above average amount. Now rewrite it using a CTE instead.
```
WITH avg_order AS(
	SELECT ROUND(AVG(o.quantity * p.price),2) AS avg_amount
	FROM orders o
	JOIN products p ON o.product_id = p.product_id
)
, order_details AS(
SELECT 
	o.order_id,
	c.name AS customer_name,
	p.name AS product_name,
	o.quantity * p.price AS order_amount
FROM orders o 
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
)
SELECT 
	od.order_id,
	od.customer_name,
	od.product_name,
	od.order_amount,
	ao.avg_amount
FROM order_details od, avg_order ao
WHERE od.order_amount > ao.avg_amount
ORDER BY order_amount DESC
```


### Q4. Using a CTE with ROW_NUMBER, return only the most recent order per customer. Show customer name, product name, and order date.
```
WITH last_order AS 
(
	SELECT
		o.*,
		ROW_NUMBER() OVER (PARTITION BY o.customer_id ORDER BY o.order_date DESC) AS rn
	FROM orders o
)
SELECT DISTINCT
	c.name AS customer_name,
	p.name AS prodcut_name,
	lo.order_date,
	lo.rn
FROM last_order lo
JOIN customers c ON c.customer_id = lo.customer_id
JOIN products p ON p.product_id = lo.product_id
WHERE lo.rn = 1
```


### Q5. Step 1: calculate total spend per customer. Step 2: find customers whose spend is above the average spend.
```
WITH customer_spend AS (
    SELECT
        c.customer_id,
        c.name                                  AS customer_name,
        COALESCE(SUM(o.quantity * p.price), 0)  AS total_spend
    FROM customers c
    LEFT JOIN orders   o ON c.customer_id = o.customer_id
    LEFT JOIN products p ON o.product_id  = p.product_id
    GROUP BY c.customer_id, c.name
),
avg_spend AS (
    SELECT AVG(total_spend) AS avg_customer_spend
    FROM customer_spend
)
SELECT
    cs.customer_name,
    cs.total_spend,
    ROUND(av.avg_customer_spend, 2) AS avg_spend
FROM customer_spend cs, avg_spend av
WHERE cs.total_spend > av.avg_customer_spend
ORDER BY cs.total_spend DESC;
```


### Q6. Using a CTE, rank customers by total spend. Show customer name, total spend, and their rank. Include all customers even those with zero spend
```
WITH customer_spend AS (
    SELECT
        c.customer_id,
        c.name                                  AS customer_name,
        COALESCE(SUM(o.quantity * p.price), 0)  AS total_spend
    FROM customers c
    LEFT JOIN orders   o ON c.customer_id = o.customer_id
    LEFT JOIN products p ON o.product_id  = p.product_id
    GROUP BY c.customer_id, c.name
)
SELECT 
cs.customer_name,
cs.total_spend,
DENSE_RANK() OVER (ORDER BY total_spend)
FROM customer_spend cs
ORDER BY total_spend
```


### Q7. Step 1: get total revenue per category. Step 2: rank categories by revenue. Step 3: return only the top ranked category.
```
WITH rev_cat AS(
SELECT 
	p.category,
	SUM(o.quantity * p.price) AS total_revenue
FROM orders o
JOIN products p ON o.product_id = p.product_id
GROUP BY p.category
),
ranked_categories AS(
	SELECT
		category,
		total_revenue,
		RANK() OVER (ORDER BY total_revenue DESC) AS rnk
	FROM rev_cat
),
top_category AS(
SELECT * FROM ranked_categories WHERE rnk = 1
)
SELECT category, total_revenue 
FROM top_category
```


### Q8 Find each customer's total spend. Find each customer's number of orders. Combine both into one output showing: customer name, city, total spend, order count, and average spend per order .Order by total spend descending
```
WITH total_spend AS(
	SELECT 
		c.customer_id,
		c.name AS customer_name,
		c.city,
		COALESCE (SUM(o.quantity * p.price),0) AS spend
	FROM customers c 
	LEFT JOIN orders o ON c.customer_id = o.customer_id
	LEFT JOIN products p ON o.product_id = p.product_id 
	GROUP BY c.customer_id, c.name, c.city
),
customers_orders AS(
SELECT
	c.customer_id,
	c.name AS customer_name,
	COUNT(o.order_id) AS order_count
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name
)
SELECT
	ts.customer_name,
	ts.city,
	ts.spend,
	COALESCE(co.order_count, 0) AS order_count,
CASE 
	WHEN COALESCE (co.order_count, 0) = 0 THEN 0
	ELSE ROUND(ts.spend / co.order_count , 2)
END
FROM total_spend ts
LEFT JOIN customers_orders co ON ts.customer_id = co.customer_id
ORDER BY ts.spend DESC
```


### Q11. Show all orders with status 'Pending'. Display order_id, customer_id, product_id, and order_date.
```
WITH pending_orders AS (
SELECT *
FROM orders
WHERE status = 'Pending'
)
SELECT 
	order_id,
	customer_id,
	product_id,
	order_date
FROM pending_orders
```


### Q12. For each customer, show their name and the total number of products they have ordered (count of order rows, including duplicates of the same product).
```
SELECT 
	c.name AS customer_name,
	COUNT(o.order_id) AS total_orders
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.name
ORDER BY total_orders DESC
```


### Q13. Show the top 3 most expensive products. Display product name, category, and price.
```
SELECT 
	name,
	category,
	price
FROM products
ORDER BY price DESC
LIMIT 3
```
