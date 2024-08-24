# Case Study - 1 : Danny's Diner

All the data and questions that I've answered in this markdown can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-1/).

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.

**You can directly skip to code block to directly view the solution. However if you are stuck understanding something I suggest you to read my approach on how I tackled the problem**\
**Also please run the code blocks youself on dbfiddle as you can look at the output and follow through my explanations when you are confused**

## Entity Relationship Diagram
![image](https://github.com/user-attachments/assets/f1971f06-eba6-4976-9294-0ed5b94d5621)

Based on the ER diagram, we can clearly see that there are three tables. The sales table is directly connected to both the members table and the menu table. So, instead of performing the join operation in every query, can we just create a CTE (Common Table Expression) that has all the tables joined? The answer is yes, but the question is how are we going to join them, and what type of join should we use?

Well, the most straightforward approach is to perform an INNER JOIN between sales and members, and then another INNER JOIN between sales and menu. But is that correct?

You see, there might be some customers who have joined the loyalty program but haven’t ordered anything, or vice versa. At the same time, there might be some products on the menu that have never been ordered.

I know that for this case study, the dataset is small, and just by looking at the data, we can rule out these edge cases. But remember, Danny just gave us a sample of the dataset due to privacy issues. Hence, it is always better to write queries for the general case, taking these edge cases into consideration.
## Creation of CTE
We will be using this CTE table as the base table to answer all the questions.
- Make sure to include this code while solving any other question, because we are taking this as our primary table.
```` sql
WITH CTE as (
SELECT s.*,m.join_date,men.product_name,men.price
FROM dannys_diner.sales s 
FULL OUTER JOIN dannys_diner.members m ON s.customer_id = m.customer_id
FULL OUTER JOIN dannys_diner.menu men ON s.product_id = men.product_id
ORDER BY s.customer_id,order_date)
SELECT * FROM cte;
````
#### Base CTE Table

| customer_id | order_date               | product_id | join_date                | product_name | price |
| ----------- | ------------------------ | ---------- | ------------------------ | ------------ | ----- |
| A           | 2021-01-01T00:00:00.000Z | 2          | 2021-01-07T00:00:00.000Z | curry        | 15    |
| A           | 2021-01-01T00:00:00.000Z | 1          | 2021-01-07T00:00:00.000Z | sushi        | 10    |
| A           | 2021-01-07T00:00:00.000Z | 2          | 2021-01-07T00:00:00.000Z | curry        | 15    |
| A           | 2021-01-10T00:00:00.000Z | 3          | 2021-01-07T00:00:00.000Z | ramen        | 12    |
| A           | 2021-01-11T00:00:00.000Z | 3          | 2021-01-07T00:00:00.000Z | ramen        | 12    |
| A           | 2021-01-11T00:00:00.000Z | 3          | 2021-01-07T00:00:00.000Z | ramen        | 12    |
| B           | 2021-01-01T00:00:00.000Z | 2          | 2021-01-09T00:00:00.000Z | curry        | 15    |
| B           | 2021-01-02T00:00:00.000Z | 2          | 2021-01-09T00:00:00.000Z | curry        | 15    |
| B           | 2021-01-04T00:00:00.000Z | 1          | 2021-01-09T00:00:00.000Z | sushi        | 10    |
| B           | 2021-01-11T00:00:00.000Z | 1          | 2021-01-09T00:00:00.000Z | sushi        | 10    |
| B           | 2021-01-16T00:00:00.000Z | 3          | 2021-01-09T00:00:00.000Z | ramen        | 12    |
| B           | 2021-02-01T00:00:00.000Z | 3          | 2021-01-09T00:00:00.000Z | ramen        | 12    |
| C           | 2021-01-01T00:00:00.000Z | 3          |                          | ramen        | 12    |
| C           | 2021-01-01T00:00:00.000Z | 3          |                          | ramen        | 12    |
| C           | 2021-01-07T00:00:00.000Z | 3          |                          | ramen        | 12    |

---

## Solutions

**1) What is the total amount each customer spent at the restaurant?**

- For total amount `sum(price)`
- For each customer `GROUP BY customer_id `
- Combining everything we get the final query
#### Final Query 
```` sql
SELECT 
    customer_id,
    SUM(price) AS total_amount
FROM 
    cte
GROUP BY 
    customer_id
ORDER BY 
    customer_id;
````
#### Output Table

    
| customer_id | total_amount |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

---

