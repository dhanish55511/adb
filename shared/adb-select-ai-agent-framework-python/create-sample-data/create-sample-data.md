# Create Sample Data for Select AI Agent Framework Sequential Pattern Using Python

## Introduction

This lab focuses on creating sample database tables and loading sales and return data for building a Sales Return Agent using the Select AI Agent Framework Sequential pattern with Python.

Estimated Time: 15 minutes

### Objectives

In this lab, you will:

- Create sample tables for customers, products, orders, order items, and returns.

- Insert sample sales and return data.

- Verify that the data was loaded successfully.

- Prepare the data for sequential agent-task workflows using Python.

## Task 1: Create the Sample Tables

Create the following tables for the workshop:

- CUSTOMERS stores customer profile and contact details, including name, email address, phone number, state, and ZIP code.

- `CUSTOMER_ORDER_STATUS` stores customer order details, including order number, product name, and the current order or return status.

Log in to Oracle Autonomous AI Database as `ADB_USER`, then run the following SQL statements.

```sql
<copy>
BEGIN EXECUTE IMMEDIATE 'DROP TABLE customers';
EXCEPTION WHEN OTHERS THEN NULL; END;
/
CREATE TABLE customers (
    customer_id  NUMBER(10) PRIMARY KEY,
    name         VARCHAR2(100),
    email        VARCHAR2(100),
    phone        VARCHAR2(20),
    state        VARCHAR2(2),
    zip          VARCHAR2(10)
);

BEGIN EXECUTE IMMEDIATE 'DROP TABLE customer_order_status';
EXCEPTION WHEN OTHERS THEN NULL; END;
/
CREATE TABLE customer_order_status (
    customer_id     NUMBER(10),
    order_number    VARCHAR2(20),
    status          VARCHAR2(30),
    product_name    VARCHAR2(100)

);
</copy>
```

## Task 2: Load Sample Customer and Customer order status Data

Insert sample customer records and customer order-status records for the Sales Return Agent. The data includes customer contact details, order numbers, purchased products, and order statuses such as delivered, pending delivery, pending return, and returned.

Log in to Oracle Autonomous AI Database as ADB_USER, then run the following SQL statements.

```sql
<copy>
INSERT INTO customers (customer_id, name, email, phone, state, zip) VALUES
(1, 'Alice Thompson', 'alice.thompson@example.com', '555-1234', 'NY', '10001'),
(2, 'Bob Martinez', 'bob.martinez@example.com', '555-2345', 'CA', '94105'),
(3, 'Carol Chen', 'carol.chen@example.com', '555-3456', 'TX', '73301'),
(4, 'David Johnson', 'david.johnson@example.com', '555-4567', 'IL', '60601'),
(5, 'Eva Green', 'eva.green@example.com', '555-5678', 'FL', '33101');
Commit;
INSERT INTO customer_order_status (customer_id, order_number, status, product_name) VALUES
(2, '7734', 'delivered', 'smartphone charging cord'),
(1, '4381', 'pending_delivery', 'smartphone protective case'),
(2, '7820', 'delivered', 'smartphone charging cord'),
(3, '1293', 'pending_return', 'smartphone stand (metal)'),
(4, '9842', 'returned', 'smartphone backup storage'),
(5, '5019', 'delivered', 'smartphone protective case'),
(2, '6674', 'pending_delivery', 'smartphone charging cord'),
(1, '3087', 'returned', 'smartphone stand (metal)'),
(3, '7635', 'pending_return', 'smartphone backup storage'),
(4, '3928', 'delivered', 'smartphone protective case'),
(5, '8421', 'pending_delivery', 'smartphone charging cord'),
(1, '2204', 'returned', 'smartphone stand (metal)'),
(2, '7031', 'pending_delivery', 'smartphone backup storage'),
(3, '1649', 'delivered', 'smartphone protective case'),
(4, '9732', 'pending_return', 'smartphone charging cord'),
(5, '4550', 'delivered', 'smartphone stand (metal)'),
(1, '6468', 'pending_delivery', 'smartphone backup storage'),
(2, '3910', 'returned', 'smartphone protective case'),
(3, '2187', 'delivered', 'smartphone charging cord'),
(4, '8023', 'pending_return', 'smartphone stand (metal)'),
(5, '5176', 'delivered', 'smartphone backup storage');
Commit;
</copy>
```

Run SQL queries to confirm that records exist in each table and that the data is ready for building the Sales Return Agent.

```sql
<copy>
SELECT * FROM customers;
SELECT * FROM customer_order_status;
</copy>
```

You may now proceed to the next lab.

## Learn More

* [Select AI Agent Framework Interfaces](https://docs.oracle.com/en/database/oracle/oracle-database/26/selai/select-ai-agent-interfaces.html)
* [Examples of Using the Select AI Agent Framework](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/examples-using-select-ai-agent.html)

## Acknowledgements

- **Created By:** Chandrakanth Putha, Oracle AI Database Product Manager
- **Contributors:** Mark Hornick, Product Management for Select AI and Machine Learning
- **Last Updated By:** Dhanish Kumar, Senior Member of Technical Staff
