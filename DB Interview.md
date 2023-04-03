# SQL Interview Questions

## Question 1

### Scenario:
You work for a retail company that has a database containing information about its sales transactions, customers, and products. The company wants to analyze its sales data to identify the top-selling products by category and location. You have been tasked with providing the following information:

1. A list of the top 5 selling products by category in each location.
2. A list of the top 5 selling products overall, along with their category and location.

### Answer:
For the first part of the question, we can use a subquery to generate the list of the top 5 selling products by category in each location. Here's the SQL query:

```SQL
SELECT s.location, p.category, p.product_name, SUM(s.quantity) as total_quantity
FROM sales s
JOIN products p ON s.product_id = p.product_id
GROUP BY s.location, p.category, p.product_name
HAVING (p.category, SUM(s.quantity)) IN 
(
  SELECT p1.category, SUM(s1.quantity) 
  FROM sales s1 
  JOIN products p1 ON s1.product_id = p1.product_id 
  WHERE s1.location = s.location 
  GROUP BY p1.category 
  ORDER BY SUM(s1.quantity) DESC 
  LIMIT 5
)
ORDER BY s.location, p.category, total_quantity DESC;


--For the second part of the question, we can use a similar query but remove the grouping by location and limit the results to the top 5 overall. Here's the SQL query:

SELECT p.category, p.product_name, SUM(s.quantity) as total_quantity, s.location
FROM sales s
JOIN products p ON s.product_id = p.product_id
GROUP BY p.category, p.product_name, s.location
HAVING (SUM(s.quantity)) IN 
(
  SELECT SUM(s1.quantity) 
  FROM sales s1 
  JOIN products p1 ON s1.product_id = p1.product_id 
  GROUP BY p1.product_name, p1.category 
  ORDER BY SUM(s1.quantity) DESC 
  LIMIT 5
)
ORDER BY total_quantity DESC;
```


## Question 2

### Scenario:
You work for a financial services company that has a database containing information about its customers, accounts, and transactions. The company wants to analyze its transaction data to identify patterns and trends in customer behavior. You have been tasked with providing the following information:

1. A list of all customers who have made at least one transaction in each of the past 6 months.
2. A list of all customers who have made transactions on more than one type of account (e.g., checking, savings, credit card).
3. A list of all customers who have made transactions that exceed their account balance.
4. A list of all customers who have made transactions that exceed a certain amount (e.g., $10,000).
5. A list of all customers who have made transactions at unusual times (e.g., outside of normal business hours).
6. A list of all customers who have made transactions from unusual locations (e.g., outside of their home state or country).

### Answer:
1. To get the list of all customers who have made at least one transaction in each of the past 6 months, we can use the following SQL query:

```SQL
SELECT c.customer_id, c.first_name, c.last_name
FROM customers c
WHERE EXISTS
(
  SELECT 1
  FROM transactions t
  WHERE t.customer_id = c.customer_id
  GROUP BY YEAR(t.transaction_date), MONTH(t.transaction_date)
  HAVING COUNT(DISTINCT YEAR(t.transaction_date), MONTH(t.transaction_date)) = 6
);

--To get the list of all customers who have made transactions on more than one type of account, we can use the following SQL query:
SELECT c.customer_id, c.first_name, c.last_name
FROM customers c
JOIN transactions t ON c.customer_id = t.customer_id
JOIN accounts a ON t.account_id = a.account_id
GROUP BY c.customer_id
HAVING COUNT(DISTINCT a.account_type) > 1;


--To get the list of all customers who have made transactions that exceed their account balance, we can use the following SQL query:
SELECT c.customer_id, c.first_name, c.last_name
FROM customers c
JOIN transactions t ON c.customer_id = t.customer_id
JOIN accounts a ON t.account_id = a.account_id
WHERE t.amount > a.balance;


--To get the list of all customers who have made transactions that exceed a certain amount (e.g., $10,000), we can use the following SQL query:
SELECT c.customer_id, c.first_name, c.last_name
FROM customers c
JOIN transactions t ON c.customer_id = t.customer_id
WHERE t.amount > 10000;


--To get the list of all customers who have made transactions at unusual times (e.g., outside of normal business hours), we can use the following SQL query:
SELECT c.customer_id, c.first_name, c.last_name
FROM customers c
JOIN transactions t ON c.customer_id = t.customer_id
WHERE HOUR(t.transaction_time) < 9 OR HOUR(t.transaction_time) > 17;


--To get the list of all customers who have made transactions from unusual locations (e.g., outside of their home state or country), we can use the following SQL query:
SELECT c.customer_id, c.first_name, c.last_name
FROM customers c
JOIN transactions t ON c.customer_id = t.customer_id
JOIN accounts a ON t.account_id = a.account_id
JOIN locations l ON a.location_id = l.location_id
WHERE l.state <>
