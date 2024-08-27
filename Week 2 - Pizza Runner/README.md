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

### Creation of CTE

- Based on the questions being asked in this section, we can combine `customers_orders_clean`, `runner_orders_clean` and `pizza_runner.pizza_name` into CTE to use it everytime.

```` sql
	SELECT c.*,runner_id,pickup_time,distance,duration,cancellation,pizza_name
	FROM customers_orders_clean c 
	JOIN runner_orders_clean r 
	ON c.order_id = r.order_id 
	JOIN pizza_runner.pizza_names n on c.pizza_id = n.pizza_id
	ORDER BY c.order_id)
SELECT * FROM CTE;
````
| order_id | customer_id | pizza_id | exclusions | extras | order_time               | runner_id | pickup_time         | distance | duration | cancellation            | pizza_name |
| -------- | ----------- | -------- | ---------- | ------ | ------------------------ | --------- | ------------------- | -------- | -------- | ----------------------- | ---------- |
| 1        | 101         | 1        |            |        | 2020-01-01T18:05:02.000Z | 1         | 2020-01-01 18:15:34 | 20       | 32       |                         | Meatlovers |
| 2        | 101         | 1        |            |        | 2020-01-01T19:00:52.000Z | 1         | 2020-01-01 19:10:54 | 20       | 27       |                         | Meatlovers |
| 3        | 102         | 1        |            |        | 2020-01-02T23:51:23.000Z | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |                         | Meatlovers |
| 3        | 102         | 2        |            |        | 2020-01-02T23:51:23.000Z | 1         | 2020-01-03 00:12:37 | 13.4     | 20       |                         | Vegetarian |
| 4        | 103         | 2        | 4          |        | 2020-01-04T13:23:46.000Z | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         | Vegetarian |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         | Meatlovers |
| 4        | 103         | 1        | 4          |        | 2020-01-04T13:23:46.000Z | 2         | 2020-01-04 13:53:03 | 23.4     | 40       |                         | Meatlovers |
| 5        | 104         | 1        |            | 1      | 2020-01-08T21:00:29.000Z | 3         | 2020-01-08 21:10:57 | 10       | 15       |                         | Meatlovers |
| 6        | 101         | 2        |            |        | 2020-01-08T21:03:13.000Z | 3         |                     |          |          | Restaurant Cancellation | Vegetarian |
| 7        | 105         | 2        |            | 1      | 2020-01-08T21:20:29.000Z | 2         | 2020-01-08 21:30:45 | 25       | 25       |                         | Vegetarian |
| 8        | 102         | 1        |            |        | 2020-01-09T23:54:33.000Z | 2         | 2020-01-10 00:15:02 | 23.4     | 15       |                         | Meatlovers |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10T11:22:59.000Z | 2         |                     |          |          | Customer Cancellation   | Meatlovers |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2020-01-11T18:34:49.000Z | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         | Meatlovers |
| 10       | 104         | 1        |            |        | 2020-01-11T18:34:49.000Z | 1         | 2020-01-11 18:50:20 | 10       | 10       |                         | Meatlovers |

- We will be using this CTE to answer our questions
  
---

**1) How many pizzas were ordered?**

#### Final Query
```` sql
SELECT count(*) as no_of_pizzas_ordered FROM CTE;
````
#### Final Output

| no_of_pizzas_ordered |
| -------------------- |
| 14                   |

---

**2) How many unique customer orders were made?**

#### Final Query
```` sql
SELECT count(distinct order_id) as unique_no_of_orders FROM CTE;
````
#### Final Output

| unique_no_of_orders |
| ------------------- |
| 10                  |

---

**3) How many successful orders were delivered by each runner?**

#### Final Query
```` sql
SELECT runner_id,count(distinct order_id) assuccessful_orders_delivered
FROM CTE 
WHERE cancellation IS NULL
GROUP BY 1;
````
#### Final Output

