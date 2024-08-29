# Case Study - 6 : Clique Bait

All the data and questions that I've answered in this markdown can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-6/).

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.

**You can directly skip to code block to directly view the solution. However if you are stuck understanding something I suggest you to read my approach on how I tackled the problem**\
**Also please run the code blocks youself on dbfiddle as you can look at the output and follow through my explanations when you are confused**

## Index


- [Digital Analysis](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#digital-analysis) \
[1](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query)
[2](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-1)
[3](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-2)
[4](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-3)
[5](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-4)
[6](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-5)
[7](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-6)
[8](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-7)
[9](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-8)
- [Product Funnel Analysis](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#product-funnel-analysis) \
[1](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-11)
[2](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-12)
[3](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-13)
[4](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-14)
[5](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-15)
- [Campaign Analysis](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#campaigns-analysis) \
[1](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-17)
[2](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-18)
[3](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-19)
[4](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%206%20-%20Clique%20Bait#final-query-20)

## Digital Analysis

**1) How many users are there?**

- no. of users : `count(distinct user_id)`

#### Final Query
```` sql
SELECT count(distinct user_id) as no_of_users FROM clique_bait.users ;
````
#### Output Table
 
| no_of_users |
| ----------- |
| 500         |

---

**2) How many cookies does each user have on average?**

- For each user we will first count the no. of unique cookies. And on that we will do avg.

#### Final Query
```` sql
SELECT ROUND(AVG(no_of_cookies), 2) AS avg_no_of_cookies_per_user
FROM (
    SELECT user_id, COUNT(DISTINCT cookie_id) AS no_of_cookies
    FROM clique_bait.users
    GROUP BY user_id
) temp;
````
#### Output Table

| avg_no_of_cookies_per_user |
| -------------------------- |
| 3.56                       |

---

**3) What is the unique number of visits by all users per month?**

- Per Month : `GROUP BY month`
- Unique number of visits : `count(distinct visit_id)`

#### Final Query
```` sql
SELECT 
    EXTRACT(MONTH FROM event_time) AS month,
    COUNT(DISTINCT visit_id) AS no_of_visits
FROM 
    clique_bait.events
GROUP BY 
    1
ORDER BY 
    1;
````
#### Output Table

| month | no_of_visits |
| ----- | ------------ |
| 1     | 876          |
| 2     | 1488         |
| 3     | 916          |
| 4     | 248          |
| 5     | 36           |

---

**4) What is the number of events for each event type?**

- For each event type : `GROUP BY event_type`
- no. of events : `count(*)`

#### Final Query
```` sql
SELECT event_type,count(*) as no_of_events FROM clique_bait.events GROUP BY 1 ORDER By 1;
````
#### Output Table

 | event_type | no_of_events |
| ---------- | ------------ |
| 1          | 20928        |
| 2          | 8451         |
| 3          | 1777         |
| 4          | 876          |
| 5          | 702          |

---
**5) What is the percentage of visits which have a purchase event?**
- This can also be solved using WHERE condition and doing a subquery in the denominator to find unique visit ids as well. 
#### Final Query
```` sql
SELECT 
    ROUND(100.0 * SUM(CASE WHEN no_of_purchases > 0 THEN 1 ELSE 0 END) / COUNT(*), 2)
FROM (
    SELECT 
        visit_id,
        SUM(CASE WHEN event_name = 'Purchase' THEN 1 ELSE 0 END) AS no_of_purchases
    FROM 
        clique_bait.events e
    JOIN 
        clique_bait.event_identifier i 
    ON 
        e.event_type = i.event_type
    GROUP BY 
        visit_id
) temp;
````
#### Output Table

| round |
| ----- |
| 49.86 |

---

**6) What is the percentage of visits which view the checkout page but do not have a purchase event?**

- Very similar to previous question, just need to add an extra column for view checkout page.

#### Final Query
```` sql
SELECT 
    round(100.0*sum(case when no_of_views_for_checkout > 0 and no_of_purchases = 0 then 1 else 0 end)/count(*),2) as perc_of_visits
FROM (
    SELECT 
        visit_id,
        SUM(CASE WHEN event_name = 'Purchase' THEN 1 ELSE 0 END) AS no_of_purchases,
  		SUM(CASE WHEN event_name = 'Page View' and page_id = 12 then 1 else 0 end) as no_of_views_for_checkout
    FROM 
        clique_bait.events e
    JOIN 
        clique_bait.event_identifier i 
    ON 
        e.event_type = i.event_type
    GROUP BY 
        visit_id
) temp;
````
#### Output Table

