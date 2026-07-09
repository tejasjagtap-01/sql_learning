#### Q1. For each order chronologically, show: customer name, order date, order amount, running total revenue to that point, and what percentage of total company revenue has been collected so far. Order by date.
```
WITH order_amount AS (
SELECT DISTINCT 
	o.order_id,
	o.order_date,
	c.name AS customer_name,
	o.quantity * p.price AS order_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = o.product_id
)
SELECT 
	order_id,
	customer_name,
	order_date,
	SUM(order_amount) 
		OVER (ORDER BY order_date) AS running_total,
	ROUND(SUM(order_amount) OVER (ORDER BY order_date) * 100 
	/ SUM(order_amount) OVER() 
	,2
	) AS cumulative_pct
FROM order_amount
ORDER BY order_date;
```


#### Q2. Using the monthly_revenue table, for each region and month show: actual revenue, 3-month moving average, and whether that month was 'Above Trend', 'Below Trend', or 'Insufficient Data' (first month only has 1 data point, second has 2).
```
WITH moving_avg_cal AS(
SELECT 
	region,
	month,
	revenue,
	ROUND(AVG(revenue) OVER(PARTITION BY region ORDER BY month
		ROWS BETWEEN 2 PRECEDING  AND CURRENT ROW), 2) AS moving_avg_pc,
	COUNT(*) OVER(PARTITION BY region ORDER BY month 
		ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) AS monthly_rev
FROM monthly_revenue
)
SELECT 
	region,
	revenue,
	TO_CHAR(month,'MON YYYY') AS month,
	moving_avg_pc,
	CASE
		WHEN monthly_rev < 3 THEN 'Insufficient data'
		WHEN revenue > monthly_rev THEN 'Above Trend'
		WHEN revenue < monthly_rev THEN 'Below Trend'
		ELSE 'On Trend'
END AS trend
FROM moving_avg_cal
ORDER BY revenue , month ;
```


#### Q3. This is a real Microsoft/Amazon interview question. For each employee show: name, department, salary, running total of salaries in their department (ordered salary low to high), their salary as a percentage of their department's total, and cumulative department percentage. Order by department and salary.
```
WITH salary_analysis AS(
SELECT 
	name,
	department,
	salary,
	SUM(salary) OVER(PARTITION BY department ORDER BY salary
		ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS dept_running_avg,
	SUM(salary) OVER(PARTITION BY department) AS dept_total
FROM employees
)
SELECT 
	name AS employee_name,
	department, 
	salary,
	dept_running_avg,
	ROUND(salary * 100.0 / dept_total, 2) AS pct_of_dept,
	ROUND(dept_running_avg * 100.0 / dept_total,2) AS cumulative_dept_pct
FROM salary_analysis
ORDER BY department, salary
```


#### Q4. Show each order with the order date, order amount, and a running total of revenue as orders come in chronologically. No customer filter — all orders.
```
SELECT 
	o.order_date,
	o.quantity * p.price AS order_amount,
	SUM(o.quantity * p.price) OVER(ORDER BY o.order_date) AS running_totl_rev
FROM orders o 
JOIN products p ON o.product_id = p.product_id
ORDER BY o.order_date
```

#### Q5. Show each order with the customer name, order amount, and what percentage that single order contributes to total company revenue. Round to 2 decimal places.
```
SELECT DISTINCT
	c.name AS customer_name,
	o.order_date,
	o.quantity * p.price AS order_amount,
	ROUND(o.quantity * p.price *100 
	/ SUM(o.quantity * p.price) OVER(), 2) AS pct_of_total
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
JOIN products p ON p.product_id = o.product_id
ORDER BY o.order_date
```
