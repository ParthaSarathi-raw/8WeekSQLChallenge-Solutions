# Case Study - 3 : Foodie-Fi
All the data and questions that I've answered in this markdown can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-3/).

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/rHJhRrXy5hbVBNJ6F6b9gJ/16) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.

**You can directly skip to code block to directly view the solution. However if you are stuck understanding something I suggest you to read my approach on how I tackled the problem.**\
**Also please run the code blocks youself on dbfiddle as you can look at the output and follow through my explanations when you are confused.**

## Entity Relationship Diagram
<img width="493" alt="image" src="https://github.com/user-attachments/assets/0ee18517-5e6a-4516-8b15-84b31215d7ed">
<br>
Based on the given data, there are only 2 tables, so I think would be easier if we create a CTE by joining these both tables and performing our required operations on them.

## Creation of CTE
````sql
WITH cte AS (
    SELECT s.*, plan_name, price
    FROM foodie_fi.subscriptions s
    JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
    ORDER BY customer_id, start_date
)
SELECT * FROM cte;
````
#### Base CTE Table

| customer_id | plan_id | start_date               | plan_name     | price  |
| ----------- | ------- | ------------------------ | ------------- | ------ |
| 1           | 0       | 2020-08-01T00:00:00.000Z | trial         | 0.00   |
| 1           | 1       | 2020-08-08T00:00:00.000Z | basic monthly | 9.90   |
| 2           | 0       | 2020-09-20T00:00:00.000Z | trial         | 0.00   |
| 2           | 3       | 2020-09-27T00:00:00.000Z | pro annual    | 199.00 |
| 3           | 0       | 2020-01-13T00:00:00.000Z | trial         | 0.00   |
| 3           | 1       | 2020-01-20T00:00:00.000Z | basic monthly | 9.90   |
| 4           | 0       | 2020-01-17T00:00:00.000Z | trial         | 0.00   |
| 4           | 1       | 2020-01-24T00:00:00.000Z | basic monthly | 9.90   |
| 4           | 4       | 2020-04-21T00:00:00.000Z | churn         |        |
| 5           | 0       | 2020-08-03T00:00:00.000Z | trial         | 0.00   |

---
## Customer Journey
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.\
Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!
\
\
**Note : You can even skip this question and get to the data analysis part as I personally don’t think this question will help your SQL skills.**
\
\
This is a very open ended question, I am not going to answer it because there is no correct answer to this and not much SQL is needed to answer this. We have already joined both tables. On the original 8weeksql page, there are 8 sample customers provide in the subscriptions example table, just write a brief description of each of these sample customers journey after joining foodie_fi plan.

## Data Analysis Solutions

**1. How many customers has Foodie-Fi ever had?**
We want the count of customers right?\
So this can simply be achieved by doing `count(distinct customer_id)` , not much explanation needed for this.
#### Final Query
```` sql
SELECT count(distinct customer_id) as no_of_customers FROM CTE;
````
#### Output Table
| no_of_customers |
| --------------- |
| 1000            |

---


**2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value**
We want the monthly distribution, so first we need to get month from the `start_date`
- This can be done using extract function : `extract (month from start_date)`
- We are only interested in trail plans only so : `WHERE plan_id = 0`
- We need the count of trail_plan  so : `count(customer_id)`
Notice how I didn’t use distinct here, because each customer will only take trial plan once at the beginning their foodie fi plan, so there won’t be any duplicates
- Next we gotta aggregate the data for each month so : `GROUP BY extract(month from start_date)`

#### Final Query
```` sql
SELECT EXTRACT(month FROM start_date) AS month, 
       COUNT(customer_id) AS no_of_customers
FROM CTE
WHERE plan_id = 0
GROUP BY 1
ORDER BY 1;
````
#### Output Table
 
| month | no_of_customers |
| ----- | --------------- |
| 1     | 88              |
| 2     | 68              |
| 3     | 94              |
| 4     | 81              |
| 5     | 88              |
| 6     | 79              |
| 7     | 89              |
| 8     | 88              |
| 9     | 87              |
| 10    | 79              |
| 11    | 75              |
| 12    | 84              |

---


**3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name**
- We are interested in the plans which have start_date after year 2020 so : `WHERE start_date >= ‘2021-01-01’` or `extract(year from start_date)>2020`
- We want to breakdown by count of events  so : `count(plan_id)` or `count(plan_name)`
- We want to aggreage this for each plan name so : `GROUP BY plan_id,plan_name`
- Any order is not necessary, but it would be good to have it ordered by plan_id so : `ORDER BY plan_id`
Combining all this the SQL query we get the final query