| perc_of_visits |
| -------------- |
| 9.15           |

---

 
**7) What are the top 3 pages by number of views?**

- We need page name, so join tables on `page_id` and perform `GROUP BY page_name`
- We need to only consider page views : `WHERE event_type = 1`
- No. of times it has been viewed : `count(*)`
- Top 3 pages : `LIMIT 3`

#### Final Query
```` sql
SELECT page_name,count(*) as views 
FROM clique_bait.events e 
JOIN clique_bait.page_hierarchy h 
ON e.page_id = h.page_id 
WHERE event_type = 1
GROUP BY 1
ORDER BY 2 DESC
LIMIT 3;
````
#### Output Table
 
| page_name    | views |
| ------------ | ----- |
| All Products | 3174  |
| Checkout     | 2103  |
| Home Page    | 1782  |

---

**8) What is the number of views and cart adds for each product category?**

- For each product category : `GROUP BY product_category`
- No. of views and cart adds : `SUM(CASE)`

#### Final Query
```` sql
SELECT product_category,
	   sum(case when event_type = 1 then 1 end) as views,
       sum(case when event_type = 2 then 1 end) as added_to_cart
FROM clique_bait.events e 
JOIN clique_bait.page_hierarchy h 
ON e.page_id = h.page_id 
WHERE product_category is not null
GROUP BY 1;
````

#### Output Table

| product_category | views | added_to_cart |
| ---------------- | ----- | ------------- |
| Luxury           | 3032  | 1870          |
| Shellfish        | 6204  | 3792          |
| Fish             | 4633  | 2789          |

---

**9) What are the top 3 products by purchases?**

- We need to have only those visit ids where purchases have happened.
- On those visit ids we will see which pages had most added to carts. These are already purchased.

**Note : This approach only works because in the data, there is only 1 purchase per each visit and that is the event_type which is last for that specific visit_id. So there is no case where, after some purchase something else was added to cart.**

#### Final Query
```` sql
SELECT page_name,count(*) as purchased_times
FROM clique_bait.events e 
JOIN clique_bait.page_hierarchy h 
ON e.page_id = h.page_id 
WHERE visit_id IN (SELECT visit_id FROM clique_bait.events WHERE event_type = 3) 
AND event_type = 2
GROUP BY 1
ORDER BY 2 DESC
LIMIT 3;
````
#### Output Table
 
| page_name | purchased_times |
| --------- | --------------- |
| Lobster   | 754             |
| Oyster    | 726             |
| Crab      | 719             |

---


## Product Funnel Analysis

**Using a single SQL query - create a new output table which has the following details: \
How many times was each product viewed? \
How many times was each product added to cart? \
How many times was each product added to a cart but not purchased (abandoned)? \
How many times was each product purchased?**

- Notice how we are using left join to determine weather a particular product/visit_id has done purchase or not.

#### Final Query

```` sql
WITH product_details AS (
    SELECT 
        page_name,
        SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS views,
        SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS added_to_cart,
        SUM(CASE WHEN event_type = 2 AND temp.visit_id IS NULL THEN 1 ELSE 0 END) AS abandoned,
        SUM(CASE WHEN event_type = 2 AND temp.visit_id IS NOT NULL THEN 1 ELSE 0 END) AS purchased
    FROM 
        clique_bait.events e 
    JOIN 
        clique_bait.page_hierarchy h ON e.page_id = h.page_id 
    LEFT JOIN 
        (SELECT visit_id FROM clique_bait.events WHERE event_type = 3) temp 
        ON e.visit_id = temp.visit_id 
    WHERE 
        e.page_id NOT IN (1, 2, 12, 13) 
    GROUP BY 
        page_name
)
SELECT * FROM product_details;

````

#### Output Table

| page_name      | views | added_to_cart | abandoned | purchased |
| -------------- | ----- | ------------- | --------- | --------- |
| Abalone        | 1525  | 932           | 233       | 699       |
| Oyster         | 1568  | 943           | 217       | 726       |
| Salmon         | 1559  | 938           | 227       | 711       |
| Crab           | 1564  | 949           | 230       | 719       |
| Tuna           | 1515  | 931           | 234       | 697       |
| Lobster        | 1547  | 968           | 214       | 754       |
| Kingfish       | 1559  | 920           | 213       | 707       |
| Russian Caviar | 1563  | 946           | 249       | 697       |
| Black Truffle  | 1469  | 924           | 217       | 707       |

