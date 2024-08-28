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
WITH CTE as(
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
```` sql
SELECT 
   ((registration_date - '2021-01-01')/7)+1 as week_number,
   registration_date - ((registration_date-'2021-01-01')%7) as week_start_date,
   count(*)
FROM pizza_runner.runners
GROUP BY 1,2 ORDER BY 2;
````
#### Final Output

| week_number | week_start_date          | count |
| ----------- | ------------------------ | ----- |
| 1           | 2021-01-01T00:00:00.000Z | 2     |
| 2           | 2021-01-08T00:00:00.000Z | 1     |
| 3           | 2021-01-15T00:00:00.000Z | 1     |

---

**2) What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

#### Final Query
```` sql
SELECT runner_id,
	round(avg(extract(minute from pickup_time::timestamp-order_time))) as minutes_took_to_reach_HQ 
FROM(
	SELECT order_id,runner_id,order_time,pickup_time
	FROM CTE
	WHERE pickup_time is not null
	GROUP BY 1,2,3,4
)temp
GROUP BY 1 ORDER BY 1;
````
#### Final Output

| runner_id | minutes_took_to_reach_hq |
| --------- | ------------------------ |
| 1         | 14                       |
| 2         | 20                       |
| 3         | 10                       |

---

**3) Is there any relationship between the number of pizzas and how long the order takes to prepare?**

#### Final Query
```` sql
SELECT count as no_of_pizzas,avg(time_taken_to_prep) as avg_time_taken_to_prep
FROM (
	SELECT order_id,count(*),
  	extract(minute from pickup_time::timestamp-order_time) as time_taken_to_prep FROM CTE
	WHERE pickup_time is not NULL
	GROUP BY 1,3
) temp
GROUP BY 1 ORDER BY 1;
````
#### Final Output

| no_of_pizzas | avg_time_taken_to_prep |
| ------------ | ---------------------- |
| 1            | 12                     |
| 2            | 18                     |
| 3            | 29                     |

---

**4) What was the average distance travelled for each customer?**

#### Final Query
```` sql
SELECT customer_id,round(avg(distance),2) as avg_distance_travelled FROM CTE
WHERE cancellation is null
GROUP BY 1 ORDER BY 1;
````
#### Final Output

| customer_id | avg_distance_travelled |
| ----------- | ---------------------- |
| 101         | 20.00                  |
| 102         | 16.73                  |
| 103         | 23.40                  |
| 104         | 10.00                  |
| 105         | 25.00                  |

---
**5) What was the difference between the longest and shortest delivery times for all orders?**

#### Final Query
```` sql
SELECT max(duration) - min(duration) as diff_between_longest_and_shortest_delivery_time FROM CTE
WHERE cancellation is null;
````
#### Final Output

| diff_between_longest_and_shortest_delivery_time |
| ----------------------------------------------- |
| 30                                              |

---

**6) What was the average speed for each runner for each delivery and do you notice any trend for these values?**

#### Final Query
```` sql
SELECT runner_id,round(avg(60.0*distance/duration),2) as avg_speed_in_km_per_hr 
FROM (
	SELECT runner_id,order_id,distance,duration FROM CTE
	WHERE cancellation is null
	GROUP BY 1,2,3,4
) temp
GROUP BY 1 ORDER BY 1;
````
#### Final Output

| runner_id | avg_speed_in_km_per_hr |
| --------- | ---------------------- |
| 1         | 45.54                  |
| 2         | 62.90                  |
| 3         | 40.00                  |

---

**7) What is the successful delivery percentage for each runner?**

#### Final Query
```` sql
SELECT runner_id,
round(100.0*sum(case when cancellation is null then 1 else 0 end)/count(*),2) as successful_delivery_percentage 
FROM (
	SELECT runner_id,order_id,cancellation FROM CTE GROUP BY 1,2,3
) temp
GROUP BY 1 ORDER BY 1;
````
#### Final Output

| runner_id | successful_delivery_percentage |
| --------- | ------------------------------ |
| 1         | 100.00                         |
| 2         | 75.00                          |
| 3         | 50.00                          |

