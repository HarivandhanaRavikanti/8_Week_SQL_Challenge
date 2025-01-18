# 8 Week SQL Challenge

## Case Study #1 - Danny's Diner 👨‍🍳

![Danny's Diner](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/danny's%20diner%20image.png)

## Contents

1. [Introduction](#introduction)
2. [Problem Statement](#problem-statement)
3. [Entity Relationship Diagram](#entity-relationship-diagram)
4. [Case Study Questions & Solutions](#case-study-questions-and-solutions)
5. [Bonus Questions & Solutions](#bonus-questions-and-solutions)
6. [Insights from Analysis](#key-insights)

---

## Introduction

In early 2021, Danny follows his passion for Japanese food and opens "Danny's Diner," a charming restaurant offering sushi, curry, and ramen. However, lacking data analysis expertise, the restaurant struggles to leverage the basic data collected during its initial months to make informed business decisions. Danny's Diner seeks assistance in using this data effectively to keep the restaurant thriving.

---

## Problem Statement

Danny aims to utilize customer data to gain valuable insights into their visiting patterns, spending habits, and favorite menu items. By establishing a deeper connection with his customers, he can provide a more personalized experience for his loyal patrons.

He plans to use these insights to make informed decisions about expanding the existing customer loyalty program. Additionally, Danny seeks assistance in generating basic datasets for his team to inspect the data conveniently, without requiring SQL expertise.

The case study revolves around three key datasets:

- **Sales**
- **Menu**
- **Members**

---

## Entity Relationship Diagram

![Entity Relationship Diagram](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/Relationship.png)

---

## Case Study Questions and Solutions

1. **What is the total amount each customer spent at the restaurant?**

```sql
SELECT s.customer_id, SUM(m.price) AS Money_spent 
FROM sales s 
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

![Answer 1](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/qn1.png)

2. **How many days has each customer visited the restaurant?**

```sql
SELECT s.customer_id, COUNT(DISTINCT order_date) AS No_of_days
FROM sales s 
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

![Answer 2](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/qn2.png)

3. **What was the first item from the menu purchased by each customer?**

```sql
WITH cte AS (
  SELECT s.customer_id, s.order_date, m.product_name, 
         ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rownum
  FROM sales s 
  JOIN menu m ON s.product_id = m.product_id
)
SELECT customer_id, product_name 
FROM cte 
WHERE rownum = 1;
```

![Answer 3](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/qn3.png)

4. **What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT m.product_name, COUNT(*) AS most_purchased 
FROM menu m 
JOIN sales s ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY most_purchased DESC
LIMIT 1;
```

![Answer 4](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/qn4.png)

5. **Which item was the most popular for each customer?**

```sql
WITH cte AS (
  SELECT s.customer_id, m.product_name, COUNT(*) AS most_purchased,
         ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY COUNT(m.product_name) DESC) AS rn
  FROM sales s 
  JOIN menu m ON m.product_id = s.product_id
  GROUP BY s.customer_id, m.product_name
)
SELECT customer_id, product_name 
FROM cte
WHERE rn = 1;
```

![Answer 5](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/qn5.png)

6. **Which item was purchased first by the customer after they became a member?**

```sql
SELECT s.customer_id, m.product_name, s.order_date
FROM menu m 
JOIN sales s ON m.product_id = s.product_id
JOIN members mem ON s.customer_id = mem.customer_id
WHERE s.order_date = (
  SELECT MIN(s2.order_date) 
  FROM sales s2 
  WHERE s2.order_date > mem.join_date AND s2.customer_id = s.customer_id
)
ORDER BY s.customer_id;
```

![Answer 6](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/qn6.png)

7. **Which item was purchased just before the customer became a member?**

```sql
WITH cte AS (
  SELECT DISTINCT s.customer_id, m.product_name, s.order_date, 
         ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rn
  FROM menu m
  JOIN sales s ON m.product_id = s.product_id
  JOIN members mem ON s.customer_id = mem.customer_id
  WHERE s.order_date = (
    SELECT MAX(s2.order_date)
    FROM sales s2
    WHERE s2.order_date < mem.join_date AND s2.customer_id = s.customer_id
  )
)
SELECT customer_id, product_name, order_date
FROM cte 
WHERE rn = 1
ORDER BY customer_id;
```

![Answer 7](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/qn7.png)

8. **What is the total items and amount spent for each member before they became a member?**

```sql
WITH cte AS (
  SELECT s.customer_id, COUNT(s.product_id) AS No_Of_Items, SUM(m.price) AS amount_spent
  FROM sales s 
  JOIN menu m ON s.product_id = m.product_id
  JOIN members mem ON mem.customer_id = s.customer_id
  WHERE s.order_date < mem.join_date 
  GROUP BY s.customer_id
)
SELECT customer_id, SUM(No_Of_Items) AS total_items, SUM(amount_spent) AS amount_purchased
FROM cte
GROUP BY customer_id
ORDER BY customer_id;
```

![Answer 8](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/qn8.png)

9. **If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

```sql
SELECT s.customer_id, 
       SUM(CASE WHEN m.product_name = 'sushi' THEN price * 2 ELSE m.price END) * 10 AS total_points
FROM sales s 
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id 
ORDER BY s.customer_id;
```

![Answer 9](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/qn9.png)

10. **In the first week after a customer joins the program (including their join date), they earn 2x points on all items, not just sushi. How many points do customer A and B have at the end of January?**

```sql
SELECT s.customer_id, s.order_date, m.product_name, m.price, 
       CASE 
         WHEN s.customer_id IN (mem.customer_id) AND s.order_date >= mem.join_date THEN 'Y' 
         ELSE 'N' 
       END AS member
FROM sales s 
JOIN members mem ON s.customer_id = mem.customer_id
JOIN menu m ON m.product_id = s.product_id
ORDER BY s.customer_id, s.order_date, m.product_name;
```

![Answer 10](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/qn10.png)

---

## Bonus Questions and Solutions

### Join All The Things

```sql
SELECT s.customer_id, s.order_date, m.product_name, m.price, 
       CASE WHEN s.order_date >= mem.join_date THEN 'Y' ELSE 'N' END AS member
FROM sales s 
LEFT JOIN members mem ON s.customer_id = mem.customer_id
JOIN menu m ON m.product_id = s.product_id
ORDER BY s.customer_id, s.order_date, m.product_name;
```

![Bonus Answer 1](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/bonus%20qn%201.png)

### Rank All The Things

```sql
WITH cte AS (
  SELECT s.customer_id, s.order_date, m.product_name, m.price,
         CASE 
           WHEN mem.join_date > s.order_date THEN 'N'
           WHEN mem.join_date <= s.order_date THEN 'Y'
           ELSE 'N' 
         END AS Member
  FROM sales s
  LEFT JOIN members mem ON s.customer_id = mem.customer_id
  INNER JOIN menu m ON s.product_id = m.product_id
)
SELECT customer_id, order_date, product_name, price, member,
       CASE 
         WHEN member = 'N' THEN NULL
         ELSE RANK() OVER (PARTITION BY customer_id, member ORDER BY order_date)
       END AS ranking
FROM cte;
```

![Bonus Answer 2](https://raw.githubusercontent.com/HarivandhanaRavikanti/8_Week_SQL_Challenge/refs/heads/main/bonus%20qn%202.png)

---

## Insights from Analysis

1. **Customer Spending Analysis:** The total amount spent by each customer at "Danny's Diner" varies widely. Some customers have spent considerably more than others, indicating potentially high-value customers.
2. **Customer Visits:** The number of days each customer visited the restaurant also varies significantly, showing patterns of customer behaviour. Some customers visit frequently, while others visit less often.
3. **First Purchases:** Understanding the first items purchased by each customer can help Danny identify popular entry items to attract new customers.
4. **Most Popular Item:** The most purchased item on the menu is valuable information for Danny. He can use this information to manage his inventory and work on it to gain more revenue.
5. **Member Points:** The points earned by each member can be used to assess their loyalty and plan further rewards and promotions accordingly.
6. **Bonus Points for New Members:** By offering 2x points to new members during their first week, Danny incentivizes more spending, encouraging members to engage more actively with the program.