---

**Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.**

#### Final Query
```` sql
WITH product_category_details AS (
    SELECT 
        product_category,
        SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS views,
        SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS added_to_cart,
        SUM(CASE WHEN event_type = 2 AND temp.visit_id IS NULL THEN 1 ELSE 0 END) AS abandoned,
        SUM(CASE WHEN event_type = 2 AND temp.visit_id IS NOT NULL THEN 1 ELSE 0 END) AS purchased
    FROM 
        clique_bait.events e 
    JOIN 
        clique_bait.page_hierarchy h ON e.page_id = h.page_id 
    LEFT JOIN 
        (SELECT visit_id FROM clique_bait.events WHERE event_type = 3) temp 
        ON e.visit_id = temp.visit_id 
    WHERE 
        product_category is not null 
    GROUP BY 
        product_category
)
SELECT * FROM product_category_details;
````
#### Output Table

| product_category | views | added_to_cart | abandoned | purchased |
| ---------------- | ----- | ------------- | --------- | --------- |
| Luxury           | 3032  | 1870          | 466       | 1404      |
| Shellfish        | 6204  | 3792          | 894       | 2898      |
| Fish             | 4633  | 2789          | 674       | 2115      |

---

**Use your 2 new output tables - answer the following questions:**

<br>

**1) Which product had the most views, cart adds and purchases?**

#### Final Query
```` sql
SELECT 
    (SELECT page_name FROM product_details ORDER BY views DESC LIMIT 1) AS most_viewed,
    (SELECT page_name FROM product_details ORDER BY added_to_cart DESC LIMIT 1) AS most_added_to_cart,
    (SELECT page_name FROM product_details ORDER BY purchased DESC LIMIT 1) AS most_purchased;
````

#### Output Table

| most_viewed | most_added_to_cart | most_purchased |
| ----------- | ------------------ | -------------- |
| Oyster      | Lobster            | Lobster        |

---

**2) Which product was most likely to be abandoned?**

#### Final Query
```` sql
SELECT 
    (SELECT page_name FROM product_details ORDER BY abandoned DESC LIMIT 1) AS most_likely_to_be_abandoned;
````

#### Output Table

| most_likely_to_be_abandoned |
| --------------------------- |
| Russian Caviar              |

---

**3) Which product had the highest view to purchase percentage?**

#### Final Query
```` sql
SELECT 
    page_name AS highest_view_to_purchase_percentage 
FROM 
    product_details 
ORDER BY 
    1.0 * purchased / views DESC 
LIMIT 1;
````
#### Output Table


| highest_view_to_purchase_percentage |
| ----------------------------------- |
| Lobster                             |

---

 
**4) What is the average conversion rate from view to cart add?**

#### Final Query
```` sql
SELECT 
    ROUND(AVG(100.0 * added_to_cart / views), 2) AS avg_conversion_rate_from_view_to_cart 
FROM 
    product_details;
````

#### Output Table


| avg_conversion_rate_from_view_to_cart |
| ------------------------------------- |
| 60.95                                 |

---

 
**5) What is the average conversion rate from cart add to purchase?**

#### Final Query
```` sql
SELECT 
    ROUND(AVG(100.0 * purchased / added_to_cart), 2) AS avg_conversion_rate_from_cart_to_purchased 
FROM 
    product_details;
````
#### Output Table

| avg_conversion_rate_from_cart_to_purchased |
| ------------------------------------------ |
| 75.93                                      |

---


## Campaigns Analysis

**Generate a table that has 1 single row for every unique visit_id record and has the following columns:**

•	user_id \
•	visit_id \
•	visit_start_time: the earliest event_time for each visit \
•	page_views: count of page views for each visit \
•	cart_adds: count of product cart add events for each visit \
•	purchase: 1/0 flag if a purchase event exists for each visit \
•	campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date \
•	impression: count of ad impressions for each visit \
•	click: count of ad clicks for each visit \
•	(Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number) 

#### Final Query 

