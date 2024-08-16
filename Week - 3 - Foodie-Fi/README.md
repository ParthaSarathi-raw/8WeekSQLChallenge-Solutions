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

**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**\
Before going to the solution I want you to try this problem for a couple more minutes, because  this is a bit tricky and I can assure you that when you get the correct solution yourself it feels pretty good.\
If you didn’t get the answer, no worries, we are here to learn after all.\
So we want the customer count and percentage breakdown for all 5 plan_names before the year 2021. So can we just put a where condition to eliminate all the plans that are after the end of 2020.\
But even after applying the condition we will have all the previous plans that the customer had used. But we only want to know the latest plan the customer was using as of before end of 2020.
- So we can use `row_number()` and `order by start_date DESC` to get the latest plan to be marked by `rn 1`.
```` sql
SELECT 
    cte.*, 
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date DESC) AS rn
FROM 
    CTE
WHERE 
    start_date < '2020-12-31';
````
Now we can filter out the latest plan by putting `rn = 1`.\
Now all we gotta do is count and find percentage of each plan_id just like previous question. So here is the query since you’re already familiar with this kind of concept.

#### Final Query
```` sql
SELECT 
    plan_id, 
    plan_name, 
    COUNT(*) AS count, 
    ROUND(100.0 * COUNT(*) / SUM(COUNT(*)) OVER(), 1) AS percentage
FROM (
    SELECT 
        cte.*, 
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date DESC) AS rn
    FROM 
        cte
    WHERE 
        start_date <= '2020-12-31'
) temp
WHERE 
    rn = 1
GROUP BY 
    plan_id, 
    plan_name;
````

#### Output Table
 
| plan_id | plan_name     | count | percentage |
| ------- | ------------- | ----- | ---------- |
| 0       | trial         | 19    | 1.9        |
| 1       | basic monthly | 224   | 22.4       |
| 2       | pro monthly   | 326   | 32.6       |
| 3       | pro annual    | 195   | 19.5       |
| 4       | churn         | 236   | 23.6       |

Now I want you to answer the same question using lead/lag functions. Can you do it?
Here is the solution using Lead() if you’re stuck trying to find alternate approach. Based on lead() hope you can build the solution using LAG() as well.

```` sql
SELECT 
    plan_id, 
    plan_name, 
    SUM(CASE 
            WHEN (next_date IS NULL OR EXTRACT(year FROM next_date) > 2020) 
            THEN 1 
            ELSE 0 
        END) AS count,
    ROUND(
        100.0 * SUM(CASE 
                        WHEN (next_date IS NULL OR EXTRACT(year FROM next_date) > 2020) 
                        THEN 1 
                        ELSE 0 
                    END) / 
        SUM(SUM(CASE 
                    WHEN (next_date IS NULL OR EXTRACT(year FROM next_date) > 2020) 
                    THEN 1 
                    ELSE 0 
                END)) OVER(), 
        1
    ) AS percentage
FROM (
    SELECT 
        cte.*, 
        LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_date
    FROM 
        cte
    WHERE 
        start_date <= '2020-12-31'
) temp
GROUP BY 
    plan_id, 
    plan_name
ORDER BY 
    plan_id;
````
- I know this is a bit complex, but you can understand it if you put enough time. You can skip this approach if you’re comfortable with row_number() method.

---


**8. How many customers have upgraded to an annual plan in 2020?**
This is a pretty easy question, we just need to filter out our results which have `start_date year in 2020` and `plan_id = 3` and we will apply the `count()` fn to get the count.
#### Final Query
```` sql
SELECT 
    COUNT(*) as customers_upgraded_to_annual_plan_in_2020
FROM 
    CTE
WHERE 
    plan_id = 3 
    AND EXTRACT(year FROM start_date) = 2020;
````
#### Output Table
 
| customers_upgraded_to_annual_plan_in_2020 |
| ----------------------------------------- |
| 195                                       |

---


