# Case Study - 6 : Clique Bait

All the data and questions that I've answered in this markdown can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-6/).

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.

**You can directly skip to code block to directly view the solution. However if you are stuck understanding something I suggest you to read my approach on how I tackled the problem**\
**Also please run the code blocks youself on dbfiddle as you can look at the output and follow through my explanations when you are confused**

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
