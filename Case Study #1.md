# Case Study #1 - Danny's Diner

### Introduction

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

### Problem Statement
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!


**Schema (PostgreSQL v13)**

    CREATE SCHEMA dannys_diner;
    SET search_path = dannys_diner;
    
    CREATE TABLE sales (
      "customer_id" VARCHAR(1),
      "order_date" DATE,
      "product_id" INTEGER
    );
    
    INSERT INTO sales
      ("customer_id", "order_date", "product_id")
    VALUES
      ('A', '2021-01-01', '1'),
      ('A', '2021-01-01', '2'),
      ('A', '2021-01-07', '2'),
      ('A', '2021-01-10', '3'),
      ('A', '2021-01-11', '3'),
      ('A', '2021-01-11', '3'),
      ('B', '2021-01-01', '2'),
      ('B', '2021-01-02', '2'),
      ('B', '2021-01-04', '1'),
      ('B', '2021-01-11', '1'),
      ('B', '2021-01-16', '3'),
      ('B', '2021-02-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-01', '3'),
      ('C', '2021-01-07', '3');
     
    
    CREATE TABLE menu (
      "product_id" INTEGER,
      "product_name" VARCHAR(5),
      "price" INTEGER
    );
    
    INSERT INTO menu
      ("product_id", "product_name", "price")
    VALUES
      ('1', 'sushi', '10'),
      ('2', 'curry', '15'),
      ('3', 'ramen', '12');
      
    
    CREATE TABLE members (
      "customer_id" VARCHAR(1),
      "join_date" DATE
    );
    
    INSERT INTO members
      ("customer_id", "join_date")
    VALUES
      ('A', '2021-01-07'),
      ('B', '2021-01-09');

---

__Query #1 What is the total amount each customer spent at the restaurant?__

    SELECT
    	s.customer_id AS Customer,
    	SUM(m.price)
    FROM
    	dannys_diner.sales AS s
    JOIN
    	dannys_diner.menu AS m
    ON
    	s.product_id = m.product_id
    GROUP BY
    	Customer
    ORDER BY
    	Customer;

| customer | sum |
| -------- | --- |
| A        | 76  |
| B        | 74  |
| C        | 36  |

---
__Query #2 How many days has each customer visited the restaurant?__

    SELECT
    	customer_id,
        COUNT(DISTINCT order_date)AS days
    FROM
     	dannys_diner.sales
    GROUP BY
    	customer_id
    ORDER BY
    	customer_id;

| customer_id | days |
| ----------- | ---- |
| A           | 4    |
| B           | 6    |
| C           | 2    |

---
**Query #3 What was the first item from the menu purchased by each customer?**

    SELECT
    	s.customer_id AS Customer,
    	first_value(m.product_name)OVER(PARTITION BY s.customer_id ORDER BY s.order_date, s.product_id)
    FROM
        dannys_diner.sales AS s
    JOIN
    	dannys_diner.menu AS m
    ON
    	s.product_id = m.product_id
    ORDER BY
    	Customer,
        s.order_date;

| customer | first_value |
| -------- | ----------- |
| A        | sushi       |
| A        | sushi       |
| A        | sushi       |
| A        | sushi       |
| A        | sushi       |
| A        | sushi       |
| B        | curry       |
| B        | curry       |
| B        | curry       |
| B        | curry       |
| B        | curry       |
| B        | curry       |
| C        | ramen       |
| C        | ramen       |
| C        | ramen       |

---
**Query #4 What is the most purchased item on the menu and how many times was it purchased by all customers?**

    SELECT
    	m.product_name AS Product,
        COUNT(s.product_id) AS purchased_times
    FROM
        dannys_diner.sales AS s
    JOIN
    	dannys_diner.menu AS m
    ON
    	s.product_id = m.product_id
    GROUP BY
    	Product
    ORDER BY
    	purchased_times DESC
    LIMIT 1;

| product | purchased_times |
| ------- | --------------- |
| ramen   | 8               |

---
**Query #5 Which item was the most popular for each customer?**

    SELECT
    	s.customer_id AS Customer,
    	m.product_name AS food,
        COUNT(s.product_id) AS times
    FROM
    	dannys_diner.sales AS s
    JOIN
    	dannys_diner.menu AS m
    ON
    	s.product_id = m.product_id
    GROUP BY
    	Customer,
        m.product_name
    ORDER BY
    	Customer ASC, times DESC;

| customer | food  | times |
| -------- | ----- | ----- |
| A        | ramen | 3     |
| A        | curry | 2     |
| A        | sushi | 1     |
| B        | ramen | 2     |
| B        | curry | 2     |
| B        | sushi | 2     |
| C        | ramen | 3     |

---
**Query #6 Which item was purchased first by the customer after they became a member?**

    SELECT
    	s.customer_id AS Customer,
        first_value(m.product_name)OVER(PARTITION BY s.customer_id) AS first_item
    FROM
    	dannys_diner.sales AS s
    JOIN
    	dannys_diner.members AS j
    ON
    	s.customer_id = j.customer_id
    JOIN
    	dannys_diner.menu AS m
    ON
    	s.product_id = m.product_id
    WHERE
    	s.order_date > j.join_date
    ORDER BY
    	Customer ASC;

| customer | first_item |
| -------- | ---------- |
| A        | ramen      |
| A        | ramen      |
| A        | ramen      |
| B        | sushi      |
| B        | sushi      |
| B        | sushi      |

---
**Query #7 Which item was purchased just before the customer became a member?**

    SELECT
    	s.customer_id AS Customer,
        s.order_date,
        m.product_name
    FROM
    	dannys_diner.sales AS s
    JOIN
    	dannys_diner.members AS j
    ON
    	s.customer_id = j.customer_id
    JOIN
    	dannys_diner.menu AS m
    ON
    	s.product_id = m.product_id
    WHERE
    	s.order_date < j.join_date
    ORDER BY
    	Customer ASC;

| customer | order_date               | product_name |
| -------- | ------------------------ | ------------ |
| A        | 2021-01-01T00:00:00.000Z | sushi        |
| A        | 2021-01-01T00:00:00.000Z | curry        |
| B        | 2021-01-04T00:00:00.000Z | sushi        |
| B        | 2021-01-01T00:00:00.000Z | curry        |
| B        | 2021-01-02T00:00:00.000Z | curry        |

---
**Query #8 What is the total items and amount spent for each member before they became a member?**

    SELECT 
    	s.customer_id AS Customer,
     	sum(s.product_id) AS items,
        sum(price) As total_price
    FROM
        dannys_diner.sales AS s
    JOIN
    	dannys_diner.menu AS m
    ON
    	s.product_id = m.product_id
    JOIN
    	dannys_diner.members AS j
    ON 
    	s.customer_id = j.customer_id
    WHERE
    	s.order_date < j.join_date
    GROUP BY
    	Customer
    ORDER BY 
    	Customer ASC;

| customer | items | total_price |
| -------- | ----- | ----------- |
| A        | 3     | 25          |
| B        | 5     | 40          |

---