---

## Ingredient Optimization

- Before we dive into questions, I want to combine `pizza_names`,`pizza_recipes` and `pizza_toppings` tables to make our calculations a bit easier.

```` sql
WITH COOK_BOOK AS (
    SELECT 
        pizza_id, pizza_name, t.topping_id, t.topping_name 
    FROM (
        SELECT 
            n.*, unnest(string_to_array(toppings, ', ')) AS topping_id 
        FROM 
            pizza_runner.pizza_names n
        JOIN 
            pizza_runner.pizza_recipes r ON n.pizza_id = r.pizza_id
    ) temp
    JOIN 
        pizza_runner.pizza_toppings t ON temp.topping_id::integer = t.topping_id
    ORDER BY 
        1, 3
)
SELECT * FROM COOK_BOOK;
````
| pizza_id | pizza_name | topping_id | topping_name |
| -------- | ---------- | ---------- | ------------ |
| 1        | Meatlovers | 1          | Bacon        |
| 1        | Meatlovers | 2          | BBQ Sauce    |
| 1        | Meatlovers | 3          | Beef         |
| 1        | Meatlovers | 4          | Cheese       |
| 1        | Meatlovers | 5          | Chicken      |
| 1        | Meatlovers | 6          | Mushrooms    |
| 1        | Meatlovers | 8          | Pepperoni    |
| 1        | Meatlovers | 10         | Salami       |
| 2        | Vegetarian | 4          | Cheese       |
| 2        | Vegetarian | 6          | Mushrooms    |
| 2        | Vegetarian | 7          | Onions       |
| 2        | Vegetarian | 9          | Peppers      |
| 2        | Vegetarian | 11         | Tomatoes     |
| 2        | Vegetarian | 12         | Tomato Sauce |

**1) What are the standard ingredients for each pizza?**

#### Final Query :
```` sql
SELECT pizza_name,string_agg(topping_name,', ') as standard_ingredients 
FROM COOK_BOOK GROUP BY 1;
````

| pizza_name | standard_ingredients                                                  |
| ---------- | --------------------------------------------------------------------- |
| Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |

---

**2) What was the most commonly added extra?**

#### Final Query :
```` sql
SELECT 
    topping_name AS most_common_extra 
FROM (
    SELECT 
        topping_name,
        COUNT(*),
        DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS dn 
    FROM (
        SELECT 
            unnest(string_to_array(extras, ', ')) AS extra_topping_id 
        FROM 
            customers_orders_clean
    ) temp 
    JOIN 
        pizza_runner.pizza_toppings t ON temp.extra_topping_id::integer = t.topping_id
    GROUP BY 
        1
) temp 
WHERE 
    dn = 1;
````

#### Output Table

| most_common_extra |
| ----------------- |
| Bacon             |

---

**3) What was the most common exclusion?**

#### Final Query

```` sql
SELECT 
    topping_name AS most_excluded 
FROM (
    SELECT 
        topping_name,
        COUNT(*),
        DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS dn 
    FROM (
        SELECT 
            unnest(string_to_array(exclusions, ', ')) AS excluded_topping_id 
        FROM 
            customers_orders_clean
    ) temp 
    JOIN 
        pizza_runner.pizza_toppings t ON temp.excluded_topping_id::integer = t.topping_id
    GROUP BY 
        1
) temp 
WHERE 
    dn = 1;
````

#### Output Table

| most_excluded |
| ------------- |
| Cheese        |

---

**4) Generate an order item for each record in the customers_orders table in the format of one of the following:**

   - Meat Lovers
  -  Meat Lovers - Exclude Beef
   - Meat Lovers - Extra Bacon
   - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

**Note : I have used row_number to index all the rows in customers_orders_clean so it will be easier to combine the tables.**

**Note2 : Again, this is not meant to be an easy question. I'm just posting my complete solution in the query. If you've gone through case study 1 and 3 - 8 , then I'm pretty sure you can look at my solution and understand my approach. However if you still don't get it, please reach out to me, I will update the solution with detailed explanations.**

