### Q1. Show all 8 customers with their total spend and order count. Customers with no orders should show 0 for both columns — not NULL. Order by total spend descending.
```
SELECT 
	c.customer_id,
	c.name AS customer_name,
	COALESCE(SUM (o.quantity * p.price),0) AS total_spend,
	COALESCE(COUNT(o.order_id),0) AS order_count
FROM customers AS c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN products p ON o.product_id = p.product_id
GROUP BY c.customer_id, c.name
ORDER BY total_spend DESC
```


### Q2. Build a data quality audit that flags two types of gaps in the system: Customers who have never placed any order. Products that have never been ordered
```
SELECT DISTINCT
'Inactive User' AS issue_type,
c.name AS entity_name,
c.city AS details,
'Never Placed Orders' AS description
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL

UNION ALL

SELECT DISTINCT
'Dead Stock Product' AS issue_type,
p.name AS entity_name,
p.category AS details,
'Never been Orders' AS description
FROM products p
LEFT JOIN orders o ON p.product_id = o.product_id
WHERE o.product_id IS NULL

ORDER BY issue_type, entity_name
```


### Q3. Using the monthly_revenue table, calculate month-over-month revenue growth percentage per region. Requirements: Use NULLIF to prevent divide-by-zero errors. Use COALESCE to show 'First Month' where no previous month exists. Classify each month as 'Growth', 'Decline', 'Flat', or 'First Month'. Show: region, month, revenue, previous revenue, growth %, and classification
```
WITH monthly_comparison AS (
SELECT 
	region,
	month,
	revenue,
	LAG(revenue) OVER(PARTITION BY region ORDER BY month) AS prev_revenue
FROM monthly_revenue
),
growth_calc AS (
SELECT 
	region,
	month,
	revenue,
	prev_revenue,
	ROUND((revenue - prev_revenue)*100.00 
		/ NULLIF(prev_revenue,0),
		2
	) AS grwth_pct
FROM monthly_comparison
)
SELECT
	region,
	TO_CHAR(month,'Mon YYYY') AS month,
	revenue,
	COALESCE(CAST(prev_revenue AS VARCHAR), 'No Data') AS prev_revenue,
	COALESCE(CAST(grwth_pct AS VARCHAR), 'First Month') AS grwth_pct,
	CASE 
		WHEN prev_revenue IS NULL THEN 'First Month'
		WHEN grwth_pct > 0 THEN 'Growth'
		WHEN grwth_pct < 0 THEN 'Decline'
		ELSE 'Flat'
	END AS classification
FROM growth_calc
ORDER BY region,month;
```


### Q4. Find all customers who have never placed an order. Show their name and city.
```
SELECT DISTINCT
	c.name AS customer_name,
	c.city
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id 
WHERE o.order_id IS NULL
```


### Q5. Show all customers with their total spend. If a customer has never ordered, display 0 instead of NULL. Order by total spend descending.
```
SELECT 
	c.customer_id,
	c.name AS customer_name,
	c.city,
	COALESCE(COUNT(o.order_id)) AS order_count,
	COALESCE(SUM(o.quantity * p.price),0) AS total_spend
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN products p ON o.product_id = p.product_id
GROUP BY c.customer_id, c.name, c.city
ORDER BY order_count DESC
```
