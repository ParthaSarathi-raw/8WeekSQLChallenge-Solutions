# Case Study - 7 : Balanced Tree Clothing Co.

All the data and questions that I've answered in this markdown can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-7/).

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/dkhULDEjGib3K58MvDjYJr/8) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.

**You can directly skip to code block to directly view the solution. However if you are stuck understanding something I suggest you to read my approach on how I tackled the problem**\
**Also please run the code blocks youself on dbfiddle as you can look at the output and follow through my explanations when you are confused**

## High Level Sales Analysis

### Creation of CTE

- Since there are only 2 main tables to answer sections 1,2 and 3 i.e excluding the challenges, lets combine them and use the CTE to answer all the questions.

```` sql

WITH CTE AS (
    SELECT s.*, p.product_name, p.category_id, p.segment_id, p.style_id, p.category_name, p.segment_name, p.style_name
    FROM balanced_tree.sales s
    JOIN balanced_tree.product_details p
    ON s.prod_id = p.product_id
)
SELECT * 
FROM CTE;
````

#### Output CTE

- Only showing first 10 rows.

| prod_id | qty | price | discount | member | txn_id | start_txn_time           | product_name                     | category_id | segment_id | style_id | category_name | segment_name | style_name          |
| ------- | --- | ----- | -------- | ------ | ------ | ------------------------ | -------------------------------- | ----------- | ---------- | -------- | ------------- | ------------ | ------------------- |
| c4a632  | 4   | 13    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z | Navy Oversized Jeans - Womens    | 1           | 3          | 7        | Womens        | Jeans        | Navy Oversized      |
| 5d267b  | 4   | 40    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z | White Tee Shirt - Mens           | 2           | 5          | 13       | Mens          | Shirt        | White Tee           |
| b9a74d  | 4   | 17    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z | White Striped Socks - Mens       | 2           | 6          | 17       | Mens          | Socks        | White Striped       |
| 2feb6b  | 2   | 29    | 17       | true   | 54f307 | 2021-02-13T01:59:43.296Z | Pink Fluro Polkadot Socks - Mens | 2           | 6          | 18       | Mens          | Socks        | Pink Fluro Polkadot |
| c4a632  | 5   | 13    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z | Navy Oversized Jeans - Womens    | 1           | 3          | 7        | Womens        | Jeans        | Navy Oversized      |
| e31d39  | 2   | 10    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z | Cream Relaxed Jeans - Womens     | 1           | 3          | 9        | Womens        | Jeans        | Cream Relaxed       |
| 72f5d4  | 3   | 19    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z | Indigo Rain Jacket - Womens      | 1           | 4          | 11       | Womens        | Jacket       | Indigo Rain         |
| 2a2353  | 3   | 57    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z | Blue Polo Shirt - Mens           | 2           | 5          | 15       | Mens          | Shirt        | Blue Polo           |
| f084eb  | 3   | 36    | 21       | true   | 26cc98 | 2021-01-19T01:39:00.345Z | Navy Solid Socks - Mens          | 2           | 6          | 16       | Mens          | Socks        | Navy Solid          |
| c4a632  | 1   | 13    | 21       | false  | ef648d | 2021-01-27T02:18:17.164Z | Navy Oversized Jeans - Womens    | 1           | 3          | 7        | Womens        | Jeans        | Navy Oversized      |

--- 

**1) What was the total quantity sold for all products?** 

- Total Quantity : `sum(qty)`

#### Final Query

```` sql
SELECT sum(qty) as total_quantity_sold FROM CTE;
````

#### Output Table

| total_quantity_sold |
| ------------------- |
| 45216               |

---

**2) What is the total generated revenue for all products before discounts?**

- Revenue before discount : `sum(price*qty)`

#### Final Query
```` sql
SELECT sum(qty*price) as total_revenue FROM CTE;
````
#### Output Table

| total_revenue |
| ------------- |
| 1289453       |

---
 
**3) What was the total discount amount for all products?**


#### Final Query
```` sql
SELECT round(sum(0.01*qty*price*discount),2) as total_discount FROM CTE;
````
#### Output Table

