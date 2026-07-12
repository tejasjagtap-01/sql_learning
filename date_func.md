### Q1. Show total revenue per month across all orders. Display: month in 'Mon YYYY' format, total revenue, and number of orders that month. Order chronologically.
```
SELECT
TO_CHAR(DATE_TRUNC('month',o.order_date),'Mon YYYY') AS month,
COUNT(o.order_id) AS order_count,
SUM(o.quantity * p.price) AS total_revenue
FROM orders o
JOIN products p ON o.product_id = p.product_id
GROUP BY DATE_TRUNC('month',o.order_date)
ORDER BY DATE_TRUNC('month',o.order_date)
```


### Q2. For each customer who has placed at least one order, show: Customer name, Date of their first order, Date of their most recent order. How many days between first and most recent order (tenure as customer). How many days since their last order (recency). Status: 'Active' if last order within 120 days of most recent order in the system, else 'Lapsed
```
SELECT 
	c.name 												AS customer_name,
	MIN(o.order_date) 									AS first_order,
	MAX(o.order_date) 									As recnt_order,
	MAX(o.order_date) - MIN(o.order_date)				AS customer_tenure_days,
	(SELECT MAX(order_date) FROM orders) 
		- MAX(o.order_date) 							AS days_since_last_order,
	CASE 
		WHEN (SELECT MAX(order_date) FROM orders) 
			- MAX(o.order_date) <= 120 THEN 'Active'
		ELSE 'Lapse'
	END 												AS status
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.name, c.customer_id
ORDER BY days_since_last_order
```


### Q3. Group customers by the quarter they signed up (Q1 2023, Q2 2023 etc.). For each cohort show: Cohort label (e.g. 'Q1-2023'), Number of customers in that cohort. How many of those customers eventually placed an order, Average days from signup to first order (for those who ordered). Conversion rate (ordered / total) as a percentage, Order by cohort chronologically
```
WITH customer_cohorts AS (
    SELECT
        c.customer_id,
        c.name,
        c.signup_date,
        DATE_TRUNC('quarter', c.signup_date)                  AS signup_quarter
    FROM customers c
),
customer_orders AS (
    SELECT
        cc.customer_id,
        cc.signup_quarter,
        MIN(o.order_date) - cc.signup_date                    AS days_to_first_order
    FROM customer_cohorts cc
    LEFT JOIN orders o ON cc.customer_id = o.customer_id
    GROUP BY cc.customer_id, cc.signup_date, cc.signup_quarter
),
cohort_summary AS (
    SELECT
        signup_quarter,
        COUNT(*)                                              AS total_customers,
        COUNT(days_to_first_order)                           AS customers_who_ordered,
        ROUND(AVG(days_to_first_order), 1)                  AS avg_days_to_first_order,
        ROUND(
            COUNT(days_to_first_order) * 100.0
            / COUNT(*),
            2
        )                                                    AS conversion_rate_pct
    FROM customer_orders
    GROUP BY signup_quarter
)
SELECT
    'Q' || EXTRACT(QUARTER FROM signup_quarter)
    || '-' || EXTRACT(YEAR FROM signup_quarter)              AS cohort,
    total_customers,
    customers_who_ordered,
    avg_days_to_first_order,
    conversion_rate_pct
FROM cohort_summary
ORDER BY signup_quarter;
```


### Q4. Find all orders placed in February 2024. Show order ID, customer name, order date, and order amount.
```
SELECT DISTINCT 
	o.order_id,
	c.name AS customer_name,
	o.order_date,
	o.quantity * p.price AS order_amount
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE EXTRACT(MONTH FROM o.order_date) = 2
	AND EXTRACT(YEAR FROM o.order_date) = 2024
ORDER BY o.order_date
```


### Q5. For each customer who has placed an order, show their name, their most recent order date, and how many days have passed since that order. Order by most recent first.
```
SELECT 
	c.name AS customer_name,
	MAX(o.order_date) AS last_order,
	CURRENT_DATE - MAX(o.order_date) AS days_since_last_order
FROM customers c
```
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.name
ORDER BY MAX(o.order_date) DESC
