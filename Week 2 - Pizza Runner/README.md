# Case Study - 2 : Pizza Runner

All the data and questions that I've answered in this markdown can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-2/).

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.

**Note : If you doing this week by week, I suggest you to come to Week 2's case study at the end because this is the most difficut out of all others. However as this will be the last case study I will be attempting, I'm assuming you are already familiar with all the concepts and I will keep my explanation to be minimal. If you still didn't get anything, feel free to leave a comment and I will update my explanation.**

## Entity Relationship Diagram

<img width="587" alt="image" src="https://github.com/user-attachments/assets/2f230655-cd2b-4f25-8882-01c4f4f21517">

- There are a lot of tables, so it would be better to combine tables on the fly based on the requirement rather than using a common CTE to answer all the questions.

## Data Cleaning

- Before we dive into solving the questions, first we have to clean up the data.

```` sql
WITH customers_orders_clean as (
SELECT order_id,customer_id,pizza_id,
case when exclusions = '' or exclusions = 'null' then NULL else exclusions end as exclusions,
case when extras = '' or extras = 'null' then NULL else extras end as extras,
order_time
FROM pizza_runner.customer_orders)
SELECT * FROM customers_orders_clean;
````
| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ |
| 1        | 101         | 1        |            |        | 2020-01-01T18:05:02.000Z |
| 2        | 101         | 1        |            |        | 2020-01-01T19:00:52.000Z |
| 3        | 102         | 1        |            |        | 2020-01-02T23:51:23.000Z |
| 3        | 102         | 2        |            |        | 2020-01-02T23:51:23.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z |
| 4        | 103         | 2        | 4          |        | 2020-01-04T13:23:46.000Z |
| 5        | 104         | 1        |            | 1      | 2020-01-08T21:00:29.000Z |
| 6        | 101         | 2        |            |        | 2020-01-08T21:03:13.000Z |
| 7        | 105         | 2        |            | 1      | 2020-01-08T21:20:29.000Z |
| 8        | 102         | 1        |            |        | 2020-01-09T23:54:33.000Z |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z |
| 10       | 104         | 1        |            |        | 2020-01-11T18:34:49.000Z |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z |

```` sql
WITH runner_orders_clean as (
SELECT order_id,runner_id,
case when pickup_time = '' or pickup_time = 'null' then NULL else pickup_time end as pickup_time,
case when distance = '' or distance = 'null' then NULL else trim('km' from distance)::decimal end as distance,
case when duration = '' or duration = 'null' then NULL when duration like '%mins' then trim('mins' from duration)::integer when duration like '%minute' then trim('minute' from duration)::integer when duration like '%minutes' then trim('minutes' from duration)::integer else duration::integer end as duration,
case when cancellation = '' or cancellation = 'null' then NULL else cancellation end as cancellation 
FROM pizza_runner.runner_orders)
SELECT * FROM runner_orders_clean;
````
| order_id | runner_id | pickup_time         | distance | duration | cancellation            |
| -------- | --------- | ------------------- | -------- | -------- | ----------------------- |
| 1        | 1         | 2020-01-01 18:15:34 | 20       | 32       |                         |
| 2        | 1         | 2020-01-01 19:10:54 | 20       | 27       |                         |
| 3        | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |                         |
| 4        | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         |
| 5        | 3         | 2020-01-08 21:10:57 | 10       | 15       |                         |
| 6        | 3         |                     |          |          | Restaurant Cancellation |
| 7        | 2         | 2020-01-08 21:30:45 | 25       | 25       |                         |
| 8        | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                         |
| 9        | 2         |                     |          |          | Customer Cancellation   |
| 10       | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         |

- We will be using these cleaned up tables to do the analysis.

  
---

## Pizza Metrics
**1) How many pizzas were ordered?**

#### Final Query

#### Final Output
 
**2) How many unique customer orders were made?**

#### Final Query

#### Final Output
 
**3) How many successful orders were delivered by each runner?**

#### Final Query

#### Final Output
 
**4) How many of each type of pizza was delivered?**

#### Final Query

#### Final Output
 
**5) How many Vegetarian and Meatlovers were ordered by each customer?**

#### Final Query

#### Final Output
 
**6) What was the maximum number of pizzas delivered in a single order?**

#### Final Query

#### Final Output
 
**7) For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

#### Final Query

#### Final Output
 
**8) How many pizzas were delivered that had both exclusions and extras?**

#### Final Query

#### Final Output
 
**9) What was the total volume of pizzas ordered for each hour of the day?**

#### Final Query

#### Final Output
 
**10) What was the volume of orders for each day of the week?**

#### Final Query

#### Final Output
 

## Runner and Customer Experience

**1) How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**

#### Final Query

#### Final Output
 
**2) What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

#### Final Query

#### Final Output
 
**3) Is there any relationship between the number of pizzas and how long the order takes to prepare?**

#### Final Query

#### Final Output
 
**4) What was the average distance travelled for each customer?**

#### Final Query

#### Final Output
 
**5) What was the difference between the longest and shortest delivery times for all orders?**

#### Final Query

#### Final Output
 
**6) What was the average speed for each runner for each delivery and do you notice any trend for these values?**

#### Final Query

#### Final Output
 
**7) What is the successful delivery percentage for each runner?**

#### Final Query

#### Final Output
 

## Ingredient Optimization


## Pricing and Ratings