| total_discount |
| -------------- |
| 156229.14      |

---

 ## Transaction Analysis

**1) How many unique transactions were there?**

#### Final Query
```` sql
SELECT count(distinct txn_id) as unique_transactions FROM CTE;
````
#### Output Table

| unique_transactions |
| ------------------- |
| 2500                |

---

**2) What is the average unique products purchased in each transaction?**

- First we find the unique products per txn and then apply average across it.

#### Final Query
```` sql
SELECT AVG(count) as avg_unique_products_purchased_per_txn
FROM (
    SELECT COUNT(DISTINCT prod_id) 
    FROM cte 
    GROUP BY txn_id
) t;
````
#### Output Table

| avg_unique_products_purchased_per_txn |
| ------------------------------------- |
| 6.0380000000000000                    |

---

**3) What are the 25th, 50th and 75th percentile values for the revenue per transaction?**

- First we find the revenue for each transaction and then find percentile.
- 
#### Final Query
```` sql
SELECT 
round(PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY revenue)::decimal,2) as percentile_25,
round(PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY revenue)::decimal,2) as percentile_50,
round(PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY revenue)::decimal,2) as percentile_75
FROM (
SELECT sum((1-0.01*discount)*qty*price) as revenue FROM CTE GROUP BY txn_id) TEMP
;
````
#### Output Table

| percentile_25 | percentile_50 | percentile_75 |
| ------------- | ------------- | ------------- |
| 326.41        | 441.23        | 572.76        |

---

**4) What is the average discount value per transaction?**

- First we find total discount for each transaction and then find average.

#### Final Query
```` sql
SELECT ROUND(AVG(discount), 2) AS avg_discount
FROM (
    SELECT txn_id, SUM(qty * price * 0.01 * discount) AS discount
    FROM cte
    GROUP BY txn_id
) temp;
````

#### Output Table

| avg_discount |
| ------------ |
| 62.49        |

---

**5) What is the percentage split of all transactions for members vs non-members?**

#### Final Query
```` sql
SELECT 
    member,
    COUNT(DISTINCT txn_id) AS no_of_txn,
    100.0 * COUNT(DISTINCT txn_id) / SUM(COUNT(DISTINCT txn_id)) OVER() AS percentage
FROM 
    CTE
GROUP BY 
    member;
````
#### Output Table
| member | no_of_txn | percentage          |
| ------ | --------- | ------------------- |
| false  | 995       | 39.8000000000000000 |
| true   | 1505      | 60.2000000000000000 |

---

**6) What is the average revenue for member transactions and non-member transactions?**

#### Final Query
```` sql
SELECT 
    member,
    AVG(revenue) AS avg_revenue
FROM (
    SELECT 
        member,
        txn_id,
        SUM((1 - 0.01 * discount) * qty * price) AS revenue
    FROM 
        CTE
    GROUP BY 
        member, txn_id
) TEMP
GROUP BY member;
````

#### Output Table

| member | avg_revenue          |
| ------ | -------------------- |
| false  | 452.0077688442211055 |
| true   | 454.1369634551495017 |

---

 ## Product Analysis

**1) What are the top 3 products by total revenue before discount?**

#### Final Query

#### Output Table
 
**2) What is the total quantity, revenue and discount for each segment?**

#### Final Query

#### Output Table
 
**3) What is the top selling product for each segment?**

#### Final Query

#### Output Table
 
**4) What is the total quantity, revenue and discount for each category?**

#### Final Query

#### Output Table
 
**5) What is the top selling product for each category?**

#### Final Query

#### Output Table
 
**6) What is the percentage split of revenue by product for each segment?**

#### Final Query

#### Output Table
 
**7) What is the percentage split of revenue by segment for each category?**

#### Final Query

#### Output Table
 
**8) What is the percentage split of total revenue by category?**

#### Final Query

#### Output Table
 
**9) What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)**

#### Final Query

#### Output Table
 
**10) What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?**

#### Final Query

#### Output Table
 

 ## Reporting Challenge


 ## Bonus Challenge