#### Final Query :
```` sql
WITH customers_orders_clean AS (
    SELECT 
        row_number() OVER() AS rn,
        order_id,
        customer_id,
        pizza_id,
        CASE 
            WHEN exclusions = '' OR exclusions = 'null' THEN NULL 
            ELSE exclusions 
        END AS exclusions,
        CASE 
            WHEN extras = '' OR extras = 'null' THEN NULL 
            ELSE extras 
        END AS extras,
        order_time
    FROM 
        pizza_runner.customer_orders
),
extras AS (
    SELECT 
        rn,
        string_agg(topping_name, ', ') AS extras_list 
    FROM (
        SELECT 
            rn,
            unnest(string_to_array(extras, ', ')) AS extra_id 
        FROM 
            customers_orders_clean
    ) temp
    JOIN 
        pizza_runner.pizza_toppings t ON temp.extra_id::integer = t.topping_id
    GROUP BY 
        rn
),
exclusions AS (
    SELECT 
        rn,
        string_agg(topping_name, ', ') AS exclusions_list 
    FROM (
        SELECT 
            rn,
            unnest(string_to_array(exclusions, ', ')) AS exclusion_id 
        FROM 
            customers_orders_clean
    ) temp
    JOIN 
        pizza_runner.pizza_toppings t ON temp.exclusion_id::integer = t.topping_id
    GROUP BY 
        rn
)
SELECT 
    order_id,
    CASE 
        WHEN extras IS NULL AND exclusions IS NULL THEN pizza_name
        WHEN extras IS NOT NULL AND exclusions IS NULL THEN concat(pizza_name, ' - Extra ', extras_list)
        WHEN extras IS NULL AND exclusions IS NOT NULL THEN concat(pizza_name, ' - Exclude ', exclusions_list)
        ELSE concat(pizza_name, ' - Exclude ', exclusions_list, ' - Extra ', extras_list) 
    END AS order_list
FROM 
    customers_orders_clean c
LEFT JOIN 
    extras ON c.rn = extras.rn
LEFT JOIN 
    exclusions ON c.rn = exclusions.rn
LEFT JOIN 
    pizza_runner.pizza_names n ON c.pizza_id = n.pizza_id;
````

#### Output Table

| order_id | order_list                                                      |
| -------- | --------------------------------------------------------------- |
| 1        | Meatlovers                                                      |
| 2        | Meatlovers                                                      |
| 3        | Meatlovers                                                      |
| 3        | Vegetarian                                                      |
| 4        | Meatlovers - Exclude Cheese                                     |
| 4        | Meatlovers - Exclude Cheese                                     |
| 4        | Vegetarian - Exclude Cheese                                     |
| 5        | Meatlovers - Extra Bacon                                        |
| 6        | Vegetarian                                                      |
| 7        | Vegetarian - Extra Bacon                                        |
| 8        | Meatlovers                                                      |
| 9        | Meatlovers - Exclude Cheese - Extra Bacon, Chicken              |
| 10       | Meatlovers                                                      |
| 10       | Meatlovers - Exclude BBQ Sauce, Mushrooms - Extra Bacon, Cheese |

- Again it might look like some big complex query at first, but if you go through each step it's really simple.
- It just involves the whole process of breaking down extras and exclusions list and then combining with their respective names and rejoining them, nothing complicated, just looks big that's it.
- Once you understand what's going in, it is just piece of cake.

---

**5) Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients**

   - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

