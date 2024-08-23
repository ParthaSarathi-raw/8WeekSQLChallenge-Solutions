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

#### Final Query

#### Output Table
 
**6) What is the percentage of visits which view the checkout page but do not have a purchase event?**

#### Final Query

#### Output Table
 
**7) What are the top 3 pages by number of views?**

#### Final Query

#### Output Table
 
**8) What is the number of views and cart adds for each product category?**

#### Final Query

#### Output Table
 
**9) What are the top 3 products by purchases?**

#### Final Query

#### Output Table
 

## Product Funnel Analysis

**Using a single SQL query - create a new output table which has the following details:
How many times was each product viewed?
How many times was each product added to cart?
How many times was each product added to a cart but not purchased (abandoned)?
How many times was each product purchased?**

#### Final Query

#### Output Table
 
**Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.**

#### Final Query

#### Output Table
 
**Use your 2 new output tables - answer the following questions:**
**1) Which product had the most views, cart adds and purchases?**

#### Final Query

#### Output Table
 
**2) Which product was most likely to be abandoned?**

#### Final Query

#### Output Table
 
**3) Which product had the highest view to purchase percentage?**

#### Final Query

#### Output Table
 
**4) What is the average conversion rate from view to cart add?**

#### Final Query

#### Output Table
 
**5) What is the average conversion rate from cart add to purchase?**

#### Final Query

#### Output Table
 

## Campaigns Analysis