**9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**
### Approach 1 : Using Lead/Lag Functions
Before we jump to lead/lag, I want you to observe this table first.
```` sql
SELECT * FROM CTE WHERE plan_id in (0,3);
````
- Each and every customer has to start their journey at foodie fi by taking a free trial first, so we have it for everyone. But for only those customers who have taken annual plans, they have 2 rows, one for `plan_id = 0` trial and one for `plan_id = 3` annual plan.
- We can find the `next_start_date` or `prev_start_date` using lead or lag functions.
- So now all we gotta do is find average of `next_start_date – start_date` or `start_date - prev_start_date`.
- Notice that when next_start_date/prev_start_date is blank/null, by default that calculation will not be counted.


#### Final Query
```` sql
SELECT 
    AVG(next_date - start_date) AS avg_days_taken_to_go_annual
FROM (
    SELECT 
        cte.*, 
        LEAD(start_date, 1) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_date
    FROM 
        CTE
    WHERE 
        plan_id IN (0, 3)
) temp;
````
#### Output Table
 
| avg_days_taken_to_go_annual |
| --------------------------- |
| 104.6201550387596899        |

### Apporach 2 : Using Joins
- Table 1 will have all the customers who have taken `plan_id = 0`
- Table 2 will have all the customers who have taken `plan_id = 3`
- When we join both tables, the output will be only those customers who have taken annual subscription.
- Next all we gotta do is find the difference between the dates and average them.
#### Final Query 2
```` sql
SELECT 
    AVG(next_date - start_date)
FROM (
    SELECT * 
    FROM CTE 
    WHERE plan_id = 0
) t1
JOIN (
    SELECT 
        customer_id, 
        start_date AS next_date 
    FROM 
        CTE 
    WHERE 
        plan_id = 3
) t2
ON t1.customer_id = t2.customer_id;
````

---

**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)**
### Approach 1 : Using Case Statements
Again just from the previous table  instead of calculating the average, we need to group them on a 30 day interval.\
The following query may look big, but overall it is just a bunch of case statements for each and every interval.\
**Note : This is just the brute force method. If you want to know how to solve this question in a generalized way directly go to approach 2**

#### Final Query
```` sql
SELECT *
FROM (
    SELECT
        CASE
            WHEN date_diff BETWEEN 1 AND 30 THEN '1-30 days'
            WHEN date_diff BETWEEN 31 AND 60 THEN '31-60 days'
            WHEN date_diff BETWEEN 61 AND 90 THEN '61-90 days'
            WHEN date_diff BETWEEN 91 AND 120 THEN '91-120 days'
            WHEN date_diff BETWEEN 121 AND 150 THEN '121-150 days'
            WHEN date_diff BETWEEN 151 AND 180 THEN '151-180 days'
            WHEN date_diff BETWEEN 181 AND 210 THEN '181-210 days'
            WHEN date_diff BETWEEN 211 AND 240 THEN '211-240 days'
            WHEN date_diff BETWEEN 241 AND 270 THEN '241-270 days'
            WHEN date_diff BETWEEN 271 AND 300 THEN '271-300 days'
            WHEN date_diff BETWEEN 301 AND 330 THEN '301-330 days'
            WHEN date_diff BETWEEN 331 AND 360 THEN '331-360 days'
        END AS days,
        COUNT(*)
    FROM (
        SELECT rn - start_date AS date_diff
        FROM (
            SELECT
                plan_id,
                customer_id,
                start_date,
                LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS rn
            FROM 
                cte
            WHERE 
                plan_id IN (0, 3)
        ) t
        WHERE rn - start_date IS NOT NULL
    ) temp
    GROUP BY days
) temp
ORDER BY 
    CASE
        WHEN days = '1-30 days' THEN 1
        WHEN days = '31-60 days' THEN 2
        WHEN days = '61-90 days' THEN 3
        WHEN days = '91-120 days' THEN 4
        WHEN days = '121-150 days' THEN 5
        WHEN days = '151-180 days' THEN 6
        WHEN days = '181-210 days' THEN 7
        WHEN days = '211-240 days' THEN 8
        WHEN days = '241-270 days' THEN 9
        WHEN days = '271-300 days' THEN 10
        WHEN days = '301-330 days' THEN 11
        WHEN days = '331-360 days' THEN 12
    END;