#### Final Query
```` sql
WITH customers_orders_clean AS (
    SELECT 
        row_number() OVER() AS rn,
        order_id, customer_id, pizza_id,
        CASE WHEN exclusions = '' OR exclusions = 'null' THEN NULL ELSE exclusions END AS exclusions,
        CASE WHEN extras = '' OR extras = 'null' THEN NULL ELSE extras END AS extras,
        order_time
    FROM pizza_runner.customer_orders
),
COOK_BOOK AS (
    SELECT 
        pizza_id, pizza_name, t.topping_id, t.topping_name 
    FROM (
        SELECT 
            n.*, unnest(string_to_array(toppings, ', ')) AS topping_id 
        FROM pizza_runner.pizza_names n
        JOIN pizza_runner.pizza_recipes r ON n.pizza_id = r.pizza_id
    ) temp
    JOIN pizza_runner.pizza_toppings t ON temp.topping_id::integer = t.topping_id
    ORDER BY 1, 3
),
extras AS (
    SELECT 
        rn, topping_id, topping_name 
    FROM (
        SELECT 
            rn, unnest(string_to_array(extras, ', ')) AS extra_id 
        FROM customers_orders_clean
    ) temp
    JOIN pizza_runner.pizza_toppings t ON temp.extra_id::integer = t.topping_id
),
exclusions AS (
    SELECT 
        rn, topping_id, topping_name AS exclusion_name
    FROM (
        SELECT 
            rn, unnest(string_to_array(exclusions, ', ')) AS exclusion_id 
        FROM customers_orders_clean
    ) temp
    JOIN pizza_runner.pizza_toppings t ON temp.exclusion_id::integer = t.topping_id
),
ingredients AS (
    SELECT 
        c.rn, b.topping_id, topping_name, exclusion_name 
    FROM customers_orders_clean c 
    JOIN COOK_BOOK b ON c.pizza_id = b.pizza_id
    LEFT JOIN exclusions ON c.rn = exclusions.rn AND b.topping_name = exclusions.exclusion_name
    UNION ALL
    SELECT rn, topping_id, topping_name, NULL AS exclusion_name 
    FROM extras
),
ingredient_list AS (
    SELECT 
        rn, string_agg(
            CASE WHEN cnt = 1 THEN topping_name ELSE concat(cnt::text, 'x ', topping_name) END, ', ' 
            ORDER BY topping_id
        ) AS ingredients_list 
    FROM (
        SELECT 
            rn, topping_id, topping_name, COUNT(*) AS cnt 
        FROM ingredients 
        WHERE exclusion_name IS NULL
        GROUP BY 1, 2, 3 
        ORDER BY 1, 2
    ) temp
    GROUP BY rn
)
SELECT 
    order_id, ingredients_list 
FROM 
    customers_orders_clean c 
JOIN 
    ingredient_list l ON c.rn = l.rn;
````
- Funfact : This might be the longest query I've written while solving all 8 case studies, but it just ran in 3.7 ms lol.
- Also we are still playing by danny's rules. Each and every question in this case study can be answered using 1 query only. I want to highlight that the above big query while it maybe big, is just a singular query.

#### Output Table

| order_id | ingredients_list                                                         |
| -------- | ------------------------------------------------------------------------ |
| 1        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 2        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3        | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce               |
| 4        | Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami            |
| 4        | Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami            |
| 4        | Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce                       |
| 5        | 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 6        | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce               |
| 7        | Bacon, Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce        |
| 8        | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 9        | 2x Bacon, BBQ Sauce, Beef, 2x Chicken, Mushrooms, Pepperoni, Salami      |
| 10       | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 10       | 2x Bacon, Beef, 2x Cheese, Chicken, Pepperoni, Salami                    |

---

**6) What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**

- We are going to use `ingredients` table that we have created as CTE in the previous solution to directly get the answer.

#### Final Query
```` sql
SELECT topping_name as ingredient,count(*) as no_of_times_ingredient_used FROM ingredients 
WHERE exclusion_name is NULL
GROUP BY 1
ORDER BY 2 DESC;
````
- Yay finally a short and sweet query lol.

#### Output Table

| ingredient   | no_of_times_ingredient_used |
| ------------ | --------------------------- |
| Bacon        | 14                          |
| Mushrooms    | 13                          |
| Chicken      | 11                          |
| Cheese       | 11                          |
| Pepperoni    | 10                          |
| Salami       | 10                          |
| Beef         | 10                          |
| BBQ Sauce    | 9                           |
| Tomato Sauce | 4                           |
| Onions       | 4                           |
| Tomatoes     | 4                           |
| Peppers      | 4                           |

