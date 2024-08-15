# Case Study - 1 : Danny's Diner

All the data and questions that I've answered in this page can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-1/)

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.

**You can directly skip to code block to directly view the solution. However if you are stuck understanding something I suggest you to read my approach on how I tackled the problem**

## Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/f1971f06-eba6-4976-9294-0ed5b94d5621)

Based on the ER diagram, we can clearly see that there are three tables. The sales table is directly connected to both the members table and the menu table. So, instead of performing the join operation in every query, can we just create a CTE (Common Table Expression) that has all the tables joined? The answer is yes, but the question is how are we going to join them, and what type of join should we use?

Well, the most straightforward approach is to perform an INNER JOIN between sales and members, and then another INNER JOIN between sales and menu. But is that correct?

You see, there might be some customers who have joined the loyalty program but haven’t ordered anything, or vice versa. At the same time, there might be some products on the menu that have never been ordered.

I know that for this case study, the dataset is small, and just by looking at the data, we can rule out these edge cases. But remember, Danny just gave us a sample of the dataset due to privacy issues. Hence, it is always better to write queries for the general case, taking these edge cases into consideration.
## Creation of CTE
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

**1) What is the total amount each customer spent at the restaurant?**\

- For total amount `sum(price)`
- For each customer `GROUP BY customer_id `
- Combining everything we get the final query
#### Final Query 
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
#### Approach 1 : Using Distinct Keyword
- For no_of_days `count(distinct order_date)`. Notice that we use distinct keyword because for each order_date, there could be multiple orders. So we ideally want to count that order_date once only.
- For every customer `group by customer_id`
- Combining everything we get the final query
#### Final Query
```` sql
SELECT 
    customer_id,
    COUNT(DISTINCT order_date)
FROM 
    CTE
GROUP BY 
    customer_id;
````


#### Approach 2 : Without Distinct Keyword
- I wouldn't recommend this method because it is a bit lenghty and this question can be solved simply by using distinct keyword. But just know this logic because these logics can be applied anywhere.
- For each customer and on each specific `order_date`, there might be multiple orders, so first we want to combine all the orders on a specific date to a single row. So here we will use `group by customer_id` as well as `order_date`.
```` sql
SELECT customer_id,order_date FROM cte group by customer_id,order_date;
````
- Now that we got all distinct customer_id and order_id, now all we need to do is `group by customer_id` again by applying `count(order_date) or count(*)` by passing this above query as sub-query.
#### Final Query
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
#### Approach 1 : Using Dense Rank
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
#### Approach 2 : Using Joins
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
**8) What is the total items and amount spent for each member before they became a member?**
```` sql
SELECT customer_id,
       COUNT(product_name) AS total_items,
       SUM(price) AS total_amount_spent
FROM cte
WHERE order_date < join_date
GROUP BY customer_id
ORDER BY customer_id;
````
#### Output Table

| customer_id | total_items | total_amount_spent |
| ----------- | ----------- | ------------------ |
| A           | 2           | 25                 |
| B           | 3           | 40                 |

---
**9) If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

```` sql
SELECT customer_id,
       SUM(
           CASE 
               WHEN product_name = 'sushi' THEN price * 20
               ELSE price * 10
           END
       ) AS points
FROM cte
GROUP BY customer_id
ORDER BY customer_id;
````
#### Output Table

| customer_id | points |
| ----------- | ------ |
| A           | 860    |
| B           | 940    |
| C           | 360    |

---

**10) In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
```` sql
SELECT customer_id,
       SUM(
           CASE 
               WHEN (
                   (order_date - join_date BETWEEN 0 AND 6)
                   OR (product_name = 'sushi')
               ) THEN 20 * price
               ELSE 10 * price
           END
       ) AS total_points
FROM cte
WHERE order_date < '2021-02-01'
  AND customer_id != 'C'
GROUP BY customer_id
ORDER BY customer_id;
````
#### Output Table

| customer_id | total_points |
| ----------- | ------------ |
| A           | 1370         |
| B           | 820          |

---

#### Please feel free to let me know if I have made any mistake or if you know a better approach to solve any question. If this helped you in anyway to improve your skills, just drop a message. It might not mean much to you, but it absolutely makes my day when I know that I’ve helped someone gain some knowledge.
#### Anyways Happy Fiddling with the Data. See You in the next case study.



