# Case Study - 7 : Balanced Tree Clothing Co.

All the data and questions that I've answered in this markdown can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-7/).

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/dkhULDEjGib3K58MvDjYJr/8) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.

**You can directly skip to code block to directly view the solution. However if you are stuck understanding something I suggest you to read my approach on how I tackled the problem**\
**Also please run the code blocks youself on dbfiddle as you can look at the output and follow through my explanations when you are confused**

## Index

- [High Level Sales Analysis](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#high-level-sales-analysis) \
[Creation of CTE](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#creation-of-cte)
[1](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query)
[2](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-1)
[3](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-2)
- [Transaction Analysis](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#transaction-analysis) \
[1](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-3)
[2](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-4)
[3](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-5)
[4](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-6)
[5](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-7)
[6](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-8)
- [Product Analysis](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#product-analysis) \
[1](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-9)
[2](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-10)
[3](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-11)
[4](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-12)
[5](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-13)
[6](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-14)
[7](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-15)
[8](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-16)
[9](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-17)
[10](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#final-query-18)
- [Reporting Challenge](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#reporting-challenge)
- [Bonus Challenge](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%207%20-%20%20Balanced%20Tree%20Clothing%20Co.#bonus-challenge)

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
```` sql
SELECT product_name as top_3_products
FROM CTE
GROUP BY 1
ORDER BY sum((1-0.01*discount)* qty * price) DESC
LIMIT 3;
````
#### Output Table

| top_3_products               |
| ---------------------------- |
| Blue Polo Shirt - Mens       |
| Grey Fashion Jacket - Womens |
| White Tee Shirt - Mens       |

---

**2) What is the total quantity, revenue and discount for each segment?**

#### Final Query
```` sql
SELECT 
    segment_name,
    SUM(qty) AS total_quantity,
    SUM((1 - 0.01 * discount) * qty * price) AS total_revenue,
    SUM(0.01 * discount * qty * price) AS total_discount
FROM 
    CTE
GROUP BY 
    segment_name;
````
#### Output Table

| segment_name | total_quantity | total_revenue | total_discount |
| ------------ | -------------- | ------------- | -------------- |
| Shirt        | 11265          | 356548.73     | 49594.27       |
| Jeans        | 11349          | 183006.03     | 25343.97       |
| Jacket       | 11385          | 322705.54     | 44277.46       |
| Socks        | 11217          | 270963.56     | 37013.44       |

---

**3) What is the top selling product for each segment?**

#### Final Query
```` sql
SELECT 
    segment_name,
    product_name AS top_product
FROM (
    SELECT 
        segment_name,
        product_name,
        DENSE_RANK() OVER (
            PARTITION BY segment_name
            ORDER BY SUM(qty) DESC
        ) AS dn
    FROM 
        CTE
    GROUP BY 
        segment_name,
        product_name
) temp
WHERE 
    dn = 1;
````
#### Output Table

| segment_name | top_product                   |
| ------------ | ----------------------------- |
| Jacket       | Grey Fashion Jacket - Womens  |
| Jeans        | Navy Oversized Jeans - Womens |
| Shirt        | Blue Polo Shirt - Mens        |
| Socks        | Navy Solid Socks - Mens       |

---

**4) What is the total quantity, revenue and discount for each category?**

#### Final Query
```` sql
SELECT 
    category_name,
    SUM(qty) AS total_quantity,
    SUM((1 - 0.01 * discount) * qty * price) AS total_revenue,
    SUM(0.01 * discount * qty * price) AS total_discount
FROM 
    CTE
GROUP BY 
    category_name;
````
#### Output Table

| category_name | total_quantity | total_revenue | total_discount |
| ------------- | -------------- | ------------- | -------------- |
| Mens          | 22482          | 627512.29     | 86607.71       |
| Womens        | 22734          | 505711.57     | 69621.43       |

---

**5) What is the top selling product for each category?**

#### Final Query
```` sql
SELECT 
    category_name,
    product_name AS top_product
FROM (
    SELECT 
        category_name,
        product_name,
        DENSE_RANK() OVER (
            PARTITION BY category_name
            ORDER BY SUM(qty) DESC
        ) AS dn
    FROM 
        CTE
    GROUP BY 
        category_name,
        product_name
) temp
WHERE 
    dn = 1;
````
#### Output Table

| category_name | top_product                  |
| ------------- | ---------------------------- |
| Mens          | Blue Polo Shirt - Mens       |
| Womens        | Grey Fashion Jacket - Womens |

---

**6) What is the percentage split of revenue by product for each segment?**

#### Final Query
```` sql
SELECT 
    segment_name,
    product_name,
    ROUND(
        100.0 * revenue / SUM(revenue) OVER (PARTITION BY segment_name),
        2
    ) AS percentage_split_of_revenue
FROM (
    SELECT 
        segment_name,
        product_name,
        SUM((1 - 0.01 * discount) * qty * price) AS revenue
    FROM 
        CTE
    GROUP BY 
        segment_name,
        product_name
) temp;
````
#### Output Table

| segment_name | product_name                     | percentage_split_of_revenue |
| ------------ | -------------------------------- | --------------------------- |
| Jacket       | Indigo Rain Jacket - Womens      | 19.44                       |
| Jacket       | Khaki Suit Jacket - Womens       | 23.57                       |
| Jacket       | Grey Fashion Jacket - Womens     | 56.99                       |
| Jeans        | Navy Oversized Jeans - Womens    | 24.04                       |
| Jeans        | Black Straight Jeans - Womens    | 58.14                       |
| Jeans        | Cream Relaxed Jeans - Womens     | 17.82                       |
| Shirt        | White Tee Shirt - Mens           | 37.48                       |
| Shirt        | Blue Polo Shirt - Mens           | 53.53                       |
| Shirt        | Teal Button Up Shirt - Mens      | 8.99                        |
| Socks        | Navy Solid Socks - Mens          | 44.24                       |
| Socks        | White Striped Socks - Mens       | 20.20                       |
| Socks        | Pink Fluro Polkadot Socks - Mens | 35.57                       |

---

**7) What is the percentage split of revenue by segment for each category?**

#### Final Query
```` sql
SELECT 
    category_name,
    segment_name,
    ROUND(
        100.0 * revenue / SUM(revenue) OVER (PARTITION BY category_name),
        2
    ) AS percentage_split_of_revenue
FROM (
    SELECT 
        category_name,
        segment_name,
        SUM((1 - 0.01 * discount) * qty * price) AS revenue
    FROM 
        CTE
    GROUP BY 
        1,
        2
) temp;
````
#### Output Table

| category_name | segment_name | percentage_split_of_revenue |
| ------------- | ------------ | --------------------------- |
| Mens          | Socks        | 43.18                       |
| Mens          | Shirt        | 56.82                       |
| Womens        | Jeans        | 36.19                       |
| Womens        | Jacket       | 63.81                       |

---

**8) What is the percentage split of total revenue by category?**

