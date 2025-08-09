# 🍜 Case Study #1: Danny's Diner

![Danny's Diner](https://github.com/pawanU03/dannys-diner-sql-case-study/blob/main/dannys_diner.png)

## 📋 Table of Contents
- [Problem Statement](#-problem-statement)
- [Entity Relationship Diagram](#-entity-relationship-diagram)
- [Dataset](#-dataset)
- [Case Study Questions](#-case-study-questions)
- [Solutions](#-solutions)

---

## 🎯 Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they've spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!

---

## 🗂️ Entity Relationship Diagram

![Entity Relationship Diagram](https://github.com/pawanU03/dannys-diner-sql-case-study/blob/main/Danny's%20Diner.png)

---

## 📊 Dataset

Danny has shared with you 3 key datasets for this case study:

### **Table 1: sales**
The sales table captures all `customer_id` level purchases with an corresponding `order_date` and `product_id` information for when and what menu items were ordered.

| customer_id | order_date | product_id |
| ----------- | ---------- | ---------- |
| A           | 2021-01-01 | 1          |
| A           | 2021-01-01 | 2          |
| A           | 2021-01-07 | 2          |
| A           | 2021-01-10 | 3          |
| A           | 2021-01-11 | 3          |
| A           | 2021-01-11 | 3          |
| B           | 2021-01-01 | 2          |
| B           | 2021-01-02 | 2          |
| B           | 2021-01-04 | 1          |
| B           | 2021-01-11 | 1          |
| B           | 2021-01-16 | 3          |
| B           | 2021-02-01 | 3          |
| C           | 2021-01-01 | 3          |
| C           | 2021-01-01 | 3          |
| C           | 2021-01-07 | 3          |

### **Table 2: menu**
The menu table maps the `product_id` to the actual `product_name` and price of each menu item.

| product_id | product_name | price |
| ---------- | ------------ | ----- |
| 1          | sushi        | 10    |
| 2          | curry        | 15    |
| 3          | ramen        | 12    |

### **Table 3: members**
The final members table captures the `join_date` when a `customer_id` joined the beta version of the Danny's Diner loyalty program.

| customer_id | join_date  |
| ----------- | ---------- |
| A           | 2021-01-07 |
| B           | 2021-01-09 |

---

## ❓ Case Study Questions

Each of the following case study questions can be answered using a single SQL statement:

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

---

## 💡 Solutions

### ✅ 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT 
    s.customer_id, 
    SUM(price) AS amount_each_customer_spent
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id;
```

**Result:**
| customer_id | amount_each_customer_spent |
| ----------- | -------------------------- |
| A           | 76                         |
| B           | 74                         |
| C           | 36                         |

### ✅ 2. How many days has each customer visited the restaurant?

```sql
SELECT
    customer_id,
    COUNT(DISTINCT order_date) AS customer_visit_days
FROM sales
GROUP BY customer_id;
```

**Result:**
| customer_id | customer_visit_days |
| ----------- | ------------------- |
| A           | 4                   |
| B           | 6                   |
| C           | 2                   |

### ✅ 3. What was the first item from the menu purchased by each customer?

```sql
WITH RankedPurchase AS (
	SELECT
		s.order_date,
		s.customer_id,
		m.product_name,
		DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date ASC) AS purchase_rank
	FROM sales s
	JOIN menu m
	ON s.product_id = m.product_id
)
SELECT
    rp.customer_id,
    rp.product_name
FROM RankedPurchase rp
WHERE rp.purchase_rank = 1
GROUP BY rp.customer_id, rp.product_name;
```

**Result:**
| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| A           | curry        |
| B           | curry        |
| C           | ramen        |

### ✅ 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT
    s.product_id,
    m.product_name,
    COUNT(s.product_id) AS purchase_count
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY m.product_name, s.product_id
ORDER BY purchase_count DESC;
```

**Result:**
| product_id | product_name | purchase_count |
| ---------- | ------------ | -------------- |
| 3          | ramen        | 8              |
| 2          | curry        | 4              |
| 1          | sushi        | 3              |

### ✅ 5. Which item was the most popular for each customer?

```sql
WITH most_popular_item AS (
	SELECT
		s.customer_id,
		s.product_id,
		m.product_name,
		COUNT(*) AS purchase_count,
        DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(*) DESC) AS rank_
	FROM
		sales s
	JOIN 
		menu m
	ON s.product_id = m.product_id
	GROUP BY s.customer_id, s.product_id, m.product_name
	ORDER BY s.customer_id, s.product_id DESC
)
SELECT
	mpi.customer_id,
	mpi.product_name,
    	mpi.purchase_count
FROM most_popular_item mpi
WHERE rank_ = 1;
```

**Result:**

| customer_id | product_name | purchase_count |
| ----------- | ------------ | -------------- |
| A           | ramen        | 3              |
| B           | ramen        | 2              |
| B           | curry        | 2              |
| B           | sushi        | 2              |
| C           | ramen        | 3              |

### ✅ 6. Which item was purchased first by the customer after they became a member?

```sql
WITH first_purchased AS (
	SELECT
		s.customer_id,
		me.product_name,
		s.order_date,
		DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rank_
	FROM sales s
	JOIN members m ON s.customer_id = m.customer_id
	JOIN menu me ON me.product_id = s.product_id
	WHERE s.order_date > m.join_date
)
SELECT
	order_date,
	customer_id,
    product_name
FROM first_purchased
WHERE rank_ = 1;
```

**Result:**

| order_date  | customer_id  | product_name   |
| ----------- | ------------ | -------------- |
| 2021-01-10  | A            | ramen          |
| 2021-01-10  | B            | sushi          |

### ✅ 7. Which item was purchased just before the customer became a member?
```sql
WITH before_membership AS (
	SELECT
		s.customer_id,
		me.product_name,
		s.order_date,
		ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rank_
	FROM sales s
	JOIN members m ON s.customer_id = m.customer_id
	JOIN menu me ON me.product_id = s.product_id
	WHERE s.order_date < m.join_date
)
SELECT
	order_date,
	customer_id,
    product_name
FROM before_membership
WHERE rank_ = 1;
```

**Result:**
| order_date  | customer_id  | product_name   |
| ----------- | ------------ | -------------- |
| 2021-01-01  | A            | sushi          |
| 2021-01-04  | B            | sushi          |

✅ 8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT
	s.customer_id,
    COUNT(s.customer_id) AS total_items,
    SUM(m.price) AS amount_spent
FROM sales s
JOIN members me
ON s.customer_id = me.customer_id
JOIN menu m
ON m.product_id = s.product_id
WHERE s.order_date < me.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

**Result:**
| customer_id  | total_items  | amount_spent   |
| ------------ | ------------ | -------------- |
| A  		   | 2            | 25       	   |
| B  		   | 3            | 40 	           |

### 🔄 Questions 9-10: Coming Soon!
*Solutions will be added as I work through each question daily.*

---

*This case study is part of the [8 Week SQL Challenge](https://8weeksqlchallenge.com/case-study-1/) by Data With Danny.*
