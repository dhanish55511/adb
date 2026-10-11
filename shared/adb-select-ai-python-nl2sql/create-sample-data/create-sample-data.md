# Create Sample Data for Select AI for Python

## Introduction

This lab creates sample database tables and loads data for the Select AI for Python examples. The dataset represents a simple sales environment with customers, products, orders, order items, and product returns.

You will use this data to try Select AI actions such as generating, running, and explaining SQL; narrating results; chatting with the AI model; and summarizing or translating information.

Estimated time: 15 minutes.

### Objectives

In this lab, you will:

- Create tables for customers, products, orders, order items, and returns.
- Insert sample sales and return data.
- Verify that the data loaded successfully.

## Task 1: Create the Sample Tables

The tables store the following information:

- **CUSTOMERS**: Customer profile and location details.
- **PRODUCTS**: Product catalog information.
- **ORDERS**: Customer order header information.
- **ORDER_ITEMS**: Product-level details for each order.
- **RETURNS**: Product return information.

Sign in as `ADB_USER` and run these SQL statements:

```sql
<copy>
CREATE TABLE customers (
    customer_id NUMBER PRIMARY KEY,
    customer_name VARCHAR2(100),
    city VARCHAR2(50),
    state VARCHAR2(50),
    country VARCHAR2(50),
    customer_segment VARCHAR2(50)
);

CREATE TABLE products (
    product_id NUMBER PRIMARY KEY,
    product_name VARCHAR2(100),
    category VARCHAR2(50),
    unit_price NUMBER(10,2),
    stock_quantity NUMBER
);

CREATE TABLE orders (
    order_id NUMBER PRIMARY KEY,
    customer_id NUMBER REFERENCES customers(customer_id),
    order_date DATE,
    order_status VARCHAR2(30)
);

CREATE TABLE order_items (
    order_item_id NUMBER PRIMARY KEY,
    order_id NUMBER REFERENCES orders(order_id),
    product_id NUMBER REFERENCES products(product_id),
    quantity NUMBER,
    unit_price NUMBER(10,2)
);

CREATE TABLE returns (
    return_id NUMBER PRIMARY KEY,
    order_id NUMBER REFERENCES orders(order_id),
    product_id NUMBER REFERENCES products(product_id),
    return_reason VARCHAR2(100),
    return_status VARCHAR2(30),
    refund_amount NUMBER(10,2),
    return_date DATE
);
</copy>
```

## Task 2: Load Customer Data

Insert customer records with names, locations, and customer segments. Sign in as `ADB_USER` and run:

```sql
<copy>
INSERT INTO customers VALUES (1, 'Ava Smith', 'San Francisco', 'California', 'USA', 'Retail');
INSERT INTO customers VALUES (2, 'Noah Johnson', 'Austin', 'Texas', 'USA', 'Retail');
INSERT INTO customers VALUES (3, 'Mia Chen', 'Seattle', 'Washington', 'USA', 'Enterprise');
INSERT INTO customers VALUES (4, 'Liam Brown', 'Chicago', 'Illinois', 'USA', 'Retail');
INSERT INTO customers VALUES (5, 'Sophia Patel', 'New York', 'New York', 'USA', 'Enterprise');

COMMIT;
</copy>
```

## Task 3: Load Product Data

Insert product names, categories, unit prices, and stock quantities. Sign in as `ADB_USER` and run:

```sql
<copy>
INSERT INTO products VALUES (101, 'Laptop Pro 14', 'Electronics', 1299.00, 25);
INSERT INTO products VALUES (102, 'Wireless Mouse', 'Accessories', 29.99, 150);
INSERT INTO products VALUES (103, 'Office Chair', 'Furniture', 249.00, 40);
INSERT INTO products VALUES (104, 'Noise Canceling Headphones', 'Electronics', 199.00, 60);
INSERT INTO products VALUES (105, 'Standing Desk', 'Furniture', 499.00, 20);

COMMIT;
</copy>
```

## Task 4: Load Order and Order Item Data

Insert customer orders and their corresponding order items. Sign in as `ADB_USER` and run:

```sql
<copy>
INSERT INTO orders VALUES (1001, 1, DATE '2026-01-10', 'Completed');
INSERT INTO orders VALUES (1002, 2, DATE '2026-01-12', 'Completed');
INSERT INTO orders VALUES (1003, 3, DATE '2026-01-15', 'Completed');
INSERT INTO orders VALUES (1004, 4, DATE '2026-01-18', 'Returned');
INSERT INTO orders VALUES (1005, 5, DATE '2026-01-20', 'Completed');

INSERT INTO order_items VALUES (1, 1001, 101, 1, 1299.00);
INSERT INTO order_items VALUES (2, 1001, 102, 2, 29.99);
INSERT INTO order_items VALUES (3, 1002, 103, 1, 249.00);
INSERT INTO order_items VALUES (4, 1003, 104, 2, 199.00);
INSERT INTO order_items VALUES (5, 1004, 105, 1, 499.00);
INSERT INTO order_items VALUES (6, 1005, 101, 2, 1299.00);

COMMIT;
</copy>
```

## Task 5: Load Product Return Data

Insert return records with the reason, status, refund amount, and return date. Sign in as `ADB_USER` and run:

```sql
<copy>
INSERT INTO returns VALUES (1, 1004, 105, 'Changed mind', 'Approved', 499.00, DATE '2026-01-22');
INSERT INTO returns VALUES (2, 1002, 103, 'Damaged item', 'Pending', 249.00, DATE '2026-01-25');

COMMIT;
</copy>
```

## Verify the Sample Data

Run this query as `ADB_USER` to confirm that each table contains the sample records:

```sql
<copy>
SELECT 'CUSTOMERS' AS table_name, COUNT(*) AS row_count FROM customers
UNION ALL
SELECT 'PRODUCTS', COUNT(*) FROM products
UNION ALL
SELECT 'ORDERS', COUNT(*) FROM orders
UNION ALL
SELECT 'ORDER_ITEMS', COUNT(*) FROM order_items
UNION ALL
SELECT 'RETURNS', COUNT(*) FROM returns
ORDER BY table_name;
</copy>
```

The query should report 5 customers, 5 products, 5 orders, 6 order items, and 2 returns. You may now proceed to the next lab.

## Learn More

* [Select AI for Python](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/select-ai-py.html)
* [Examples of Using Select AI](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/select-ai-examples.html)

## Acknowledgements

- **Created By/Date:** Chandrakanth Putha, Oracle AI Database Product Manager
- **Contributors:** Mark Hornick, Product Management for Select AI and Machine Learning
- **Last Updated By:** Dhanish Kumar, Senior Member of Technical Staff