#### Final Query
```` sql
SELECT 
    category_name,
    ROUND(
        100.0 * revenue / SUM(revenue) OVER (),
        2
    ) AS percentage_split_of_revenue
FROM (
    SELECT 
        category_name,
        SUM((1 - 0.01 * discount) * qty * price) AS revenue
    FROM 
        CTE
    GROUP BY 
        1
) temp;
````
#### Output Table

| category_name | percentage_split_of_revenue |
| ------------- | --------------------------- |
| Mens          | 55.37                       |
| Womens        | 44.63                       |

---

**9) What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)**

#### Final Query
```` sql
SELECT 
    product_name,
    ROUND(
        100.0 * COUNT(*) / (SELECT COUNT(DISTINCT txn_id) FROM CTE), 
        2
    ) AS penetration_perc
FROM 
    CTE
GROUP BY 
    product_name;
````
#### Output Table

| product_name                     | penetration_perc |
| -------------------------------- | ---------------- |
| White Tee Shirt - Mens           | 50.72            |
| Navy Solid Socks - Mens          | 51.24            |
| Grey Fashion Jacket - Womens     | 51.00            |
| Navy Oversized Jeans - Womens    | 50.96            |
| Pink Fluro Polkadot Socks - Mens | 50.32            |
| Khaki Suit Jacket - Womens       | 49.88            |
| Black Straight Jeans - Womens    | 49.84            |
| White Striped Socks - Mens       | 49.72            |
| Blue Polo Shirt - Mens           | 50.72            |
| Indigo Rain Jacket - Womens      | 50.00            |
| Cream Relaxed Jeans - Womens     | 49.72            |
| Teal Button Up Shirt - Mens      | 49.68            |