---

## Pricing and Ratings

**1) If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**

#### Final Query

```` sql
SELECT 
    sum(case when pizza_id = 1 then 12 else 10 end) as "total_amount_made ($)" 
FROM 
    CTE
WHERE cancellation is NULL;
````
#### Output Table

| total_amount_made ($) |
| --------------------- |
| 138                   |

---

**2) What if there was an additional $1 charge for any pizza extras?**

#### Final Query

```` sql
SELECT 
    SUM(CASE WHEN pizza_id = 1 THEN 12 ELSE 10 END) + SUM(no_of_extras) AS total_amount_earned
FROM 
    CTE 
LEFT JOIN (
    SELECT 
        rn, COUNT(*) AS no_of_extras
    FROM 
        extras
    GROUP BY 
        rn
) temp 
ON 
    cte.rn = temp.rn
WHERE cancellation is NULL;
````

#### Output Table

| total_amount_earned |
| ------------------- |
| 142                 |

---

**3) The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.**

#### Final Query

```` sql
CREATE TABLE runner_ratings (
  order_id integer,
  runner_id integer,
  rating integer,
  feedback text
);

INSERT INTO runner_ratings(order_id,runner_id,rating,feedback) values
(1,1,5,'Delivered on time'),
(2,1,5,NULL),
(3,1,3,'Quick delivery but delivery boy was rude'),
(4,2,1,'Delivered very late and pizza was cold'),
(5,3,5,'Fast like Flash , I love it'),
(7,2,4,NULL),
(8,2,5,'I''m telling you this guy can compete in Moto Grand Prix'),
(10,1,5,'Blink it and you miss it');

SELECT * FROM runner_ratings;
````

#### Output Table

| order_id | runner_id | rating | feedback                                                |
| -------- | --------- | ------ | ------------------------------------------------------- |
| 1        | 1         | 5      | Delivered on time                                       |
| 2        | 1         | 5      |                                                         |
| 3        | 1         | 3      | Quick delivery but delivery boy was rude                |
| 4        | 2         | 1      | Delivered very late and pizza was cold                  |
| 5        | 3         | 5      | Fast like Flash , I love it                             |
| 7        | 2         | 4      |                                                         |
| 8        | 2         | 5      | I'm telling you this guy can compete in Moto Grand Prix |
| 10       | 1         | 5      | Blink it and you miss it                                |

---

**4) Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?**
- customer_id
- order_id
- runner_id
- rating
- order_time
- pickup_time
- Time between order and pickup
- Delivery duration
- Average speed
- Total number of pizzas

#### Final Query

```` sql
SELECT 
    customer_id,
    CTE.order_id,
    CTE.runner_id,
    rating,
    order_time,
    pickup_time,
    EXTRACT(MINUTE FROM pickup_time::TIMESTAMP - order_time) AS time_between_order_and_pickup_in_minutes,
    duration AS delivery_duration_in_minutes,
    ROUND(60.0 * distance / duration, 2) AS avg_speed_in_km_per_hr,
    COUNT(*) AS no_of_pizzas
FROM CTE
JOIN runner_ratings r ON CTE.order_id = r.order_id
WHERE cancellation IS NULL
GROUP BY 
    customer_id, 
    CTE.order_id, 
    CTE.runner_id, 
    rating, 
    order_time, 
    pickup_time, 
    duration, 
    distance
ORDER BY 
    customer_id, 
    CTE.order_id;
````

#### Output Table

