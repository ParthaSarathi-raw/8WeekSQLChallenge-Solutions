# Case Study - 1 : Danny's Diner

All the data and questions that I've answered in this page can be found at  [here](https://8weeksqlchallenge.com/case-study-1/)

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle.

## Creating of CTE
We will be using this CTE table as the base table to answer all the questions.

```` sql
WITH CTE as (
SELECT s.*,m.join_date,men.product_name,men.price
FROM dannys_diner.sales s 
FULL OUTER JOIN dannys_diner.members m ON s.customer_id = m.customer_id
FULL OUTER JOIN dannys_diner.menu men ON s.product_id = men.product_id
ORDER BY s.customer_id,order_date)
SELECT * FROM cte;
````
#### Base CTE Table

| customer_id | order_date               | product_id | join_date                | product_name | price |
| ----------- | ------------------------ | ---------- | ------------------------ | ------------ | ----- |
| A           | 2021-01-01T00:00:00.000Z | 2          | 2021-01-07T00:00:00.000Z | curry        | 15    |
| A           | 2021-01-01T00:00:00.000Z | 1          | 2021-01-07T00:00:00.000Z | sushi        | 10    |
| A           | 2021-01-07T00:00:00.000Z | 2          | 2021-01-07T00:00:00.000Z | curry        | 15    |
| A           | 2021-01-10T00:00:00.000Z | 3          | 2021-01-07T00:00:00.000Z | ramen        | 12    |
| A           | 2021-01-11T00:00:00.000Z | 3          | 2021-01-07T00:00:00.000Z | ramen        | 12    |
| A           | 2021-01-11T00:00:00.000Z | 3          | 2021-01-07T00:00:00.000Z | ramen        | 12    |
| B           | 2021-01-01T00:00:00.000Z | 2          | 2021-01-09T00:00:00.000Z | curry        | 15    |
| B           | 2021-01-02T00:00:00.000Z | 2          | 2021-01-09T00:00:00.000Z | curry        | 15    |
| B           | 2021-01-04T00:00:00.000Z | 1          | 2021-01-09T00:00:00.000Z | sushi        | 10    |
| B           | 2021-01-11T00:00:00.000Z | 1          | 2021-01-09T00:00:00.000Z | sushi        | 10    |
| B           | 2021-01-16T00:00:00.000Z | 3          | 2021-01-09T00:00:00.000Z | ramen        | 12    |
| B           | 2021-02-01T00:00:00.000Z | 3          | 2021-01-09T00:00:00.000Z | ramen        | 12    |
| C           | 2021-01-01T00:00:00.000Z | 3          |                          | ramen        | 12    |
| C           | 2021-01-01T00:00:00.000Z | 3          |                          | ramen        | 12    |
| C           | 2021-01-07T00:00:00.000Z | 3          |                          | ramen        | 12    |

---

## Solutions

**1) What is the total amount each customer spent at the restaurant?**

```` sql
SELECT 
    customer_id,
    SUM(price) AS total_amount
FROM 
    cte
GROUP BY 
    customer_id
ORDER BY 
    customer_id;
````
#### Output Table

    
| customer_id | total_amount |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

---

**2) How many days has each customer visited the restaurant?**
### Approach 1 : Using Distinct Keyword
```` sql
SELECT 
    customer_id,
    COUNT(DISTINCT order_date)
FROM 
    CTE
GROUP BY 
    customer_id;
````


### Approach 2 : Without Distinct Keyword
```` sql
SELECT customer_id,
       COUNT(*)
FROM (
    SELECT customer_id,
           order_date 
    FROM cte 
    GROUP BY customer_id, 
             order_date
) temp
GROUP BY customer_id
ORDER BY customer_id;
````
#### Output Table


| customer_id | count |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

---
**3) What was the first item from the menu purchased by each customer?**
### Approach 1 : Using Dense Rank
```` sql
SELECT DISTINCT customer_id,
                product_name AS first_item_purchased
FROM (
    SELECT customer_id,
           product_name,
           DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS dn
    FROM CTE
) temp
WHERE dn = 1;
````
### Approach 2 : Using Joins
```` sql
SELECT cte.customer_id,
       product_name AS first_item_ordered
FROM cte
JOIN (
    SELECT customer_id,
           MIN(order_date) AS min_date
    FROM CTE
    GROUP BY customer_id
) temp ON cte.customer_id = temp.customer_id
      AND cte.order_date = temp.min_date
GROUP BY 1, 2;
````
#### Output Table


| customer_id | first_item_purchased |
| ----------- | -------------------- |
| A           | curry                |
| A           | sushi                |
| B           | curry                |
| C           | ramen                |

---

**4) What is the most purchased item on the menu and how many times was it purchased by all customers?**
```` sql
SELECT product_name AS most_purchased_product,
       COUNT(*) AS no_of_times_purchased
FROM CTE
GROUP BY product_name
ORDER BY 2 DESC
LIMIT 1;
````
#### Output Table

| most_purchased_product | no_of_times_purchased |
| ---------------------- | --------------------- |
| ramen                  | 8                     |

---
**5) Which item was the most popular for each customer?**
```` sql
SELECT customer_id,
       product_name
FROM (
    SELECT customer_id,
           product_name as most_popular_item,
           DENSE_RANK() OVER (
               PARTITION BY customer_id
               ORDER BY COUNT(*) DESC
           ) AS dn
    FROM CTE
    GROUP BY 1, 2
) temp
WHERE dn = 1
ORDER BY 1;
````
#### Output Table


| customer_id | most_popular_item |
| ----------- | ----------------- |
| A           | ramen             |
| B           | ramen             |
| B           | curry             |
| B           | sushi             |
| C           | ramen             |

---

**6) Which item was purchased first by the customer after they became a member?**
```` sql
SELECT customer_id,
       product_name AS first_product_purchased_after_becoming_member
FROM (
    SELECT customer_id,
           product_name,
           DENSE_RANK() OVER (
               PARTITION BY customer_id
               ORDER BY order_date
           ) AS dn
    FROM cte
    WHERE order_date > join_date
) temp
WHERE dn = 1;
````
#### Output Table


| customer_id | first_product_purchased_after_becoming_member |
| ----------- | --------------------------------------------- |
| A           | ramen                                         |
| B           | sushi                                         |

---

**7) Which item was purchased just before the customer became a member?**
````sql
SELECT customer_id,
       product_name AS product_purchased_just_before_becoming_member
FROM (
    SELECT customer_id,
           product_name,
           DENSE_RANK() OVER (
               PARTITION BY customer_id
               ORDER BY order_date DESC
           ) AS dn
    FROM cte
    WHERE order_date < join_date
) temp
WHERE dn = 1;
````
#### Output Table
| customer_id | product_purchased_just_before_becoming_member |
| ----------- | --------------------------------------------- |
| A           | sushi                                         |
| A           | curry                                         |
| B           | sushi                                         |

---