---


**10) What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?**

#### Final Query
```` sql
SELECT a.product_name,b.product_name,c.product_name FROM CTE a
JOIN CTE b on a.txn_id = b.txn_id and a.prod_id < b.prod_id 
JOIN CTE c on a.txn_id = c.txn_id and b.prod_id < c.prod_id 
GROUP BY 1,2,3 
ORDER BY count(*) DESC
LIMIT 1;
````
#### Output Table
 

| product_name           | product_name                 | product_name                |
| ---------------------- | ---------------------------- | --------------------------- |
| White Tee Shirt - Mens | Grey Fashion Jacket - Womens | Teal Button Up Shirt - Mens |

---

 ## Reporting Challenge

**Write a single SQL script that combines all of the previous questions into a scheduled report that the Balanced Tree team can run at the beginning of each month to calculate the previous month’s values.**

<br>

**He first wants you to generate the data for January only - but then he also wants you to demonstrate that you can easily run the samne analysis for February without many changes (if at all).**

<br>

**Feel free to split up your final outputs into as many tables as you need**

- Even though danny did mention split final output into as many tables as you need, try to stick to the least amount of tables at which you can cover all questions.
- Again this is not something very complicated, I'm just going to show you 1 output table. You can create table for other questions if you'd like.

#### Final Query
```` sql
SELECT 
    category_name,
    segment_name,
    product_name,
    SUM(qty) AS total_quantity,
    SUM(qty * price * (1 - 0.01 * discount)) AS total_revenue,
    SUM(qty * price * 0.01 * discount) AS total_discount,
    ROUND(
        100.0 * SUM(qty * price * (1 - 0.01 * discount)) / 
        SUM(SUM(qty * price * (1 - 0.01 * discount))) OVER (), 
        2
    ) AS revenue_perc,
    ROUND(
        100.0 * SUM(CASE WHEN member = 't' THEN 1 ELSE 0 END) / 
        COUNT(txn_id), 
        2
    ) AS members_perc_txns,
    ROUND(
        100.0 * SUM(CASE WHEN member = 'f' THEN 1 ELSE 0 END) / 
        COUNT(txn_id), 
        2
    ) AS nonmembers_perc_txns,
    ROUND(
        AVG(CASE WHEN member = 't' THEN qty * price * (1 - 0.01 * discount) END), 
        2
    ) AS avg_members_revenue,
    ROUND(
        AVG(CASE WHEN member = 'f' THEN qty * price * (1 - 0.01 * discount) END), 
        2
    ) AS avg_nonmembers_revenue
FROM 
    cte
WHERE 
    EXTRACT(MONTH FROM start_txn_time) = 2
GROUP BY 
    category_name, 
    segment_name, 
    product_name
ORDER BY 
    category_name, 
    segment_name, 
    total_revenue;
````

#### Output Table