**2) How many days has each customer visited the restaurant?**
#### Approach 1 : Using Distinct Keyword
- For no_of_days `count(distinct order_date)`. Notice that we use distinct keyword because for each order_date, there could be multiple orders. So we ideally want to count that order_date once only.
- For every customer `group by customer_id`
- Combining everything we get the final query
#### Final Query
```` sql
SELECT 
    customer_id,
    COUNT(DISTINCT order_date)
FROM 
    CTE
GROUP BY 
    customer_id;
````


#### Approach 2 : Without Distinct Keyword
- I wouldn't recommend this method because it is a bit lenghty and this question can be solved simply by using distinct keyword. But just know this logic because these logics can be applied anywhere.
- For each customer and on each specific `order_date`, there might be multiple orders, so first we want to combine all the orders on a specific date to a single row. So here we will use `group by customer_id` as well as `order_date`.
```` sql
SELECT customer_id,order_date FROM cte group by customer_id,order_date;
````
- Now that we got all customer_id and order_id combinations, now all we need to do is `group by customer_id` again by applying `count(order_date) or count(*)` by passing this above query as sub-query.
#### Final Query
```` sql
SELECT customer_id,
       COUNT(*)
FROM (
    SELECT customer_id,
           order_date 
    FROM cte 
    GROUP BY customer_id, 
             order_date
) temp
GROUP BY customer_id
ORDER BY customer_id;
````
#### Output Table


| customer_id | count |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

---
**3) What was the first item from the menu purchased by each customer?**
#### Approach 1 : Using Dense Rank
- Note that we are using *Dense Rank* because we are only given `order_date` which does not have any time . So we technically do not know what the customer first ordered when there are multiple products on a single day. Hence we will show all the products on that particular `order_date`.
- Now lets focus on *dense_rank()* logic itself.
We need their first order, so we will `order by order_date asc` order so the 1st rank assigned will be the one with least order_date.\
And we need this value for each customer, hence we will `partition by customer_id`, so that for each customer on their first order_date, it will be assigned a dense rank 1.
```` sql
SELECT customer_id,product_name,dense_rank() OVER(PARTITION BY customer_id ORDER BY order_date) FROM CTE;
````
- When we run the above query, it will give us the required table and from this we need only those rows which have dn as 1
- Hence we will pass this as subquery in FROM clause and put where condition to achieve the result
#### Final Query
```` sql
SELECT DISTINCT customer_id,
                product_name AS first_item_purchased
FROM (
    SELECT customer_id,
           product_name,
           DENSE_RANK() OVER (PARTITION BY customer_id ORDER BY order_date) AS dn
    FROM CTE
) temp
WHERE dn = 1;
````
**Note : Notice how we are using distinct Keyword here, because on the first day the customer could have ordered the same product multiple times and we don’t want that to be repeating multiple times in our output.**
#### Approach 2 : Using Joins
- For each customer, we need their minimum order date, so it can done with the help of aggregate function *min()*
```` sql
SELECT customer_id,min(order_date) as min_date FROM CTE GROUP BY customer_id;
````
- For each `customer_id`, this query will give their `minimum order_date`
- Now we need to perform join on CTE and this table using SUB QUERY on both these columns to get the desired result
#### Final Query
```` sql
SELECT cte.customer_id,
       product_name AS first_item_ordered
FROM cte
JOIN (
    SELECT customer_id,
           MIN(order_date) AS min_date
    FROM CTE
    GROUP BY customer_id
) temp ON cte.customer_id = temp.customer_id
      AND cte.order_date = temp.min_date
GROUP BY 1, 2;
````
#### Output Table


| customer_id | first_item_purchased |
| ----------- | -------------------- |
| A           | curry                |
| A           | sushi                |
| B           | curry                |
| C           | ramen                |

---

**4) What is the most purchased item on the menu and how many times was it purchased by all customers?**
- We want the most purchased item and no. of times it was purchased. 
- As we want the count of each item, we will apply group by on `product_name` and sort it by `count(product_name) or count(*)` desending and LIMIT it by 1 row only.(Ideally it should be done on product_id, but assuming all the product_names are different because having the same product_name for different products won’t make sense anyway)
- As we also want the no. of times it was purchased we use the count() fn in select statement as well.


#### Final Query
```` sql
SELECT product_name AS most_purchased_product,
       COUNT(*) AS no_of_times_purchased
FROM CTE
GROUP BY product_name
ORDER BY 2 DESC
LIMIT 1;
````
#### Output Table