| customer_id | order_id | runner_id | rating | order_time               | pickup_time         | time_between_order_and_pickup_in_minutes | delivery_duration_in_minutes | avg_speed_in_km_per_hr | no_of_pizzas |
| ----------- | -------- | --------- | ------ | ------------------------ | ------------------- | ---------------------------------------- | ---------------------------- | ---------------------- | ------------ |
| 101         | 1        | 1         | 5      | 2020-01-01T18:05:02.000Z | 2020-01-01 18:15:34 | 10                                       | 32                           | 37.50                  | 1            |
| 101         | 2        | 1         | 5      | 2020-01-01T19:00:52.000Z | 2020-01-01 19:10:54 | 10                                       | 27                           | 44.44                  | 1            |
| 102         | 3        | 1         | 3      | 2020-01-02T23:51:23.000Z | 2020-01-03 00:12:37 | 21                                       | 20                           | 40.20                  | 2            |
| 102         | 8        | 2         | 5      | 2020-01-09T23:54:33.000Z | 2020-01-10 00:15:02 | 20                                       | 15                           | 93.60                  | 1            |
| 103         | 4        | 2         | 1      | 2020-01-04T13:23:46.000Z | 2020-01-04 13:53:03 | 29                                       | 40                           | 35.10                  | 3            |
| 104         | 5        | 3         | 5      | 2020-01-08T21:00:29.000Z | 2020-01-08 21:10:57 | 10                                       | 15                           | 40.00                  | 1            |
| 104         | 10       | 1         | 5      | 2020-01-11T18:34:49.000Z | 2020-01-11 18:50:20 | 15                                       | 10                           | 60.00                  | 2            |
| 105         | 7        | 2         | 4      | 2020-01-08T21:20:29.000Z | 2020-01-08 21:30:45 | 10                                       | 25                           | 60.00                  | 1            |

---

**5) If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?**

#### Final Query

```` sql
SELECT (
SELECT 
    sum(case when pizza_id = 1 then 12 else 10 end) as "total_amount_made ($)" 
FROM 
    CTE
WHERE cancellation is NULL)
-
(SELECT round(0.30*sum(distance),2) as paid_to_runners FROM runner_orders_clean WHERE cancellation is NULL) as left_over_money_in_$   ;
````
#### Output Table

| left_over_money_in_$ |
| -------------------- |
| 94.44                |

---

## Bonus Question

**If Danny wants to expand his range of pizzas - how would this impact the existing data design? Write an INSERT statement to demonstrate what would happen if a new Supreme pizza with all the toppings was added to the Pizza Runner menu?**

- Since Danny has structed the pizza runner very well, it won't be an issue to add any new pizzas. All our previous queries will work fine with multiple new pizzas added.
- Just we need to update 2 tables only.

#### Final Query
```` sql
INSERT INTO pizza_runner.pizza_names VALUES (3,'Supreme');
INSERT INTO pizza_runner.pizza_recipes VALUES (3,'1, 2, 3, 4, 5, 6, 7, 8, 9, 10');
SELECT * FROM pizza_runner.pizza_names;
SELECT * FROM pizza_runner.pizza_recipes;
````
#### Output Table

| pizza_id | pizza_name |
| -------- | ---------- |
| 1        | Meatlovers |
| 2        | Vegetarian |
| 3        | Supreme    |


| pizza_id | toppings                      |
| -------- | ----------------------------- |
| 1        | 1, 2, 3, 4, 5, 6, 8, 10       |
| 2        | 4, 6, 7, 9, 11, 12            |
| 3        | 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 |

### Checking if our queries are working properly or not.

- To check I will add a new order and run the query of Ingredient Optimization Question 4. This is because I believe this is the most complex query that we have created.
- If this query works perfectly fine, then most probably every thing else works fine.
- After inserting all the required data we can see that the new pizza is being shown.
  
| 11       | Supreme - Exclude Bacon, Onions - Extra Chicken, Mushrooms      |

- So everything is working fine.

---

### Please feel free to let me know if I have made any mistake or if you know a better approach to solve any question. If this helped you in anyway to improve your skills, just drop a message. It might not mean much to you, but it absolutely makes my day when I know that I’ve helped someone gain some knowledge.

### Anyways Happy Fiddling with the Data. See you in the next case study.

### Oh wait, this is the final case study. It's been a ride solving all these 8 case studies. Hopefully all these solutions I've provided really came in handy while you're solving these case studies yourself. I wish you all the best in your future endeavours. Peace <3