| category_name | segment_name | product_name                     | total_quantity | total_revenue | total_discount | revenue_perc | members_perc_txns | nonmembers_perc_txns | avg_members_revenue | avg_nonmembers_revenue |
| ------------- | ------------ | -------------------------------- | -------------- | ------------- | -------------- | ------------ | ----------------- | -------------------- | ------------------- | ---------------------- |
| Mens          | Shirt        | Teal Button Up Shirt - Mens      | 1205           | 10589.40      | 1460.60        | 2.86         | 60.57             | 39.43                | 25.22               | 25.06                  |
| Mens          | Shirt        | White Tee Shirt - Mens           | 1198           | 42074.00      | 5846.00        | 11.37        | 60.76             | 39.24                | 106.91              | 105.91                 |
| Mens          | Shirt        | Blue Polo Shirt - Mens           | 1281           | 64048.62      | 8968.38        | 17.32        | 62.77             | 37.23                | 153.46              | 151.84                 |
| Mens          | Socks        | White Striped Socks - Mens       | 1252           | 18763.75      | 2520.25        | 5.07         | 62.68             | 37.32                | 44.42               | 45.68                  |
| Mens          | Socks        | Pink Fluro Polkadot Socks - Mens | 1246           | 31833.30      | 4300.70        | 8.61         | 62.75             | 37.25                | 80.72               | 73.47                  |
| Mens          | Socks        | Navy Solid Socks - Mens          | 1190           | 37583.64      | 5256.36        | 10.16        | 62.08             | 37.92                | 90.37               | 91.46                  |
| Womens        | Jacket       | Indigo Rain Jacket - Womens      | 1245           | 20724.44      | 2930.56        | 5.60         | 58.31             | 41.69                | 48.95               | 51.32                  |
| Womens        | Jacket       | Khaki Suit Jacket - Womens       | 1296           | 26252.66      | 3555.34        | 7.10         | 61.31             | 38.69                | 58.27               | 61.18                  |
| Womens        | Jacket       | Grey Fashion Jacket - Womens     | 1254           | 59200.74      | 8515.26        | 16.00        | 63.52             | 36.48                | 144.53              | 151.03                 |
| Womens        | Jeans        | Cream Relaxed Jeans - Womens     | 1205           | 10595.50      | 1454.50        | 2.86         | 58.93             | 41.07                | 26.28               | 28.10                  |
| Womens        | Jeans        | Navy Oversized Jeans - Womens    | 1224           | 13983.06      | 1928.94        | 3.78         | 62.34             | 37.66                | 34.77               | 35.04                  |
| Womens        | Jeans        | Black Straight Jeans - Womens    | 1224           | 34243.52      | 4924.48        | 9.26         | 62.35             | 37.65                | 82.92               | 85.06                  |



- If we want the data for any other month, in the where condition, you will just change the month number, that's it.

---

 ## Bonus Challenge

**Use a single SQL query to transform the product_hierarchy and product_prices datasets to the product_details table.**

<br>

**Hint: you may want to consider using a recursive CTE to solve this problem!**

### Without Using RECURSIVE CTE

#### Final Query
```` sql
WITH cte AS (
    SELECT 
        p.product_id,
        p.price,
        h.parent_id,
        h.level_text,
        p.id AS style_id,
        h.level_text AS style_name
    FROM 
        balanced_tree.product_prices p
    JOIN 
        balanced_tree.product_hierarchy h
    ON 
        p.id = h.id
), 
cte2 AS (
    SELECT 
        cte.product_id,
        cte.price,
        CONCAT(cte.level_text, ' ', h.level_text) AS product_name,
        h.parent_id,
        h.id AS segment_id,
        cte.style_id,
        h.level_text AS segment_name,
        cte.style_name
    FROM 
        cte
    JOIN 
        balanced_tree.product_hierarchy h
    ON 
        cte.parent_id = h.id
), 
product_details AS (
    SELECT 
        cte2.product_id,
        cte2.price,
        CONCAT(cte2.product_name, ' - ', h.level_text) AS product_name,
        h.id AS category_id,
        cte2.segment_id,
        cte2.style_id,
        h.level_text AS category_name,
        cte2.segment_name,
        cte2.style_name
    FROM 
        cte2
    JOIN 
        balanced_tree.product_hierarchy h
    ON 
        cte2.parent_id = h.id
)

SELECT 
    * 
FROM 
    product_details;
````
#### Output Table