```` sql
WITH campaign_analysis as (SELECT 
    user_id,
    visit_id,
    MIN(event_time) AS visit_start_time,
    SUM(CASE WHEN event_type = 1 THEN 1 ELSE 0 END) AS page_views,
    SUM(CASE WHEN event_type = 2 THEN 1 ELSE 0 END) AS cart_adds,
    SUM(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase,
    campaign_name,
    SUM(CASE WHEN event_type = 4 THEN 1 ELSE 0 END) AS impression,
    SUM(CASE WHEN event_type = 5 THEN 1 ELSE 0 END) AS click,
    STRING_AGG(CASE WHEN event_type = 2 THEN page_name ELSE NULL END, ', ' ORDER BY sequence_number) AS cart_products
FROM 
    clique_bait.users u 
JOIN 
    clique_bait.events e ON u.cookie_id = e.cookie_id
LEFT JOIN 
    clique_bait.campaign_identifier i ON u.start_date BETWEEN i.start_date AND i.end_date 
JOIN 
    clique_bait.page_hierarchy h ON e.page_id = h.page_id
GROUP BY 
    user_id, visit_id, campaign_name
ORDER BY 
	user_id,visit_start_time)
SELECT * FROM campaign_analysis;
````
- Even though the question clearly mentions to join campaign_identifier table when visit_start_time falls between that campaigns start_date and end_date, I joined them on visit_id's start_date because visit_start_time's date is same as visit_id's start_date.

#### Output Table

- Only showing data for user_id 1 and 2. Original table has data for 500 users.

| user_id | visit_id | visit_start_time         | page_views | cart_adds | purchase | campaign_name                     | impression | click | cart_products                                                                         |
| ------- | -------- | ------------------------ | ---------- | --------- | -------- | --------------------------------- | ---------- | ----- | ------------------------------------------------------------------------------------- |
| 1       | 0fc437   | 2020-02-04T17:49:49.602Z | 10         | 6         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Tuna, Russian Caviar, Black Truffle, Abalone, Crab, Oyster                            |
| 1       | ccf365   | 2020-02-04T19:16:09.182Z | 7          | 3         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster, Crab, Oyster                                                                 |
| 1       | 0826dc   | 2020-02-26T05:58:37.918Z | 1          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 1       | 02a5d5   | 2020-02-26T16:57:26.260Z | 4          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 1       | f7c798   | 2020-03-15T02:23:26.312Z | 9          | 3         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Russian Caviar, Crab, Oyster                                                          |
| 1       | 30b94d   | 2020-03-15T13:12:54.023Z | 9          | 7         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Abalone, Lobster, Crab                        |
| 1       | 41355d   | 2020-03-25T00:11:17.860Z | 6          | 1         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Lobster                                                                               |
| 1       | eaffde   | 2020-03-25T20:06:32.342Z | 10         | 8         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster           |
| 2       | 3b5871   | 2020-01-18T10:16:32.158Z | 9          | 6         | 1        | 25% Off - Living The Lux Life     | 1          | 1     | Salmon, Kingfish, Russian Caviar, Black Truffle, Lobster, Oyster                      |
| 2       | c5c0ee   | 2020-01-18T10:35:22.765Z | 1          | 0         | 0        | 25% Off - Living The Lux Life     | 0          | 0     |                                                                                       |
| 2       | e26a84   | 2020-01-18T16:06:40.907Z | 6          | 2         | 1        | 25% Off - Living The Lux Life     | 0          | 0     | Salmon, Oyster                                                                        |
| 2       | d58cbd   | 2020-01-18T23:40:54.761Z | 8          | 4         | 0        | 25% Off - Living The Lux Life     | 0          | 0     | Kingfish, Tuna, Abalone, Crab                                                         |
| 2       | 910d9a   | 2020-02-01T10:40:46.875Z | 8          | 1         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Abalone                                                                               |
| 2       | 1f1198   | 2020-02-01T21:51:55.078Z | 1          | 0         | 0        | Half Off - Treat Your Shellf(ish) | 0          | 0     |                                                                                       |
| 2       | 49d73d   | 2020-02-16T06:21:27.138Z | 11         | 9         | 1        | Half Off - Treat Your Shellf(ish) | 1          | 1     | Salmon, Kingfish, Tuna, Russian Caviar, Black Truffle, Abalone, Lobster, Crab, Oyster |
| 2       | 0635fb   | 2020-02-16T06:42:42.735Z | 9          | 4         | 1        | Half Off - Treat Your Shellf(ish) | 0          | 0     | Salmon, Kingfish, Abalone, Crab                                                       | 

--- 

**Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event**

- First step is to find the no. of campaigns for each user that had impression on the user.

```` sql
,impression_count as (
SELECT 
    user_id,
    SUM(CASE WHEN campaign_name IS NOT NULL THEN 1 ELSE 0 END) AS campaigns_impressioned
FROM (
    SELECT 
        user_id, 
        campaign_name 
    FROM 
        campaign_analysis 
    WHERE 
        impression = 1 
    GROUP BY 
        user_id, campaign_name 
    ORDER BY 
        1
) temp
GROUP BY 
    user_id 
ORDER BY 
    user_id)
SELECT * FROM impression_count;
````
- Showing first 10 rows only. Original data has 500 users data.

| user_id | campaigns_impressioned |
| ------- | ---------------------- |
| 1       | 1                      |
| 2       | 2                      |
| 3       | 0                      |
| 4       | 1                      |
| 5       | 1                      |
| 6       | 1                      |
| 7       | 1                      |
| 8       | 1                      |
| 9       | 1                      |
| 10      | 2                      |

- Now all we gotta do is join the above table with `campaign_analysis` and see how each metric is doing for users with 3 impressions and users with no impressions in any campaign.

#### Final Query

```` sql
SELECT 
    campaigns_impressioned,
    ROUND(AVG(page_views), 2) AS avg_page_views,
    ROUND(AVG(cart_adds), 2) AS avg_cart_adds,
    ROUND(100.0 * AVG(purchase), 2) AS purchase_percentage
FROM 
    impression_count c 
JOIN 
    campaign_analysis a ON c.user_id = a.user_id
WHERE 
    campaigns_impressioned IN (0, 3)
GROUP BY 
    1 
ORDER BY 
    1;
````

#### Output Table

| campaigns_impressioned | avg_page_views | avg_cart_adds | purchase_percentage |
| ---------------------- | -------------- | ------------- | ------------------- |
| 0                      | 5.45           | 2.02          | 45.08               |
| 3                      | 7.48           | 3.23          | 62.50               |

- I'm here to do the sql part, I will leave the insights up to you.

---




- Does clicking on an impression lead to higher purchase rates?


#### Final Query 

```` sql
SELECT 
    click,
    ROUND(100.0 * AVG(purchase), 2) AS purchase_percentage
FROM 
    campaign_analysis
GROUP BY 
    1;
````

#### Output Table

| click | purchase_percentage |
| ----- | ------------------- |
| 0     | 40.29               |
| 1     | 88.89               |

---


- What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?

#### Final Query 

```` sql
SELECT 
    impression,
    click,
    ROUND(100.0 * AVG(purchase), 2) AS purchase_percentage
FROM 
    campaign_analysis
GROUP BY 
    1, 2
ORDER BY 
    1, 2;
````
#### Output Table

| impression | click | purchase_percentage |
| ---------- | ----- | ------------------- |
| 0          | 0     | 38.69               |
| 1          | 0     | 64.94               |
| 1          | 1     | 88.89               |

---


- What metrics can you use to quantify the success or failure of each campaign compared to eachother?

#### Final Query

```` sql
SELECT 
    campaign_name,
    SUM(impression) AS no_of_impressions,
    SUM(click) AS no_of_clicks,
    ROUND(100.0 * SUM(click) / SUM(impression), 2) AS click_percentage
FROM 
    campaign_analysis
WHERE 
    impression > 0 
    AND campaign_name IS NOT NULL
GROUP BY 
    campaign_name;
````
#### Output Table

| campaign_name                     | no_of_impressions | no_of_clicks | click_percentage |
| --------------------------------- | ----------------- | ------------ | ---------------- |
| 25% Off - Living The Lux Life     | 114               | 88           | 77.19            |
| BOGOF - Fishing For Compliments   | 74                | 64           | 86.49            |
| Half Off - Treat Your Shellf(ish) | 585               | 469          | 80.17            |

- Why are there impressions and clicks when campaign_name is null? Maybe that data is for 3rd party ads. Unless I am making a really stupid mistake, there are impressions and clicks going on even when no campaign is going on. I cross check it for one visit id . Let me know if you think I've made a mistake somewhere.

---


### Please feel free to let me know if I have made any mistake or if you know a better approach to solve any question. If this helped you in anyway to improve your skills, just drop a message. It might not mean much to you, but it absolutely makes my day when I know that I’ve helped someone gain some knowledge.

### Anyways Happy Fiddling with the Data. See you in the next case study.