| most_purchased_product | no_of_times_purchased |
| ---------------------- | --------------------- |
| ramen                  | 8                     |

**Note : What if there are multiple products which are the most purchased? In that case this approach would be wrong. When there are multiple products you have to print those rows which have count = max(count) or you could use dense rank as well. I hope you can implement this. Its just a bit lenghty and in our data, there is only single item that is being purchased the most. Ideally you should always write the query assuming that in future, more data could be added to the table. So I suggest you to show the ouput using max(no_of_times_purchased) or dense rank as well.**

---
**5) Which item was the most popular for each customer?**
- Here for each customer, we need the highest count of no. of times they purchased one product.
- Also a customer could have multiple products which are purchased the most times. Sounds like something a dense rank could solve.
- If it wasn’t the first intuition, its completely fine. You are here to learn and improving your skills in SQL.
- First lets break it down into simple steps. How about we first find the count of each product that each customer has purchased. Lets worry about the most_purchased later.
```` sql
SELECT customer_id,product_name,count(*) FROM CTE
GROUP BY 1,2
ORDER BY 1,3 DESC;
````
- Cool now we have each `customer,product_name` and the no. of times it has been ordered. Now we only need to extract the product_name which has highest count. But how can we achieve this?
- That’s right, now that we have broken the problem into 2 steps, at the 2nd step it is much clear that we need to use *dense_rank()*.
```` sql
SELECT customer_id,product_name,count(*),dense_rank() OVER (PARTITION BY customer_id ORDER BY count(*) DESC) as dn FROM CTE
GROUP BY 1,2
ORDER BY 1,3 DESC;
````
- I want you to note that there is no need for count(*) to be in select statement it is just for understanding. In the next query I will omit it because it is not necessary for the question mentioned, we don’t care how many times it has been purchased, we just want the highest one.
- And now all we have to do is extract all the rows from this table which has dn = 1, so as usual we will use sub query to achieve this.


#### Final Query
```` sql
SELECT customer_id,
       product_name
FROM (
    SELECT customer_id,
           product_name as most_popular_item,
           DENSE_RANK() OVER (
               PARTITION BY customer_id
               ORDER BY COUNT(*) DESC
           ) AS dn
    FROM CTE
    GROUP BY 1, 2
) temp
WHERE dn = 1
ORDER BY 1;
````
#### Output Table


| customer_id | most_popular_item |
| ----------- | ----------------- |
| A           | ramen             |
| B           | ramen             |
| B           | curry             |
| B           | sushi             |
| C           | ramen             |

---

**6) Which item was purchased first by the customer after they became a member?**
- Before we find the sol. Of the question, lets take an example case and see what the desired output is.
Lets say customer X has ordered more than 1 item everyday from day 1 to day 3. Lets say he bought the membership on day 2. Here the thing is the customer could have bought the products and then later in the evening he could have taken the member ship or he could have took the member ship before and after that he could have ordered. But we only have join_date and order_date, we don’t know the details of the time.
- So because of this reason I am choosing to give the orders which are placed after the joining date.
You could argue that it is most likely that the customer got the membership first and then ordered the products. That is right as well as there is no correct answer for the question based on the given data. However our main aim is to focus on the query rather than worrying about this. So lets get to SQL part, shall we?
- As discussed above, I want the first product after the joining date, so in the where clause we would include `order_date > join_date`. This will filter out all the orders after getting membership but we want the first product they ordered. So we will use dense rank which partitions over each `customer_id` and ordered by `order_date asc`. Then we pass this as subquery at which we will put `dn = 1` condition.
#### Final Query
```` sql
SELECT customer_id,
       product_name AS first_product_purchased_after_becoming_member
FROM (
    SELECT customer_id,
           product_name,
           DENSE_RANK() OVER (
               PARTITION BY customer_id
               ORDER BY order_date
           ) AS dn
    FROM cte
    WHERE order_date > join_date
) temp
WHERE dn = 1;
````

#### Output Table


| customer_id | first_product_purchased_after_becoming_member |
| ----------- | --------------------------------------------- |
| A           | ramen                                         |
| B           | sushi                                         |

**Note :
What happens to customers who have never took the membership?
For those customers the join_date would be NULL, so in the where condition of subquery (order_date>join_date) there is a NULL value, hence it will ignore those rows which is also what we want because we are only interested in customers who took membership.**

---