| product_id | price | product_name                     | category_id | segment_id | style_id | category_name | segment_name | style_name          |
| ---------- | ----- | -------------------------------- | ----------- | ---------- | -------- | ------------- | ------------ | ------------------- |
| c4a632     | 13    | Navy Oversized Jeans - Womens    | 1           | 3          | 7        | Womens        | Jeans        | Navy Oversized      |
| e83aa3     | 32    | Black Straight Jeans - Womens    | 1           | 3          | 8        | Womens        | Jeans        | Black Straight      |
| e31d39     | 10    | Cream Relaxed Jeans - Womens     | 1           | 3          | 9        | Womens        | Jeans        | Cream Relaxed       |
| d5e9a6     | 23    | Khaki Suit Jacket - Womens       | 1           | 4          | 10       | Womens        | Jacket       | Khaki Suit          |
| 72f5d4     | 19    | Indigo Rain Jacket - Womens      | 1           | 4          | 11       | Womens        | Jacket       | Indigo Rain         |
| 9ec847     | 54    | Grey Fashion Jacket - Womens     | 1           | 4          | 12       | Womens        | Jacket       | Grey Fashion        |
| 5d267b     | 40    | White Tee Shirt - Mens           | 2           | 5          | 13       | Mens          | Shirt        | White Tee           |
| c8d436     | 10    | Teal Button Up Shirt - Mens      | 2           | 5          | 14       | Mens          | Shirt        | Teal Button Up      |
| 2a2353     | 57    | Blue Polo Shirt - Mens           | 2           | 5          | 15       | Mens          | Shirt        | Blue Polo           |
| f084eb     | 36    | Navy Solid Socks - Mens          | 2           | 6          | 16       | Mens          | Socks        | Navy Solid          |
| b9a74d     | 17    | White Striped Socks - Mens       | 2           | 6          | 17       | Mens          | Socks        | White Striped       |
| 2feb6b     | 29    | Pink Fluro Polkadot Socks - Mens | 2           | 6          | 18       | Mens          | Socks        | Pink Fluro Polkadot |


### With using RECURSIVE CTE

- To be honest I solved this question without using RECURSIVE CTE at first and later I got to know that there is something known as RECURSIVE CTE that we can use to solve this problem.
- This is my first time implementing recursive CTE, so I'm not really sure how can I even optimise this. Because with using Recursive CTE and without using recursive CTE, I'm able to generate the same output, but I feel that both processes I did are kind of lengthy.
- I feel like with help of RECURSIVE CTE the problem could be solved much easily but I'm not able to find the solution
- So If you please know how to properly use RECURSIVE CTE to our advantage, please let me know, Thank You.

```` sql
WITH RECURSIVE CTE AS (
    SELECT temp.*, 1 AS level 
    FROM balanced_tree.product_hierarchy temp
    WHERE id IN (1, 2)
    UNION 
    SELECT h.id, h.parent_id, h.level_text, h.level_name, level + 1 AS level
    FROM CTE
    JOIN balanced_tree.product_hierarchy h
    ON CTE.id = h.parent_id
),
CTE2 AS (
    SELECT c.id AS category_id, segment_id, style_id, c.level_text AS category_name, segment_name, style_name
    FROM (
        SELECT a.id AS style_id, a.parent_id AS segment_id, a.level_text AS style_name, b.level_text AS segment_name
        FROM CTE a
        JOIN CTE b
        ON a.parent_id = b.id AND a.level = 3 AND b.level = 2
    ) temp
    JOIN CTE c
    ON temp.parent_id = c.id
)

SELECT product_id, price, CONCAT(style_name, ' ', segment_name, ' - ', category_name) AS product_name,
       category_id, segment_id, style_id, category_name, segment_name, style_name
FROM balanced_tree.product_prices p
JOIN CTE2
ON p.id = CTE2.style_id
ORDER BY p.id;
````
- Output would be same table.

---

### Please feel free to let me know if I have made any mistake or if you know a better approach to solve any question. If this helped you in anyway to improve your skills, please reach out to me at rishik130406@gmail.com . It might not mean much to you, but it absolutely makes my day when I know that I’ve helped someone gain some knowledge.

### Anyways Happy Fiddling with the Data. See you in the next case study.
