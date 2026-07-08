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

## 2. Products
```
CREATE TABLE products (
  product_id  INT,
  name        VARCHAR(50),
  category    VARCHAR(50),
  price       DECIMAL(10,2)
);
```

## 3. Orders
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

## 4. Employees (for self join & cross join questions)
```
CREATE TABLE employees (
  emp_id      INT,
  name        VARCHAR(50),
  department  VARCHAR(50),
  manager_id  INT,        -- references emp_id of their manager
  salary      INT
);
```

## 5. Monthly Revenue 
```
CREATE TABLE monthly_revenue (
    month   DATE,
    revenue INT,
    region  VARCHAR(50)
);
```

---

### Insert Customers (some will have NO orders — intentional)
```
INSERT INTO customers VALUES
(1,  'Aarav',   'Pune',      '2023-01-15'),
(2,  'Diya',    'Mumbai',    '2023-03-10'),
(3,  'Kabir',   'Delhi',     '2023-05-20'),
(4,  'Meera',   'Pune',      '2023-07-08'),
(5,  'Vihaan',  'Bangalore', '2024-01-01'),
(6,  'Anaya',   'Mumbai',    '2024-02-14'),
(7,  'Ishaan',  'Chennai',   '2024-03-30'),  -- no orders
(8,  'Riya',    'Delhi',     '2024-04-11');  -- no orders
```

### Insert Products (some will never be ordered — intentional)
```
INSERT INTO products VALUES
(101, 'Laptop',     'Electronics', 75000.00),
(102, 'Phone',      'Electronics', 45000.00),
(103, 'Headphones', 'Electronics',  5000.00),
(104, 'Desk Chair', 'Furniture',   12000.00),
(105, 'Notebook',   'Stationery',    200.00),
(106, 'Webcam',     'Electronics',  3500.00);  -- never ordered
```

### Insert Orders
```
INSERT INTO orders VALUES
(1001, 1, 101, 1, '2024-01-10', 'Delivered'),
(1002, 1, 103, 2, '2024-02-15', 'Delivered'),
(1003, 2, 102, 1, '2024-02-20', 'Delivered'),
(1004, 3, 101, 1, '2024-03-05', 'Cancelled'),
(1005, 3, 104, 1, '2024-03-18', 'Delivered'),
(1006, 4, 105, 5, '2024-04-01', 'Delivered'),
(1007, 5, 102, 2, '2024-04-15', 'Pending'),
(1008, 6, 103, 1, '2024-05-01', 'Delivered'),
(1009, 2, 104, 1, '2024-05-10', 'Delivered'),
(1010, 1, 102, 1, '2024-06-01', 'Pending');
```

### Insert Employees
```
INSERT INTO employees VALUES
(1, 'Rajesh',  'Management', NULL,  95000),  -- CEO, no manager
(2, 'Sunita',  'Sales',      1,     60000),
(3, 'Prakash', 'Sales',      1,     58000),
(4, 'Divya',   'Tech',       1,     75000),
(5, 'Nikhil',  'Tech',       4,     65000),
(6, 'Priya',   'Tech',       4,     63000),
(7, 'Amit',    'Sales',      2,     45000),
(8, 'Sneha',   'Tech',       4,     61000);
```


### Insert Into Monthly Revenue
```
INSERT INTO monthly_revenue VALUES
('2024-01-01', 185000, 'North'),
('2024-02-01', 210000, 'North'),
('2024-03-01', 195000, 'North'),
('2024-04-01', 230000, 'North'),
('2024-05-01', 225000, 'North'),
('2024-06-01', 250000, 'North'),
('2024-01-01', 95000,  'South'),
('2024-02-01', 88000,  'South'),
('2024-03-01', 102000, 'South'),
('2024-04-01', 97000,  'South'),
('2024-05-01', 115000, 'South'),
('2024-06-01', 110000, 'South');
```

### Table Previews
1. Customer Table
```
SELECT * FROM customers;
```
2. Products Table
```
SELECT * FROM products;
```
3. Orders Table
```
SELECT * FROM orders;
```
4. Employees Table
```
SELECT * FROM employees;
```
5. Monthly Revenue
```
SELECT * FROM monthly_revenue
```