````
#### Output Table
 
| days         | count |
| ------------ | ----- |
| 1-30 days    | 49    |
| 31-60 days   | 24    |
| 61-90 days   | 34    |
| 91-120 days  | 35    |
| 121-150 days | 42    |
| 151-180 days | 36    |
| 181-210 days | 26    |
| 211-240 days | 4     |
| 241-270 days | 5     |
| 271-300 days | 1     |
| 301-330 days | 1     |
| 331-360 days | 1     |

### Approach 2 : Using Mathematics
- The reason we are learning this approach is because, what if there are more than these 12 intervals, what if we have data for 10 years, how will we implement that? We cannot write these case statements for each and every interval.
- Again this approach just plays with maths, so I will try my best to explain through words but I can assure you the maths is not to the level of calculus lol. You can easily understand it, it is just bunch of multiplications,additions and divisions.
````sql
SELECT 
    days, 
    COUNT(*)
FROM (
    SELECT 
        (date_diff - 1) / 30 AS order_value,
        (((date_diff - 1) / 30) * 30) + 1 || ' - ' || (((date_diff - 1) / 30) + 1) * 30 || ' days' AS days
    FROM (
        SELECT 
            cte.*, 
            start_date - LAG(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS date_diff
        FROM 
            cte
        WHERE 
            plan_id IN (0, 3)
    ) temp
) temp
WHERE 
    days IS NOT NULL
GROUP BY 
    days, 
    order_value
ORDER BY 
    order_value;
````
- This will produce the exact same output as before, but this is a general query that works for infinite number of intervals, much smaller and preferred method. Please leave a comment if you feel like you didn't understand what I was doing, I will update it with explanation.\
\
**Note : There are some in-built functions in sql that deal with bins, however I don't think they work in every sql dialect. You can explore those bins as well and solve this question using them, but I prefer mathematical approach because it works anywhere.**
---

**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**\
Again this problem can be solved in multiple approaches using lead/lag and joins. As you’re already familiar with this type of questions  I will directly jump to the solution using lead(). I hope you can write alternate queries in any approach you would like.
#### Final Query
```` sql
SELECT 
    COUNT(DISTINCT customer_id) as downgraded_from_pro_monthly_to_basic_monthly
FROM (
    SELECT 
        cte.*, 
        LEAD(plan_id, 1) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_plan_id
    FROM 
        CTE
) temp
WHERE 
    next_plan_id = 1 
    AND plan_id = 2;
````
#### Output Table
 

| downgraded_from_pro_monthly_to_basic_monthly |
| -------------------------------------------- |
| 0                                            |

- I didn’t even include the condition for it to be in 2020 and the count is already coming out to be zero. So by common sense we can say that in 2020 no one downgraded from pro monthly to normal monthly.
---

## Challenge Payment Solution

They gave us a sample output of what our table should look like :-
![image](https://github.com/user-attachments/assets/daae800e-c66a-4e1f-8cd1-29e4c44242c0)

Based on this table I can see that, the monthly subscriptions are being repeated every month. 
- Before we dive into the solution I want you to understand the generate_series function. Look at the example below.
```` sql
SELECT generate_series(
    '2020-01-01'::date,
    '2021-01-01'::date,
    '1 month'::interval
) AS first_of_month;
````
Generate_Series Output :
<br>
![image](https://github.com/user-attachments/assets/cb8ed405-8d49-4e16-bcbe-e412afeadef5)

- Next I want you to take a look at this table
```` sql
SELECT 
    CTE.*, 
    LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_date
FROM 
    CTE
WHERE 
    EXTRACT(year FROM start_date) = 2020 
    AND plan_id NOT IN (0);
````
Table Output :
![image](https://github.com/user-attachments/assets/c635a424-7de8-4a9b-a95b-9b763a928f68)
<br>
Here we are just only extracting the data that we only require for the questions.\
We don’t need to display free trails.\
Next we will create a payment_date column using the generate series function, but if next_date is null, then we want it generate until the end of year 2020.
```` sql
SELECT 
    customer_id, 
    plan_id, 
    plan_name, 
    GENERATE_SERIES(start_date, COALESCE(next_date - 1, '2020-12-31'::date), '1 month'::interval) AS payment_date, 
    price AS amount
FROM (
    SELECT 
        CTE.*, 
        LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_date
    FROM 
        CTE
    WHERE 
        EXTRACT(year FROM start_date) = 2020 
        AND plan_id != 0
) t;
````
![image](https://github.com/user-attachments/assets/7b4f1451-14ef-442b-97ff-4a3fbe450a48)
---
**NOTE : Did you see instead of next_date, I took next_date -1. This is because coalesce is inclusive in nature. So when a customer changes their plan exactly after 1 month, it will count it as twice and we don’t want that to happen.**\
Look at the following cases.\
Case 1 : Using next_date-1
![image](https://github.com/user-attachments/assets/f7593d69-72d6-4b74-900c-362b1a5879f6)
Case 2 : Using next_date 
![image](https://github.com/user-attachments/assets/f4297592-a702-4c36-85fb-733e2e783e2a)
- You see  how both pro monthly and pro annual are included in this which makes this incorrect. It is a minor thing but we gotta be careful about these edge cases. How did I generate these output tables,we will find out soon, I just showed these to give you an example of why we need to do next_date – 1.
Lets get back to the solution.
---
- Also did you notice that this is also generating monthly payments for annual payments which doesn’t make sense lol and it will generate it the same for churns.
- If we exclude plan_id 0,3,4 in the where clause, the series will generate till the end of the month for every monthly plan. We gotta know when to stop the generation of series. This is why we will include plan ids 3 and 4 in the beginning. And finally after calculation we can exclude them by passing this table as subquery.
```` sql
SELECT *
FROM (
    SELECT 
        customer_id, 
        plan_id, 
        plan_name, 
        GENERATE_SERIES(start_date, COALESCE(next_date - 1, '2020-12-31'::date), '1 month'::interval) AS payment_date, 
        price AS amount
    FROM (
        SELECT 
            CTE.*, 
            LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_date
        FROM 
            CTE
        WHERE 
            EXTRACT(year FROM start_date) = 2020 
            AND plan_id != 0
    ) t
) temp
WHERE 
    plan_id IN (1, 2);
````
- Next we have to also include plan_id 3 which only happens once a year. So we can combine it with using UNION ALL.
- Also there is no need to include plan_id 4 because that is not a payment and it will not be included in the payments table.
```` sql
SELECT * 
FROM (
    SELECT 
        customer_id, 
        plan_id, 
        plan_name, 
        GENERATE_SERIES(start_date, COALESCE(next_date - 1, '2020-12-31'::date), '1 month'::interval) AS payment_date, 
        price AS amount
    FROM (
        SELECT 
            CTE.*, 
            LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_date
        FROM 
            CTE
        WHERE 
            EXTRACT(year FROM start_date) = 2020 
            AND plan_id != 0
    ) t
    WHERE 
        plan_id IN (1, 2)

    UNION ALL

    SELECT 
        customer_id, 
        plan_id, 
        plan_name, 
        start_date AS payment_date, 
        price AS amount
    FROM 
        CTE
    WHERE 
        plan_id = 3
) temp
ORDER BY 
    customer_id, 
    payment_date;
````
![image](https://github.com/user-attachments/assets/7b30a878-60e5-48f0-9ec7-b8a28acc5e85)
Cool, now this is kind of looking like the final output table that we might need. 
However there is still one condition we need to implement.
- Upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- First lets create the above output table as CTE and perform operations on there.
- Here the condition would be if cur_plan_id = 2 or 3 and prev_plan_id = 1 then amount would be `(price – 9.90)` when the cur_plan was bought before the prev_plan 1 expired.
- Also we have to implement payment_order which can easily be done with the help of row_number(). Pls look at the following final Query.
#### Final Query
```` sql
,CTE2 AS (
    SELECT * 
    FROM (
        SELECT 
            customer_id,
            plan_id,
            plan_name,
            GENERATE_SERIES(start_date, COALESCE(next_date - 1, '2020-12-31'::date), '1 month'::interval) AS payment_date,
            price AS amount
        FROM (
            SELECT 
                CTE.*, 
                LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_date
            FROM 
                CTE
            WHERE 
                EXTRACT(year FROM start_date) = 2020 
                AND plan_id != 0
        ) t
        WHERE 
            plan_id IN (1, 2)

        UNION ALL

        SELECT 
            customer_id,
            plan_id,
            plan_name,
            start_date AS payment_date,
            price AS amount
        FROM 
            CTE
        WHERE 
            plan_id = 3
    ) temp
    ORDER BY 
        customer_id, 
        payment_date
)

SELECT 
    temp.*, 
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY payment_date) AS payment_order
FROM (
    SELECT 
        customer_id,
        plan_id,
        plan_name,
        payment_date,
        CASE 
            WHEN (
                LAG(plan_id, 1) OVER (PARTITION BY customer_id ORDER BY payment_date) = 1 
                AND plan_id IN (2, 3)
                AND LAG(payment_date, 1) OVER (PARTITION BY customer_id ORDER BY payment_date) + '1 month'::interval > payment_date
            ) THEN amount - 9.90 
            ELSE amount 
        END AS amount
    FROM 
        CTE2
) temp ;
````
#### Output Table (Limited to 10 Rows)


| customer_id | plan_id | plan_name     | payment_date             | amount | payment_order |
| ----------- | ------- | ------------- | ------------------------ | ------ | ------------- |
| 1           | 1       | basic monthly | 2020-08-08T00:00:00.000Z | 9.90   | 1             |
| 1           | 1       | basic monthly | 2020-09-08T00:00:00.000Z | 9.90   | 2             |
| 1           | 1       | basic monthly | 2020-10-08T00:00:00.000Z | 9.90   | 3             |
| 1           | 1       | basic monthly | 2020-11-08T00:00:00.000Z | 9.90   | 4             |
| 1           | 1       | basic monthly | 2020-12-08T00:00:00.000Z | 9.90   | 5             |
| 2           | 3       | pro annual    | 2020-09-27T00:00:00.000Z | 199.00 | 1             |
| 3           | 1       | basic monthly | 2020-01-20T00:00:00.000Z | 9.90   | 1             |
| 3           | 1       | basic monthly | 2020-02-20T00:00:00.000Z | 9.90   | 2             |
| 3           | 1       | basic monthly | 2020-03-20T00:00:00.000Z | 9.90   | 3             |
| 3           | 1       | basic monthly | 2020-04-20T00:00:00.000Z | 9.90   | 4             |

### Checking of Solution

- And If I am not mistaken , this should be the final payments table that we gotta generate.
- I know this is a lengthy process and some mistakes could have crept in, so lets double check with the sample output that has been provided.
- I am just going to put the where condition for customer_ids, which are available in sample table.
```` sql
,CTE2 AS (
    SELECT * 
    FROM (
        SELECT 
            customer_id,
            plan_id,
            plan_name,
            GENERATE_SERIES(start_date, COALESCE(next_date - 1, '2020-12-31'::date), '1 month'::interval) AS payment_date,
            price AS amount
        FROM (
            SELECT 
                CTE.*, 
                LEAD(start_date) OVER (PARTITION BY customer_id ORDER BY start_date) AS next_date
            FROM 
                CTE
            WHERE 
                EXTRACT(year FROM start_date) = 2020 
                AND plan_id != 0
        ) t
        WHERE 
            plan_id IN (1, 2)

        UNION ALL

        SELECT 
            customer_id,
            plan_id,
            plan_name,
            start_date AS payment_date,
            price AS amount
        FROM 
            CTE
        WHERE 
            plan_id = 3
    ) temp
    ORDER BY 
        customer_id, 
        payment_date
)

SELECT 
    temp.*, 
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY payment_date) AS payment_order
FROM (
    SELECT 
        customer_id,
        plan_id,
        plan_name,
        payment_date,
        CASE 
            WHEN (
                LAG(plan_id, 1) OVER (PARTITION BY customer_id ORDER BY payment_date) = 1 
                AND plan_id IN (2, 3)
                AND LAG(payment_date, 1) OVER (PARTITION BY customer_id ORDER BY payment_date) + '1 month'::interval > payment_date
            ) THEN amount - 9.90 
            ELSE amount 
        END AS amount
    FROM 
        CTE2
) temp 
WHERE 
    customer_id IN (1, 2, 13, 15, 16, 18, 19);
````
#### Output Sample Table generated by us

| customer_id | plan_id | plan_name     | payment_date             | amount | payment_order |
| ----------- | ------- | ------------- | ------------------------ | ------ | ------------- |
| 1           | 1       | basic monthly | 2020-08-08T00:00:00.000Z | 9.90   | 1             |
| 1           | 1       | basic monthly | 2020-09-08T00:00:00.000Z | 9.90   | 2             |
| 1           | 1       | basic monthly | 2020-10-08T00:00:00.000Z | 9.90   | 3             |
| 1           | 1       | basic monthly | 2020-11-08T00:00:00.000Z | 9.90   | 4             |
| 1           | 1       | basic monthly | 2020-12-08T00:00:00.000Z | 9.90   | 5             |
| 2           | 3       | pro annual    | 2020-09-27T00:00:00.000Z | 199.00 | 1             |
| 13          | 1       | basic monthly | 2020-12-22T00:00:00.000Z | 9.90   | 1             |
| 15          | 2       | pro monthly   | 2020-03-24T00:00:00.000Z | 19.90  | 1             |
| 15          | 2       | pro monthly   | 2020-04-24T00:00:00.000Z | 19.90  | 2             |
| 16          | 1       | basic monthly | 2020-06-07T00:00:00.000Z | 9.90   | 1             |
| 16          | 1       | basic monthly | 2020-07-07T00:00:00.000Z | 9.90   | 2             |
| 16          | 1       | basic monthly | 2020-08-07T00:00:00.000Z | 9.90   | 3             |
| 16          | 1       | basic monthly | 2020-09-07T00:00:00.000Z | 9.90   | 4             |
| 16          | 1       | basic monthly | 2020-10-07T00:00:00.000Z | 9.90   | 5             |
| 16          | 3       | pro annual    | 2020-10-21T00:00:00.000Z | 189.10 | 6             |
| 18          | 2       | pro monthly   | 2020-07-13T00:00:00.000Z | 19.90  | 1             |
| 18          | 2       | pro monthly   | 2020-08-13T00:00:00.000Z | 19.90  | 2             |
| 18          | 2       | pro monthly   | 2020-09-13T00:00:00.000Z | 19.90  | 3             |
| 18          | 2       | pro monthly   | 2020-10-13T00:00:00.000Z | 19.90  | 4             |
| 18          | 2       | pro monthly   | 2020-11-13T00:00:00.000Z | 19.90  | 5             |
| 18          | 2       | pro monthly   | 2020-12-13T00:00:00.000Z | 19.90  | 6             |
| 19          | 2       | pro monthly   | 2020-06-29T00:00:00.000Z | 19.90  | 1             |
| 19          | 2       | pro monthly   | 2020-07-29T00:00:00.000Z | 19.90  | 2             |
| 19          | 3       | pro annual    | 2020-08-29T00:00:00.000Z | 199.00 | 3             |


- We can see that this table that we generate is exactly similar to the sample table given by Danny. So fingers crossed I can confidently say that we have generated correct output.


---

### Please feel free to let me know if I have made any mistake or if you know a better approach to solve any question. If this helped you in anyway to improve your skills, just drop a message. It might not mean much to you, but it absolutely makes my day when I know that I’ve helped someone gain some knowledge.
### Anyways Happy Fiddling with the Data. See you in the next case study.









