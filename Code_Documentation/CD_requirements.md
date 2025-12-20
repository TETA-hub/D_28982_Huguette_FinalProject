1️⃣ DDL – CREATE TABLE STATEMENTS
🔹 CUSTOMERS

Stores customer profile and membership details.

Primary Key: customer_id

Unique: email

Business Rule: Membership tier constrained to BRONZE, SILVER, GOLD, PLATINUM

CREATE TABLE customers (
    customer_id NUMBER(10) PRIMARY KEY,
    first_name VARCHAR2(50) NOT NULL,
    last_name VARCHAR2(50) NOT NULL,
    email VARCHAR2(100) UNIQUE NOT NULL,
    phone_number VARCHAR2(15) NOT NULL,
    date_of_birth DATE,
    registration_date DATE DEFAULT SYSDATE,
    membership_tier VARCHAR2(20)
        CHECK (membership_tier IN ('BRONZE','SILVER','GOLD','PLATINUM'))
);

🔹 PURCHASES

Records all customer purchases.

Foreign Key: customer_id → customers(customer_id)

Cascade delete enabled

CREATE TABLE purchases (
    purchase_id NUMBER(10) PRIMARY KEY,
    customer_id NUMBER(10) NOT NULL,
    purchase_date TIMESTAMP DEFAULT SYSTIMESTAMP,
    total_amount NUMBER(10,2) CHECK (total_amount >= 0),
    payment_method VARCHAR2(20)
        CHECK (payment_method IN ('CASH','CARD','MOMO','BANK')),
    description VARCHAR2(200),
    CONSTRAINT fk_purchase_customer
        FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
        ON DELETE CASCADE
);

🔹 REWARDS

Tracks points earned and redeemed.

CREATE TABLE rewards (
    reward_id NUMBER(10) PRIMARY KEY,
    customer_id NUMBER(10) NOT NULL,
    purchase_id NUMBER(10),
    points_earned NUMBER DEFAULT 0,
    points_redeemed NUMBER DEFAULT 0,
    transaction_date TIMESTAMP DEFAULT SYSTIMESTAMP,
    transaction_type VARCHAR2(10)
        CHECK (transaction_type IN ('EARN','REDEEM','ADJUST')),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (purchase_id) REFERENCES purchases(purchase_id)
);

🔹 POINT_RULES

Defines how loyalty points are calculated.

CREATE TABLE point_rules (
    rule_id NUMBER PRIMARY KEY,
    rule_name VARCHAR2(50) UNIQUE NOT NULL,
    amount_threshold NUMBER(10,2),
    points_per_unit NUMBER(5,2),
    effective_date DATE NOT NULL,
    expiry_date DATE
);

🔹 TRANSACTIONS_LOG

Audit table for all database operations.

CREATE TABLE transactions_log (
    log_id NUMBER PRIMARY KEY,
    table_name VARCHAR2(50),
    operation_type VARCHAR2(10),
    record_id NUMBER,
    user_name VARCHAR2(50) DEFAULT USER,
    log_timestamp TIMESTAMP DEFAULT SYSTIMESTAMP,
    old_value CLOB,
    new_value CLOB
);

2️⃣ DML – INSERT SCRIPTS (Sample Data)
INSERT INTO customers VALUES (
    seq_customer_id.NEXTVAL,
    'Alice','Mukamana','alice@gmail.com',
    '0788001122',DATE '1998-05-12',
    SYSDATE,'BRONZE'
);

INSERT INTO point_rules VALUES (
    seq_rule_id.NEXTVAL,
    'STANDARD RULE',
    100,
    1,
    SYSDATE,
    NULL
);

3️⃣ FUNCTIONS (WITH RETURN TYPES)
🔹 fn_calculate_points

Purpose: Calculates loyalty points based on purchase amount.
Return Type: NUMBER

FUNCTION fn_calculate_points(p_amount NUMBER)
RETURN NUMBER;

🔹 fn_get_total_points

Purpose: Returns total available points for a customer.
Return Type: NUMBER

FUNCTION fn_get_total_points(p_customer_id NUMBER)
RETURN NUMBER;

🔹 fn_is_restricted_day

Purpose: Checks if today is a weekday or public holiday.
Return Type: BOOLEAN

FUNCTION fn_is_restricted_day
RETURN BOOLEAN;

4️⃣ PROCEDURES (WITH PARAMETERS)
🔹 pr_record_purchase

Purpose:

Inserts a purchase

Calculates points

Inserts reward record

Parameters:

p_customer_id IN NUMBER

p_amount IN NUMBER

p_payment_method IN VARCHAR2

p_description IN VARCHAR2

🔹 pr_redeem_points

Purpose: Redeems loyalty points for a customer.

Parameters:

p_customer_id IN NUMBER

p_points IN NUMBER

🔹 pr_upgrade_membership

Purpose:
Uses a cursor to upgrade membership tiers based on lifetime points.

5️⃣ PACKAGE (SPECIFICATION & BODY)
🔹 pkg_loyalty (Specification)
CREATE PACKAGE pkg_loyalty IS
    PROCEDURE record_purchase(...);
    PROCEDURE redeem_points(...);
    FUNCTION total_points(p_customer_id NUMBER) RETURN NUMBER;
END pkg_loyalty;

🔹 pkg_loyalty (Body)

Implements all procedures and functions using internal logic and calls.

6️⃣ TRIGGERS
🔹 Restriction & Audit Triggers

Trigger Type: BEFORE INSERT / UPDATE / DELETE

Tables: CUSTOMERS, PURCHASES, REWARDS

Logic:

Blocks operations on weekdays (Mon–Fri)

Blocks operations on public holidays

Logs all attempts into TRANSACTIONS_LOG

Error Message Example:

ORA-20010: Operation denied – allowed only on weekends

7️⃣ ANALYTICS QUERIES

(Window Functions & Aggregations)

🔹 Total Points per Customer
SELECT customer_id,
       SUM(points_earned - points_redeemed) AS total_points
FROM rewards
GROUP BY customer_id;

🔹 Ranking Customers by Loyalty Points (Window Function)
SELECT customer_id,
       SUM(points_earned - points_redeemed) AS total_points,
       RANK() OVER (ORDER BY SUM(points_earned - points_redeemed) DESC) AS rank_no
FROM rewards
GROUP BY customer_id;

🔹 Monthly Sales Trend
SELECT TO_CHAR(purchase_date,'YYYY-MM') AS month,
       SUM(total_amount) AS total_sales
FROM purchases
GROUP BY TO_CHAR(purchase_date,'YYYY-MM')
ORDER BY month;