#### Final Query
```` sql
SELECT plan_id, plan_name, COUNT(plan_name) AS no_of_plans
FROM CTE
WHERE start_date >= '2021-01-01'
GROUP BY plan_id, plan_name
ORDER BY plan_id;
````
#### Output Table
 
| plan_id | plan_name     | no_of_plans |
| ------- | ------------- | ----------- |
| 1       | basic monthly | 8           |
| 2       | pro monthly   | 60          |
| 3       | pro annual    | 63          |
| 4       | churn         | 71          |

---

**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**
### Approach 1 : Using SubQuery
- The first intuition after reading this question is to do : `count(customer_id) WHERE plan_id = 4`. However you can simply get the count, but what about percentage? 
- You see when you try to divide by `count(distinct customer_id)` you won’t get all customers, because where condition is executed first and filters everyone , so `count(distinct customer_id)` will give the same as `count(customer_id)` = 317 because they are the customers who have churned. 
So to get the total customers you can apply subquery in the denominator.
```` sql
(SELECT count(distinct customer_id) FROM CTE)
````
This gives the total customers i.e 1000.\
So after dividing this and multiplying to 100.0 to convert to percentage and doing round off, we get the following final query.


#### Final Query
```` sql
SELECT 
    COUNT(customer_id), 
    ROUND(100.0 * COUNT(customer_id) / (SELECT COUNT(DISTINCT customer_id) FROM CTE), 1) AS percentage_of_customers
FROM 
    CTE
WHERE 
    plan_id = 4;
````
#### Output Table
 
| count | percentage_of_customers |
| ----- | ----------------------- |
| 307   | 30.7                    |

### Approach 2 : Using Sum(Case)
Instead of putting `WHERE plan_id = 4` and doing subquery for denominator, we can do sum(case statement)
- `sum(case when plan_id = 4 then 1 else 0 end)` -> this will add up each and every `customer_id` who had churned. So now when you do `count(distinct customer_id)` it will give the correct count now since you didn’t apply any filters.

#### Final Query 2
```` sql
SELECT 
    SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END) AS count,
    ROUND(
        100.0 * SUM(CASE WHEN plan_id = 4 THEN 1 ELSE 0 END) / 
        COUNT(DISTINCT customer_id), 
        1
    ) AS percentage
FROM 
    CTE;
````
- Both the methods are correct and I personally prefer the 2nd approach, but its totally personal preference.
---

**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**
We can solve this by using either lead/lag functions or by using row_number. Let us see both approaches.
### Approach 1 : Using Row Number
- I don’t know if you have noticed this, but every customer of foodie_fi needs to start their journey by first taking a free trial. Atleast that is what I understood from the data. So this approach banks on the fact that the initial trial 
- Lets say for example if a person started their plan by directly taking monthly subscription and churned right after. Our approach will consider this as valid person and counts. But they didn’t take a free trial and churned right after that. So I want you to keep this in mind while we see the row_number() approach. Again this approach only works because we are going with the fact the no matter who it is, everyone starts their foodie_fi experience with a free trial.

Coming to SQL part, first we will assign row number to each and every customer ordered on their start_date.

```` sql
SELECT cte.*,row_number() OVER (PARTITION BY customer_id ORDER BY start_date) as rn FROM CTE
````
- Then from this table we just gotta check if `plan_id = 4` for `rn = 2` because we already know that `rn = 1` will be free trial only.
- We can put these conditions on where clause and apply `subquery` for denominator while calculating percentage. Or we can use `sum(case)` as well, you please follow whatever you prefer.

#### Final Query
```` sql
SELECT 
    ROUND(
        100.0 * SUM(CASE WHEN rn = 2 AND plan_id = 4 THEN 1 ELSE 0 END) / 
        COUNT(DISTINCT customer_id), 
        1
    ) AS percentage_churned
FROM (
    SELECT 
        cte.*, 
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS rn
    FROM 
        CTE
) temp;
````
#### Output Table
 
| percentage_churned |
| ------------------ |
| 9.2                |

