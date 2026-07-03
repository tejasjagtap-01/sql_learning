# Tables
---

## 1. Customers
```
CREATE TABLE customers (
  customer_id INT,
  name        VARCHAR(50),
  city        VARCHAR(50),
  signup_date DATE
);
```

## Products
```
CREATE TABLE products (
  product_id  INT,
  name        VARCHAR(50),
  category    VARCHAR(50),
  price       DECIMAL(10,2)
);
```

## Orders
```
CREATE TABLE orders (
  order_id    INT,
  customer_id INT,
  product_id  INT,
  quantity    INT,
  order_date  DATE,
  status      VARCHAR(20)
);
```

## Employees (for self join & cross join questions)
```
CREATE TABLE employees (
  emp_id      INT,
  name        VARCHAR(50),
  department  VARCHAR(50),
  manager_id  INT,        -- references emp_id of their manager
  salary      INT
);
```