| runner_id | successful_orders_delivered |
| --------- | --------------------------- |
| 1         | 4                           |
| 2         | 3                           |
| 3         | 1                           |

---

**4) How many of each type of pizza was delivered?**

#### Final Query
```` sql
SELECT pizza_name,count(*) as pizzas_delivered_successfully 
FROM CTE
WHERE cancellation is null
GROUP BY 1;
````
#### Final Output

| pizza_name | pizzas_delivered_successfully |
| ---------- | ----------------------------- |
| Meatlovers | 9                             |
| Vegetarian | 3                             |

---

**5) How many Vegetarian and Meatlovers were ordered by each customer?**

#### Final Query
```` sql
SELECT customer_id,
	sum(case when pizza_id = 1 then 1 else 0 end) as no_of_Meatlovers_ordered,
	sum(case when pizza_id = 2 then 1 else 0 end) as no_of_Vegetarian_ordered
FROM CTE
GROUP BY 1
ORDER BY 1;
````
#### Final Output

| customer_id | no_of_meatlovers_ordered | no_of_vegetarian_ordered |
| ----------- | ------------------------ | ------------------------ |
| 101         | 2                        | 1                        |
| 102         | 2                        | 1                        |
| 103         | 3                        | 1                        |
| 104         | 3                        | 0                        |
| 105         | 0                        | 1                        |

---

**6) What was the maximum number of pizzas delivered in a single order?**

#### Final Query
```` sql
SELECT count(*) as max_pizzas_delivered_in_single_order
FROM CTE
WHERE cancellation is NULL
GROUP BY order_id
ORDER BY 1 DESC
LIMIT 1;
````
#### Final Output

| max_pizzas_delivered_in_single_order |
| ------------------------------------ |
| 3                                    |

---

**7) For each customer, how many delivered pizzas had at least 1 change and how many had no changes?**

#### Final Query
```` sql
SELECT customer_id,
	sum(case when exclusions is null and extras is null then 1 else 0 end) as no_changes,
    sum(case when exclusions is not null or extras is not null then 1 else 0 end) as atleast_1_change
FROM CTE
WHERE cancellation is NULL
GROUP BY 1 ORDER BY 1;
````
#### Final Output

| customer_id | no_changes | atleast_1_change |
| ----------- | ---------- | ---------------- |
| 101         | 2          | 0                |
| 102         | 3          | 0                |
| 103         | 0          | 3                |
| 104         | 1          | 2                |
| 105         | 0          | 1                |

---

**8) How many pizzas were delivered that had both exclusions and extras?**

#### Final Query
```` sql
SELECT 
    sum(case when exclusions is not null and extras is not null then 1 else 0 end) as no_of_delivered_pizzas_with_both_exclusions_and_extras
FROM CTE
WHERE cancellation is NULL;
````
#### Final Output

| no_of_delivered_pizzas_with_both_exclusions_and_extras |
| ------------------------------------------------------ |
| 1                                                      |

---

**9) What was the total volume of pizzas ordered for each hour of the day?**

#### Final Query
```` sql
SELECT 
    extract(hour from order_time) as hour,count(*) as pizzas_ordered
FROM CTE
GROUP BY 1 ORDER BY 1;
````
#### Final Output

| hour | pizzas_ordered |
| ---- | -------------- |
| 11   | 1              |
| 13   | 3              |
| 18   | 3              |
| 19   | 1              |
| 21   | 3              |
| 23   | 3              |

---

**10) What was the volume of orders for each day of the week?**

#### Final Query
```` sql
SELECT 
    TO_CHAR(order_time,'day') as day,count(*) as pizzas_ordered
FROM CTE
GROUP BY 1;
````
#### Final Output
 

| day       | pizzas_ordered |
| --------- | -------------- |
| wednesday | 5              |
| thursday  | 3              |
| friday    | 1              |
| saturday  | 5              |

---

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