### Approach 2 : Using Lead/Lag Functions
We can use any function that we prefer, I will give you both examples as well.\
### Using Lead() :
```` sql
SELECT  cte.*,lead(plan_id,1) OVER(PARTITION BY customer_id ORDER BY START_DATE) as next_plan FROM CTE;
````
This will give the next plan_id for every row.\
Now we gotta sum only when current `plan_id = 0` and `next_plan_id = 4`\
Notice how we didn’t have to check if the before plan_id was 0 in the above approach. But in this approach we have to check that condition. Then we will pass this as subquery to get the desired result.
#### Final Query 2A
````sql
SELECT 
    ROUND(
        100.0 * SUM(CASE WHEN plan_id = 0 AND next_plan = 4 THEN 1 ELSE 0 END) / 
        COUNT(DISTINCT customer_id), 
        1
    ) AS perc_churned
FROM (
    SELECT 
        cte.*, 
        LEAD(plan_id, 1) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_plan
    FROM 
        CTE
) temp;
````
### Using Lag() :
- I won’t get into much detail as this is pretty similar to Lead(), but with few minor changes.
#### Final Query 2B
```` sql
SELECT 
    ROUND(
        100.0 * SUM(CASE WHEN plan_id = 4 AND prev_plan = 0 THEN 1 ELSE 0 END) / 
        COUNT(DISTINCT customer_id), 
        1
    ) AS perc_churned
FROM (
    SELECT 
        cte.*, 
        LAG(plan_id, 1) OVER (PARTITION BY customer_id ORDER BY start_date) AS prev_plan
    FROM 
        CTE
) temp;
````
---

**6. What is the number and percentage of customer plans after their initial free trial?**
### Approach 1 : Using Row Nmber
- We need to find something that happened right after initial free trial. So, my first inituition to approach this question is by using `row_numbers` because we can filter out our required columns by using `rn = 2`.
Next we need to find the count of each `plan_id` and its percentage. First lets focus on just the count.
- As we need it for each `plan_id`, we need to `GROUP BY plan_id,plan_name`
```` sql
SELECT 
    plan_id, 
    plan_name, 
    COUNT(*) AS count
FROM (
    SELECT 
        cte.*, 
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS rn
    FROM 
        CTE
) temp
WHERE rn = 2
GROUP BY 
    plan_id, 
    plan_name
ORDER BY 1;
````
- Next to calculate percentage,you can use the normal subquery method where in denominator you will do `SELECT count(distinct customer_id) FROM cte`
However I want you to know that instead sub query, we can do this :\
`sum(count(*)) over()` -> this will sum up all the values in the count column\
So combining all this we get our final query.

#### Final Query
```` sql
SELECT 
    plan_id, 
    plan_name, 
    COUNT(*) AS count,
    ROUND(
        100.0 * COUNT(*) / SUM(COUNT(*)) OVER(), 
        1
    ) AS perc
FROM (
    SELECT 
        cte.*, 
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) AS rn
    FROM 
        CTE
) temp
WHERE 
    rn = 2
GROUP BY 
    plan_id, 
    plan_name
ORDER BY 
    plan_id;
````
#### Output Table
 
| plan_id | plan_name     | count | perc |
| ------- | ------------- | ----- | ---- |
| 1       | basic monthly | 546   | 54.6 |
| 2       | pro monthly   | 325   | 32.5 |
| 3       | pro annual    | 37    | 3.7  |
| 4       | churn         | 92    | 9.2  |

- Just like previous questions we can use lead/lag functions to get the answer as well. I don’t think much detail is needed as you are already familiar with alternate approaches, hopefully you can solve this by yourself.

---

**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**
Before going to the solution I want you to try this problem for a couple more minutes, because  this is a bit tricky and I can assure you that when you get the correct solution yourself it feels pretty good.\
If you didn’t get the answer, no worries, we are here to learn after all.\
So we want the customer count and percentage breakdown for all 5 plan_names before the year 2021. So can we just put a where condition to eliminate all the plans that are after the end of 2020.\
But even after applying the condition we will have all the previous plans that the customer had used. But we only want to know the latest plan the customer was using as of before end of 2020.
- So we can use `row_number()` and `order by start_date DESC` to get the latest plan to be marked by `rn 1`.

#### Final Query
SELECT 
    cte.*, 
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date DESC) AS rn
FROM 
    CTE
WHERE 
    start_date < '2020-12-31';

#### Output Table
 
**8. How many customers have upgraded to an annual plan in 2020?**

#### Final Query

#### Output Table
 
**9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**

#### Final Query

#### Output Table
 
**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)**

#### Final Query

#### Output Table
 
**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**

#### Final Query

#### Output Table
 