**7) Which item was purchased just before the customer became a member?**
Isn’t this question very similar to the previous question? Lets see the differences .
- This time we need to find last purchased item before the customer became a member. So in the where clause instead of `order_date>join_date` we would reverse the sign, `order_date<join_date`.
Does that solve the question or do we need to do something else as well?
- That’s right there is a change in the *dense_rank()* fn, instead of `order by order_date ASC`, we have to use `order_Date DESC` so we get the last date to have `dn=1`. 

#### Final Query
````sql
SELECT customer_id,
       product_name AS product_purchased_just_before_becoming_member
FROM (
    SELECT customer_id,
           product_name,
           DENSE_RANK() OVER (
               PARTITION BY customer_id
               ORDER BY order_date DESC
           ) AS dn
    FROM cte
    WHERE order_date < join_date
) temp
WHERE dn = 1;
````
#### Output Table
| customer_id | product_purchased_just_before_becoming_member |
| ----------- | --------------------------------------------- |
| A           | sushi                                         |
| A           | curry                                         |
| B           | sushi                                         |

---
**8) What is the total items and amount spent for each member before they became a member?**
Lets directly jump to sol shall we. 
- Before they became a member : `ORDER BY order_date<join_date`
- Total_items : `count(product_name)`
- Amount_spent : `sum(price)`
- For each member : `GROUP BY  customer_id`

#### Final Query
```` sql
SELECT customer_id,
       COUNT(product_name) AS total_items,
       SUM(price) AS total_amount_spent
FROM cte
WHERE order_date < join_date
GROUP BY customer_id
ORDER BY customer_id;
````
#### Output Table

| customer_id | total_items | total_amount_spent |
| ----------- | ----------- | ------------------ |
| A           | 2           | 25                 |
| B           | 3           | 40                 |

---
**9) If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
This may feel a bit tough if you are a beginner, but this is really one of the easiest questions in this case study.
- This can be simply solved through the help of case statement.
```` sql
Case when product_name = ‘sushi’ then 20*price else 10*price end as points
````
- This will give the points for each product that was purchased. Now all we gotta do is `sum()` this for each and every customer, so we apply `GROUP BY customer_id`.

#### Final Query

```` sql
SELECT customer_id,
       SUM(
           CASE 
               WHEN product_name = 'sushi' THEN price * 20
               ELSE price * 10
           END
       ) AS points
FROM cte
GROUP BY customer_id
ORDER BY customer_id;
````
#### Output Table

| customer_id | points |
| ----------- | ------ |
| A           | 860    |
| B           | 940    |
| C           | 360    |

---

**10) In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
Finally in this question we get some clarity , it mentions to give 2x points on all the items when customer joins the program including their join date. Does it indirectly imply that they got the membership before making any purchases later in the day? I guess it does, so based on this you can change the answer to a couple of previous questions discussed above, but I leave that up to you as this would be a minor change and won’t help you get better at SQL lol.
- Anyways coming to the question, we can solve this by using the same case statement , but the condition here would be a bit complex.
- We need to do 20 x price for all items when `order_date is between the join_date and within first week of join_date`, or do 20 x price for `items that are sushi`. If none of these cases are true then we simply do 10 x price as usual.
-  So after modifying the case statement , and adding few extra conditions such as `before ending of January`, only wanting to show `output of A and B`  etc, we get the query as:


#### Final Query
```` sql
SELECT customer_id,
       SUM(
           CASE 
               WHEN (
                   (order_date - join_date BETWEEN 0 AND 6)
                   OR (product_name = 'sushi')
               ) THEN 20 * price
               ELSE 10 * price
           END
       ) AS total_points
FROM cte
WHERE order_date < '2021-02-01'
  AND customer_id != 'C'
GROUP BY customer_id
ORDER BY customer_id;
````
#### Output Table

| customer_id | total_points |
| ----------- | ------------ |
| A           | 1370         |
| B           | 820          |

**Note:
The answer might change based on the WHERE conditions. In a few online resources that I’ve referred while doing this case study myself for the first time most of the solutions included where order_date>=join_date condition. I personally don’t think this makes sense because the points system is available before buying the membership as well and it is no where mentioned in the question to ignore the points that are before membership.**


---


### Please feel free to let me know if I have made any mistake or if you know a better approach to solve any question. If this helped you in anyway to improve your skills, just drop a message. It might not mean much to you, but it absolutely makes my day when I know that I’ve helped someone gain some knowledge.

### Anyways Happy Fiddling with the Data. See you in the next case study.



