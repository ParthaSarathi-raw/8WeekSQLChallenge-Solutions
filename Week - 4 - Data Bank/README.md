# Case Study - 4 : Data Bank
All the data and questions that I've answered in this markdown can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-4/).

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/2GtQz4wZtuNNu7zXH5HtV4/3) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.

**You can directly skip to code block to directly view the solution. However if you are stuck understanding something I suggest you to read my approach on how I tackled the problem.**\
**Also please run the code blocks youself on dbfiddle as you can look at the output and follow through my explanations when you are confused.**

## Entity Relationship Diagram
<img width="532" alt="image" src="https://github.com/user-attachments/assets/3d07fdd4-73d6-464b-adf8-02e8280618cc">
<br>
You can create a base CTE table if you want, I personally don't think it is necessary for this case study.

## Customer Nodes Exploration Solutions
**1.	How many unique nodes are there on the Data Bank system?**
- Simple question, I hope this doesn't need explanation.
#### Final Query
```` sql
SELECT count(distinct node_id) from data_bank.customer_nodes;
````
#### Output Table
 
| count |
| ----- |
| 5     |

---

**2.	What is the number of nodes per region?**
- Very similar to last question, we just perform `join` to get the region name and `GROUP BY region_id,region_name` while doing `count(distinct node_id)` so we get the number of nodes per region.
#### Final Query
```` sql
SELECT 
    c.region_id,
    region_name,
    COUNT(DISTINCT node_id)
FROM 
    data_bank.customer_nodes c
INNER JOIN 
    data_bank.regions r 
    ON c.region_id = r.region_id
GROUP BY 
    c.region_id,
    region_name;
````
#### Output Table
 
| region_id | region_name | count |
| --------- | ----------- | ----- |
| 1         | Australia   | 5     |
| 2         | America     | 5     |
| 3         | Africa      | 5     |
| 4         | Asia        | 5     |
| 5         | Europe      | 5     |

---

**3.	How many customers are allocated to each region?**
- Exactly similar to the previous question, but instead of counting nodes, we are counting the customers
#### Final Query
```` sql
SELECT 
    c.region_id,
    region_name,
    COUNT(DISTINCT customer_id)
FROM 
    data_bank.customer_nodes c
INNER JOIN 
    data_bank.regions r 
    ON c.region_id = r.region_id
GROUP BY 
    1, 
    2
ORDER BY 
    1;
````
#### Output Table
 
| region_id | region_name | count |
| --------- | ----------- | ----- |
| 1         | Australia   | 110   |
| 2         | America     | 105   |
| 3         | Africa      | 102   |
| 4         | Asia        | 95    |
| 5         | Europe      | 88    |

---



**4.	How many days on average are customers reallocated to a different node?**

Before tackling this question I want you to think of one thing. For every row we have start_date and end_date. But what will be the end_date when the customer is currently in still in the node. He/she will have end_date = '9999-12-31' because they are still alloted to that node and no one knows when they will be realloted again. Hence we need to exclude these rows for our calculation. So whatever we do will be done on the table without these values.
```` sql
SELECT * FROM data_bank.customer_nodes 
WHERE end_date != '9999-12-31';
````
Before we dive into the correct solution I want to show you general wrong solutions for this problem which I've seen across internet.\
Look at the following sample table for a single customer for understanding.

```` sql
 CREATE TABLE nodes (node_id integer,start_date date,end_date date,date_diff integer);

INSERT INTO nodes (node_id,start_date,end_date,date_diff) VALUES

(1,'2020-01-02', '2020-01-03',1),(1,'2020-01-04','2020-01-10',6),(2,'2020-01-11','2020-01-17',6),(1,'2020-01-18','2020-01-26',8),(1,'2020-01-27','2020-01-31',4),(1,'2020-02-01','2020-02-10',9); 

SELECT * FROM nodes;
````

| node_id | start_date               | end_date                 | date_diff |
| ------- | ------------------------ | ------------------------ | --------- |
| 1       | 2020-01-02T00:00:00.000Z | 2020-01-03T00:00:00.000Z | 1         |
| 1       | 2020-01-04T00:00:00.000Z | 2020-01-10T00:00:00.000Z | 6         |
| 2       | 2020-01-11T00:00:00.000Z | 2020-01-17T00:00:00.000Z | 6         |
| 1       | 2020-01-18T00:00:00.000Z | 2020-01-26T00:00:00.000Z | 8         |
| 1       | 2020-01-27T00:00:00.000Z | 2020-01-31T00:00:00.000Z | 4         |
| 1       | 2020-02-01T00:00:00.000Z | 2020-02-10T00:00:00.000Z | 9         |

Now I will give you 3 questions. The solutions will be just down below but hidden as spoilers. First try to answer these questions for yourself.


<br>

**Q1. How many days on average does it take for the customer to get reallocated?**
<br>
<details>
  <summary>Mathematical Solution (Click me to see answer)</summary>
    (1+6+6+8+4+9)/6 = 5.66
</details>
<details>
  <summary>Query (Click me to see answer)</summary>
    SELECT avg(date_diff) FROM nodes;
</details>

**Q2. What is the average time spent on a single node overall by the user?**
<br>
<details>
  <summary>Mathematical Solution (Click me to see answer)</summary>
    ((1+6+8+4+9)+6)/2 = 17
</details>
<details>
  <summary>Query (Click me to see answer)</summary>
    SELECT avg(date_diff) FROM (SELECT sum(date_diff) as date_diff FROM nodes GROUP BY node_id) temp;
</details>

 **Q3. How many days on average did it take for the customer to get reallocated to a different node?**
<br>
Before seeing the answer I want you to see the difference between Q1 and Q3. Did you see **reallocated to a different node?** \
Will the answer be same as Q1? Lets find out.
<br>
<details>
  <summary>Mathematical Solution (Click me to see answer)</summary>
    ((1+6)+(6)+(8+4+9))/3 = 11.33 -> In this case first 2 rows should be treated as one because he is still on same node. Similarly last 3 rows should be treated as one beacause they all are same node.
</details>
<details>
  <summary>Query (Click me to see answer)</summary>
    Even though this might seem like a simple question, the query to solve this is a bit complex, we will build the solution to this question from ground up.
</details>

- I don't know what's up with everyone, but every online resource I looked up answered with solution 1 or solution 2 for question 3 which is just wrong in my opinion. 
- I am too poor to buy danny's official course, but from what I heard from others who had the official course, even the official solution provied by danny is also wrong. 
- Maybe I am stupid and I don't understand proper english, but when it is clearly mentioned **reallocated to a different node** I genuinly don't think either solution 1 or solution 2 is right for that question.

---

### Note : In the above answers that I've given for Q1,Q2 and Q3, there is a small flaw. Notice that date_diff is not same as no. of days stayed on that node. For no. of days stayed on that node, the correct answer would be date_diff + 1. But for the sake of simplicity, I'm ignoring the  " + 1 " in the calculation for now, but this can easily be fixed at the end by adding + 1 to date_diff. I just wanted you to know if you're wondering why am I complaining about wrong solutions available on the internet and I myself am giving wrong solutions lol.  

---


## Approach 1 : Creating the combined table and then finding the answer (Lenghty process to create combined table)

- By combined table what I mean is this:-

| node_id | start_date               | end_date                 | date_diff |
| ------- | ------------------------ | ------------------------ | --------- |
| 1       | 2020-01-02T00:00:00.000Z | 2020-01-10T00:00:00.000Z | 7         |
| 2       | 2020-01-11T00:00:00.000Z | 2020-01-17T00:00:00.000Z | 6         |
| 1       | 2020-01-18T00:00:00.000Z | 2020-02-10T00:00:00.000Z | 21        |


 
Here we can see all the rows which are adjacent and having the same node_id in the sample table I've given got combined to form a single row with start_date being min(start_dates),end_date being max(start_dates) and date_diff being sum(date_diffs) for all the adjacent rows.

**Note : I'd say just know the process of creating this combined table because, the logic that we use to combine is applicable anywhere. No need to implement this yourself, but just know how to it's done.**
### Building up the solution for Sample example table

- Look, when ever we have to deal with these type of problems where there would be multiple consecutive rows which needs to be treated as one, in general it is better to categorize them by different categories.
- First I will write a case statement to determine which type each row belongs to , namely Start, Middle, End and StandAlone which are pretty self explanatory.
- Start : When ever there is a chain of consecutive rows with same node, the first row would be labelled as start.
- End : When ever there is a chain of consecutive rows with same node, the last row would be labelled as end.
- Middle : When ever there is a chain of consecutive rows with same node, all the rows which are not either start or end will be labelled as middle.
- StandAlone : When a particular row's node doesn't match with either the previous or next node, then it will be labelled as standalone.

```` sql
WITH CTE as (SELECT 
    node_id,
    CASE 
        WHEN LAG(node_id) OVER (ORDER BY start_date) = node_id 
             AND LEAD(node_id) OVER (ORDER BY start_date) = node_id THEN 'MIDDLE'
        WHEN LEAD(node_id) OVER (ORDER BY start_date) = node_id THEN 'START'
        WHEN LAG(node_id) OVER (ORDER BY start_date) = node_id THEN 'END'
        ELSE 'STANDALONE' 
    END AS type,
    start_date,
    end_date,
    date_diff
FROM 
    nodes)
SELECT * FROM CTE;
````
| node_id | type       | start_date               | end_date                 | date_diff |
| ------- | ---------- | ------------------------ | ------------------------ | --------- |
| 1       | START      | 2020-01-02T00:00:00.000Z | 2020-01-03T00:00:00.000Z | 1         |
| 1       | END        | 2020-01-04T00:00:00.000Z | 2020-01-10T00:00:00.000Z | 6         |
| 2       | STANDALONE | 2020-01-11T00:00:00.000Z | 2020-01-17T00:00:00.000Z | 6         |
| 1       | START      | 2020-01-18T00:00:00.000Z | 2020-01-26T00:00:00.000Z | 8         |
| 1       | MIDDLE     | 2020-01-27T00:00:00.000Z | 2020-01-31T00:00:00.000Z | 4         |
| 1       | END        | 2020-02-01T00:00:00.000Z | 2020-02-10T00:00:00.000Z | 9         |

- We need to combine the types `(START,END)` as one and `(START,MIDDLE,END)` as one.
- I want you to notice one property that the dates (start_date and end_date) for type `MIDDLE` always fall between `start_date of START` and `end_date of END`. Hence my plan is to perform `GROUP BY` to add all the rows which fall under these days. Lets see how.
```` sql
,CTE2 as (
SELECT 
    cte.*, 
    CASE 
        WHEN type = 'START' THEN LEAD(end_date) OVER (ORDER BY start_date) 
    END AS final_end_date
FROM 
    CTE 
WHERE 
    type IN ('START', 'END'))
    SELECT * FROM CTE2;
````

| node_id | type  | start_date               | end_date                 | date_diff | final_end_date           |
| ------- | ----- | ------------------------ | ------------------------ | --------- | ------------------------ |
| 1       | START | 2020-01-02T00:00:00.000Z | 2020-01-03T00:00:00.000Z | 1         | 2020-01-10T00:00:00.000Z |
| 1       | END   | 2020-01-04T00:00:00.000Z | 2020-01-10T00:00:00.000Z | 6         |                          |
| 1       | START | 2020-01-18T00:00:00.000Z | 2020-01-26T00:00:00.000Z | 8         | 2020-02-10T00:00:00.000Z |
| 1       | END   | 2020-02-01T00:00:00.000Z | 2020-02-10T00:00:00.000Z | 9         |                          |

- I am only taking types START and END because the type MIDDLE will be in between these two types.
- Now for type START, there has to be another type END i.e they always come in pairs and when we order by start_date , every START will be followed by END.
- So when we take the start_date of START and end_date of END, it will give us with the range of dates that need to be grouped together and need to be treated as one.
- Next all we gotta do is combine our original cte table with this to get the types START,MIDDLE,END all together in a single table.

```` sql
SELECT CTE.*,final_end_date FROM CTE JOIN CTE2 
ON CTE.start_date BETWEEN CTE2.start_date and CTE2.final_end_date;
````
| node_id | type   | start_date               | end_date                 | date_diff | final_end_date           |
| ------- | ------ | ------------------------ | ------------------------ | --------- | ------------------------ |
| 1       | START  | 2020-01-02T00:00:00.000Z | 2020-01-03T00:00:00.000Z | 1         | 2020-01-10T00:00:00.000Z |
| 1       | END    | 2020-01-04T00:00:00.000Z | 2020-01-10T00:00:00.000Z | 6         | 2020-01-10T00:00:00.000Z |
| 1       | START  | 2020-01-18T00:00:00.000Z | 2020-01-26T00:00:00.000Z | 8         | 2020-02-10T00:00:00.000Z |
| 1       | MIDDLE | 2020-01-27T00:00:00.000Z | 2020-01-31T00:00:00.000Z | 4         | 2020-02-10T00:00:00.000Z |
| 1       | END    | 2020-02-01T00:00:00.000Z | 2020-02-10T00:00:00.000Z | 9         | 2020-02-10T00:00:00.000Z |

- Now we are getting somewhere. Do you notice that now we can simply GROUP BY final_end_date to combine all the date_diff so that they would be treated as one. Lets do this.
```` sql
SELECT cte.node_id,min(cte.start_date) as start_date,max(cte.end_date) as end_date,sum(cte.date_diff) as date_diff
FROM CTE JOIN CTE2 
ON CTE.start_date BETWEEN CTE2.start_date and CTE2.final_end_date
GROUP BY cte.node_id,final_end_date;
````
| node_id | start_date               | end_date                 | date_diff |
| ------- | ------------------------ | ------------------------ | --------- |
| 1       | 2020-01-02T00:00:00.000Z | 2020-01-10T00:00:00.000Z | 7         |
| 1       | 2020-01-18T00:00:00.000Z | 2020-02-10T00:00:00.000Z | 21        |

- Ayy lets go, now all we gotta do is combine this table with all other rows that are just standalone type and find average across everything to get the desired answer.
```` sql
SELECT * FROM (
SELECT cte.node_id,min(cte.start_date) as start_date,max(cte.end_date) as end_date,sum(cte.date_diff) as date_diff
FROM CTE JOIN CTE2 
ON CTE.start_date BETWEEN CTE2.start_date and CTE2.final_end_date
GROUP BY cte.node_id,final_end_date

UNION ALL

SELECT node_id,start_date,end_date,date_diff FROM CTE WHERE type = 'STANDALONE')temp
ORDER BY start_date;
````
| node_id | start_date               | end_date                 | date_diff |
| ------- | ------------------------ | ------------------------ | --------- |
| 1       | 2020-01-02T00:00:00.000Z | 2020-01-10T00:00:00.000Z | 7         |
| 2       | 2020-01-11T00:00:00.000Z | 2020-01-17T00:00:00.000Z | 6         |
| 1       | 2020-01-18T00:00:00.000Z | 2020-02-10T00:00:00.000Z | 21        |

- This is what our table looks like after combining all the required rows.
- Now finally instead of SELECT * FROM (.......), we have to do SELECT avg(date_diff) FROM (.....)
```` sql
SELECT avg(date_diff) FROM (
SELECT cte.node_id,min(cte.start_date) as start_date,max(cte.end_date) as end_date,sum(cte.date_diff) as date_diff
FROM CTE JOIN CTE2 
ON CTE.start_date BETWEEN CTE2.start_date and CTE2.final_end_date
GROUP BY cte.node_id,final_end_date

UNION ALL

SELECT node_id,start_date,end_date,date_diff FROM CTE WHERE type = 'STANDALONE')temp;
````
| avg                 |
| ------------------- |
| 11.3333333333333333 |

- And Voila, that checks out with out mathematical answer (1+6)+(6)+(8+4+9)/3 = 11.33.
- Cool, the reason I did this to small data set is to explain the logic, now all we gotta do is apply this logic to the original table.

## Approach 2 : Short-Cut Process to get the answer (Without calculation of combined table, so ShortCut)

I don't know if you have noticed this, but this question can be solved much simpler way without needing to create the combined table.

The mathematical answer would be = ((1+6)+6+(8+4+9))/3 
<br>
Can we say the answer is just = (sum of date_diff)/(no. of times nodes changed + 1)
<br>
Think about this for a moment. Makes sense right?
<br>
Now which approach do you prefer, the approach 1 or approach 2 lol. If you've skipped approach 1, I strongly suggest you to go and just have a look on how I solved the problem in approach 1, because it could be helpful in other problems as well.
<br>
Anyway coming to the shortcut method, we just need to know where the nodes are changing. So I will create a column called flag which has values 1 or 0.\
1 indicates that w.r.t previous node, the current node is changed. 0 Means the previous node and current node is same.

```` sql
WITH node_changes AS (
    SELECT 
        nodes.*,
        CASE 
            WHEN node_id != LAG(node_id) OVER (ORDER BY start_date) THEN 1 
            ELSE 0 
        END AS change_flag
    FROM 
        nodes
)
SELECT * FROM node_changes;
````
| node_id | start_date               | end_date                 | date_diff | change_flag |
| ------- | ------------------------ | ------------------------ | --------- | ----------- |
| 1       | 2020-01-02T00:00:00.000Z | 2020-01-03T00:00:00.000Z | 1         | 0           |
| 1       | 2020-01-04T00:00:00.000Z | 2020-01-10T00:00:00.000Z | 6         | 0           |
| 2       | 2020-01-11T00:00:00.000Z | 2020-01-17T00:00:00.000Z | 6         | 1           |
| 1       | 2020-01-18T00:00:00.000Z | 2020-01-26T00:00:00.000Z | 8         | 1           |
| 1       | 2020-01-27T00:00:00.000Z | 2020-01-31T00:00:00.000Z | 4         | 0           |
| 1       | 2020-02-01T00:00:00.000Z | 2020-02-10T00:00:00.000Z | 9         | 0           |

- Now just simply apply the mathematical formula and we get the answer.
```` sql
SELECT 1.0*sum(date_diff)/(sum(change_flag)+1) as avg_days_stayed_on_single_node_before_switching FROM node_changes;
````
| avg_days_stayed_on_single_node_before_switching |
| ----------------------------------------------- |
| 11.3333333333333333                             |

- See much easier right? Lets now see both approaches for Original Data shall we?



### Building up Solution For Original Data 
- I hope I made it crystal clear on the logic. I won't be explaining again because it is just the same thing. Please comment if you are confused about the following query.
- Look the query will be confusing because you are directly viewing it. But if you try to build it yourself just like I did for the sample table step by step, it makes sense.
- I strongly recommend you to build the solution yourself so you get a strong understanding of the concept.

#### Combined Table for the original data query
```` sql
WITH CTE AS (
    SELECT 
        CASE 
            WHEN LAG(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) = node_id 
                 AND LEAD(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) = node_id THEN 'MIDDLE' 
            WHEN LEAD(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) = node_id THEN 'START' 
            WHEN LAG(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) = node_id THEN 'END' 
            ELSE 'IGNORE' 
        END AS type, n.* 
    FROM data_bank.customer_nodes n 
    WHERE end_date != '9999-12-31' 
    ORDER BY customer_id, start_date
),
CTE2 AS (
    SELECT cte.*, 
           CASE WHEN type = 'START' THEN LEAD(end_date) OVER (ORDER BY customer_id, start_date) END AS final_end_date 
    FROM CTE 
    WHERE type IN ('START', 'END') 
    ORDER BY customer_id, start_date
),
COMBINED_TABLE AS (
SELECT * FROM (
    SELECT cte.customer_id, cte.region_id, cte.node_id, MIN(cte.start_date) AS start_date, final_end_date AS end_date 
    FROM CTE 
    JOIN CTE2 ON CTE.start_date BETWEEN CTE2.start_date AND CTE2.final_end_date AND cte.customer_id = cte2.customer_id 
    GROUP BY cte.customer_id, cte.region_id, cte.node_id, final_end_date
    UNION ALL
    SELECT customer_id, region_id, node_id, start_date, end_date 
    FROM CTE 
    WHERE type = 'IGNORE'
) temp ORDER BY customer_id,start_date)
SELECT * FROM COMBINED_TABLE;

````
#### Output Combined Table

- I'm just including data for 5 customers, in the final output there would be data of 500 customers.

| customer_id | region_id | node_id | start_date               | end_date                 |
| ----------- | --------- | ------- | ------------------------ | ------------------------ |
| 1           | 3         | 4       | 2020-01-02T00:00:00.000Z | 2020-01-14T00:00:00.000Z |
| 1           | 3         | 2       | 2020-01-15T00:00:00.000Z | 2020-01-16T00:00:00.000Z |
| 1           | 3         | 5       | 2020-01-17T00:00:00.000Z | 2020-01-28T00:00:00.000Z |
| 1           | 3         | 3       | 2020-01-29T00:00:00.000Z | 2020-02-18T00:00:00.000Z |
| 1           | 3         | 2       | 2020-02-19T00:00:00.000Z | 2020-03-16T00:00:00.000Z |
| 2           | 3         | 5       | 2020-01-03T00:00:00.000Z | 2020-01-17T00:00:00.000Z |
| 2           | 3         | 3       | 2020-01-18T00:00:00.000Z | 2020-02-21T00:00:00.000Z |
| 2           | 3         | 5       | 2020-02-22T00:00:00.000Z | 2020-03-07T00:00:00.000Z |
| 2           | 3         | 2       | 2020-03-08T00:00:00.000Z | 2020-03-12T00:00:00.000Z |
| 2           | 3         | 4       | 2020-03-13T00:00:00.000Z | 2020-03-13T00:00:00.000Z |
| 3           | 5         | 4       | 2020-01-27T00:00:00.000Z | 2020-02-18T00:00:00.000Z |
| 3           | 5         | 5       | 2020-02-19T00:00:00.000Z | 2020-03-06T00:00:00.000Z |
| 3           | 5         | 3       | 2020-03-07T00:00:00.000Z | 2020-03-24T00:00:00.000Z |
| 3           | 5         | 4       | 2020-03-25T00:00:00.000Z | 2020-04-08T00:00:00.000Z |
| 3           | 5         | 1       | 2020-04-09T00:00:00.000Z | 2020-04-09T00:00:00.000Z |
| 3           | 5         | 4       | 2020-04-10T00:00:00.000Z | 2020-04-24T00:00:00.000Z |
| 4           | 5         | 4       | 2020-01-07T00:00:00.000Z | 2020-02-13T00:00:00.000Z |
| 4           | 5         | 3       | 2020-02-14T00:00:00.000Z | 2020-03-02T00:00:00.000Z |
| 4           | 5         | 5       | 2020-03-03T00:00:00.000Z | 2020-03-12T00:00:00.000Z |
| 4           | 5         | 3       | 2020-03-13T00:00:00.000Z | 2020-04-01T00:00:00.000Z |
| 5           | 3         | 3       | 2020-01-15T00:00:00.000Z | 2020-01-23T00:00:00.000Z |
| 5           | 3         | 1       | 2020-01-24T00:00:00.000Z | 2020-02-05T00:00:00.000Z |
| 5           | 3         | 4       | 2020-02-06T00:00:00.000Z | 2020-02-15T00:00:00.000Z |
| 5           | 3         | 2       | 2020-02-16T00:00:00.000Z | 2020-02-29T00:00:00.000Z |
| 5           | 3         | 5       | 2020-03-01T00:00:00.000Z | 2020-03-05T00:00:00.000Z |


#### Final Query Using Approach 1 
- Now that we got the combined query all we gotta do is instead of " * " to retrieved the whole table, we do `avg(end_date-start_date+1)`

```` sql
SELECT avg(end_date - start_date + 1) FROM COMBINED_TABLE;
````
- Notice how I added `+ 1`. A minor change but can easily be looked over when solving fastly.

#### Output Table using Approach 1
 
| avg                 |
| ------------------- |
| 18.8285828984343637 |

#### Final Query Using Approach 2
```` sql
WITH CTE AS (
    SELECT n.*, 
           CASE WHEN node_id != LAG(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) THEN 1 ELSE 0 END AS flag 
    FROM data_bank.customer_nodes n 
    WHERE end_date != '9999-12-31' 
    ORDER BY customer_id, start_date
)
SELECT 1.0 * SUM(end_date - start_date + 1) / (SUM(flag) + (SELECT COUNT(DISTINCT customer_id) FROM CTE)) 
FROM CTE;
````
- Have you noticed what I did in the denominator? This is because if you remember we did +1 in the denominator for that single particular customer. Now there are 500 customers, so we need to do +500 , but I wrote the general query for N number of customers. Remember to write the general query rather than hard coding values into the queries.
#### Output Table using Approach 2
| avg                 |
| ------------------- |
| 18.8285828984343637 |

- Exactly everything, including the 17th decimal is also the same in both the approaches.

## Using Approach - 2's Logic to Created Combined Table (Not so lengthy)
- Again I feel like we've spent way too much time on this problem, but the reason I decided to create combined table through lengthy process is because it is easier to understand, but however with some neat math tricks like using cumulative sum, you are able to create combined table in a much shorter way.
```` sql
WITH CTE as (SELECT n.*,case when node_id != lag(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) then 1 else 0 end as node_switched FROM data_bank.customer_nodes n WHERE end_date != '9999-12-31')
,CTE2 as (SELECT cte.*,sum(node_switched) OVER(PARTITION BY customer_id ORDER BY start_date) as cum_sum FROM CTE)
SELECT customer_id,region_id,node_id,min(start_date) as start_date,max(end_date) as end_date FROM CTE2 GROUP BY customer_id,region_id,node_id,cum_sum ORDER BY 1,4;
````
- Just showing rows for first 5 customers.

| customer_id | region_id | node_id | start_date               | end_date                 |
| ----------- | --------- | ------- | ------------------------ | ------------------------ |
| 1           | 3         | 4       | 2020-01-02T00:00:00.000Z | 2020-01-14T00:00:00.000Z |
| 1           | 3         | 2       | 2020-01-15T00:00:00.000Z | 2020-01-16T00:00:00.000Z |
| 1           | 3         | 5       | 2020-01-17T00:00:00.000Z | 2020-01-28T00:00:00.000Z |
| 1           | 3         | 3       | 2020-01-29T00:00:00.000Z | 2020-02-18T00:00:00.000Z |
| 1           | 3         | 2       | 2020-02-19T00:00:00.000Z | 2020-03-16T00:00:00.000Z |
| 2           | 3         | 5       | 2020-01-03T00:00:00.000Z | 2020-01-17T00:00:00.000Z |
| 2           | 3         | 3       | 2020-01-18T00:00:00.000Z | 2020-02-21T00:00:00.000Z |
| 2           | 3         | 5       | 2020-02-22T00:00:00.000Z | 2020-03-07T00:00:00.000Z |
| 2           | 3         | 2       | 2020-03-08T00:00:00.000Z | 2020-03-12T00:00:00.000Z |
| 2           | 3         | 4       | 2020-03-13T00:00:00.000Z | 2020-03-13T00:00:00.000Z |
| 3           | 5         | 4       | 2020-01-27T00:00:00.000Z | 2020-02-18T00:00:00.000Z |
| 3           | 5         | 5       | 2020-02-19T00:00:00.000Z | 2020-03-06T00:00:00.000Z |
| 3           | 5         | 3       | 2020-03-07T00:00:00.000Z | 2020-03-24T00:00:00.000Z |
| 3           | 5         | 4       | 2020-03-25T00:00:00.000Z | 2020-04-08T00:00:00.000Z |
| 3           | 5         | 1       | 2020-04-09T00:00:00.000Z | 2020-04-09T00:00:00.000Z |
| 3           | 5         | 4       | 2020-04-10T00:00:00.000Z | 2020-04-24T00:00:00.000Z |
| 4           | 5         | 4       | 2020-01-07T00:00:00.000Z | 2020-02-13T00:00:00.000Z |
| 4           | 5         | 3       | 2020-02-14T00:00:00.000Z | 2020-03-02T00:00:00.000Z |
| 4           | 5         | 5       | 2020-03-03T00:00:00.000Z | 2020-03-12T00:00:00.000Z |
| 4           | 5         | 3       | 2020-03-13T00:00:00.000Z | 2020-04-01T00:00:00.000Z |
| 5           | 3         | 3       | 2020-01-15T00:00:00.000Z | 2020-01-23T00:00:00.000Z |
| 5           | 3         | 1       | 2020-01-24T00:00:00.000Z | 2020-02-05T00:00:00.000Z |
| 5           | 3         | 4       | 2020-02-06T00:00:00.000Z | 2020-02-15T00:00:00.000Z |
| 5           | 3         | 2       | 2020-02-16T00:00:00.000Z | 2020-02-29T00:00:00.000Z |
| 5           | 3         | 5       | 2020-03-01T00:00:00.000Z | 2020-03-05T00:00:00.000Z |

- I hope you can understand that when we do group by cum_sum , all the adjacent rows which have same node will get combined into one. This is the shortcut. If you don't understand, feel free to comment, I will try to explain it in detail.

#### Another Final Note
- What we did here is found the average of days across the whole dataset in a single aggregation.
- One could argue that instead of finding overall average, it would be better to find the average of days for each customer first and then find the average across all the customers.
- Again this is not that complicated, you just need to add an intermediate step in both apporaches where you would first find average for each customer by doing `GROUP BY customer_id`. Then on that resultant table you find the avg again.
- So what actually was asked in the question. avg(date_diff) or avg(date_diff's avg of each customer).
- Like I said for this question both these answers are fine because the almost give the same result. Why do they give the same result? Check in the next section.
#### When does the problem arise between avg(across) and avg(avg(each customer))
- These 2 values in the above problem give the same result because all the customers are **treated equally (same weightage)** by data_bank system while alloting nodes.
- Lets say there are customers who are the creators of the data_bank itself like danny and team. Because they are special, they don't get reallocated much often. So the no. of days these special customers stay in the nodes is significantly different from other regular users.
- In this case, the deviation between avg(across) and avg(avg(each customer)) increases. In the broad scope as there will be a lot of regular customers which are much higher than these special customers, it won't matter that much, but the difference between these two values increases when compared to the previous case where there are no special users.
- If you are really interested in this topic, I suggest you to go through **Simpson's Paradox (Weighted Average Dilemma)**, where you can gain even deeper understand of this problem and decide which type of avg to use depending on the specific need.

<br>

**5.	What is the median, 80th and 95th percentile for this same reallocation days metric for each region?**

It is mentioned **for this same reallocation days metric** in the question. So in this case we need to use combined table. Again if you've skipped approach 1, this is universe telling you to don't skip it lol.
Anyways coming to the question, we can calculate percentile in the general way. It can be done generally but it takes a lot of time. Hence by default there will be functions which can directly calculate percentiles. 
- We will be using `percentile_disc` function in postgre sql to solve this problem. The percentile functions could change depending on the dialect you are using.
- Also research on when to use percentile discrete and percentile continous if you don't know the difference between them.
#### Final Query
```` sql
WITH CTE as (SELECT n.*,case when node_id != lag(node_id) OVER (PARTITION BY customer_id ORDER BY start_date) then 1 else 0 end as node_switched FROM data_bank.customer_nodes n WHERE end_date != '9999-12-31')
,CTE2 as (SELECT cte.*,sum(node_switched) OVER(PARTITION BY customer_id ORDER BY start_date) as cum_sum FROM CTE)
,CTE3 as (SELECT customer_id,region_id,node_id,min(start_date) as start_date,max(end_date) as end_date FROM CTE2 GROUP BY customer_id,region_id,node_id,cum_sum ORDER BY 1,4)
,CTE4 as (SELECT CTE3.*,end_date-start_date + 1 as days_stayed FROM CTE3)
SELECT cte4.region_id,region_name,
       percentile_disc(0.5) within group ( order by days_stayed) as "50th percentile",
       percentile_disc(0.8) within group ( order by days_stayed) as "80th percentile",
       percentile_disc(0.95) within group ( order by days_stayed) as "95th percentile"
FROM CTE4 JOIN data_bank.regions  r ON  cte4.region_id = r.region_id
GROUP BY cte4.region_id,region_name;
````


#### Output Table

 | region_id | region_name | 50th percentile | 80th percentile | 95th percentile |
| --------- | ----------- | --------------- | --------------- | --------------- |
| 1         | Australia   | 18              | 27              | 42              |
| 2         | America     | 18              | 27              | 38              |
| 3         | Africa      | 18              | 28              | 40              |
| 4         | Asia        | 18              | 27              | 41              |
| 5         | Europe      | 19              | 28              | 39              |

Yes I know my percentile values, especially the 95th percentile column values are very different from other online solutions available. This is because no one bothered to actually combine the rows which have same node id.

- However if you feel like I've made a mistake some where, please feel free to let me know, so I will correct it.

---

## Customer Transactions Solutions

**1.	What is the unique count and total amount for each transaction type?**

Very Simple question so not much explanation needed.
- For each transaction type : `GROUP BY txn_type`
- Unique count for each txn_type : `Count(*) or count(txn_type)`
- Total Amount : `sum(txn_amount)`

#### Final Query

```` sql
SELECT 
    txn_type,
    COUNT(*) AS no_of_txns,
    SUM(txn_amount) AS total_amount
FROM 
    data_bank.customer_transactions
GROUP BY 
    txn_type;
````

#### Output Table

| txn_type   | no_of_txns | total_amount |
| ---------- | ---------- | ------------ |
| purchase   | 1617       | 806537       |
| deposit    | 2671       | 1359168      |
| withdrawal | 1580       | 793003       |

---

**2.	What is the average total historical deposit counts and amounts for all customers?**

They asked this average value for all customers. 
- So first we need to `Group By customer_id` first and then apply the averages.
- we need data for historical deposit : `WHERE txn_type = 'deposit'
- total deposit counts : `count(*) or count(txn_type)`
- total amount : `sum(txn_amount)`

Then we pass this table as subquery and find averages to get the output.

#### Final Query

```` sql
SELECT 
    ROUND(AVG(no_of_deposit_txns), 2) AS avg_historial_deposit_count_for_all_customers,
    ROUND(AVG(total_deposited), 2) AS avg_total_amount_deposited_for_all_customers
FROM (
    SELECT 
        customer_id,
        COUNT(*) AS no_of_deposit_txns,
        SUM(txn_amount) AS total_deposited
    FROM 
        data_bank.customer_transactions
    WHERE 
        txn_type = 'deposit'
    GROUP BY 
        customer_id
) temp;
````

#### Output Table

| avg_historial_deposit_count_for_all_customers | avg_total_amount_deposited_for_all_customers |
| --------------------------------------------- | -------------------------------------------- |
| 5.34                                          | 2718.34                                      |

---

**3.	For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**
- We need the data for each month, so from txn_date, we can get month : `extract(month from txn_date)`
- We also need counts for no. of deposits, purchases and withdrawls : `sum(case statements)`
- We need to find these counts for each customer in every month so : `GROUP BY customer_id,month`
- We will pass this table as subquery and get the count of those customers per every month whose `deposits > 1 and (purchases>0 or withdrawls>0)`
#### Final Query

```` sql
SELECT 
    month,
    SUM(CASE 
        WHEN (deposit_count > 1) AND (purchase_count > 0 OR withdrawl_count > 0) 
        THEN 1 
        ELSE 0 
    END) AS no_of_customers
FROM (
    SELECT 
        EXTRACT(MONTH FROM txn_date) AS month,
        customer_id,
        SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count,
        SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
        SUM(CASE WHEN txn_type = 'withdrawl' THEN 1 ELSE 0 END) AS withdrawl_count
    FROM 
        data_bank.customer_transactions
    GROUP BY 
        1, 2
) temp
GROUP BY 
    1
ORDER BY 
    1;
````

#### Output Table

| month | no_of_customers |
| ----- | --------------- |
| 1     | 128             |
| 2     | 135             |
| 3     | 146             |
| 4     | 55              |

---
 
**4.	What is the closing balance for each customer at the end of the month?**

First let us try to find the net_txn_amount for each month. 
- First we need to get month from txn_date : `extract(month from txn_date)`
- Then we need to find the net_txn_amount for each customer for each month  so : `GROUP BY month,customer_id`
- To find the net_txn_amount we can use case statements : `sum(case when txn_type = 'deposit' then txn_amount else  -txn_amount end)`
- We will pass this table as subquery and do running total across each month for each customer to get the closing amount for each month.
```` sql
SELECT customer_id,month,sum(net_value_per_month) OVER(PARTITION BY customer_id ORDER BY month) as closing_balance FROM (

SELECT extract(month from txn_date) as month,customer_id,
sum(case 
    when txn_type = 'deposit' then txn_amount
    else  -txn_amount
    end
    ) as net_value_per_month FROM data_bank.customer_transactions
    GROUP BY 1,2
    ORDER BY 2,1) temp
````
| customer_id | month | closing_balance |
| ----------- | ----- | --------------- |
| 1           | 1     | 312             |
| 1           | 3     | -640            |
| 2           | 1     | 549             |
| 2           | 3     | 610             |
| 3           | 1     | 144             |
| 3           | 2     | -821            |
| 3           | 3     | -1222           |
| 3           | 4     | -729            |
| 4           | 1     | 848             |
| 4           | 3     | 655             |
| 5           | 1     | 954             |
| 5           | 3     | -1923           |
| 5           | 4     | -2413           |

- Only showing values for first 5 customers.
- But few months in between are missing right? Since we only have data of 4 months, I would like to generate balances for 4 months.
- Look at the following final query which includes missing months as well, which is achieved using generate_series.

#### Final Query

```` sql
WITH CTE AS (
    SELECT 
        customer_id,
        month,
        SUM(net_value_per_month) OVER (PARTITION BY customer_id ORDER BY month) AS closing_balance
    FROM (
        SELECT 
            EXTRACT(MONTH FROM txn_date) AS month,
            customer_id,
            SUM(CASE 
                WHEN txn_type = 'deposit' THEN txn_amount
                ELSE -txn_amount
            END) AS net_value_per_month
        FROM 
            data_bank.customer_transactions
        GROUP BY 
            1, 2
        ORDER BY 
            2, 1
    ) temp
),
CTE2 AS (
    SELECT 
        customer_id,
        month,
        LEAD(month) OVER (PARTITION BY customer_id ORDER BY month) AS next_month,
        closing_balance
    FROM 
        CTE
)
SELECT 
    customer_id,
    GENERATE_SERIES(
        month::INTEGER,
        CASE 
            WHEN next_month IS NULL THEN 4
            ELSE next_month::INTEGER - 1
        END,
        1
    ) AS month,
    closing_balance
FROM 
    CTE2;
````

#### Output Table
- Only showing values for 5 customers only. In final table it has 500 customers x 4 rows per customers = 2000 rows

 | customer_id | month | closing_balance |
| ----------- | ----- | --------------- |
| 1           | 1     | 312             |
| 1           | 2     | 312             |
| 1           | 3     | -640            |
| 1           | 4     | -640            |
| 2           | 1     | 549             |
| 2           | 2     | 549             |
| 2           | 3     | 610             |
| 2           | 4     | 610             |
| 3           | 1     | 144             |
| 3           | 2     | -821            |
| 3           | 3     | -1222           |
| 3           | 4     | -729            |
| 4           | 1     | 848             |
| 4           | 2     | 848             |
| 4           | 3     | 655             |
| 4           | 4     | 655             |
| 5           | 1     | 954             |
| 5           | 2     | 954             |
| 5           | 3     | -1923           |
| 5           | 4     | -2413           |

---

**5.	What is the percentage of customers who increase their closing balance by more than 5%?**

- Again this is not tricky just a straight forward question. The only thing that is not mentioned is whether they are talking about successive months or for initial and final months.
- So I'm calculating this answer by taking month 1 and month 4 as reference as they are initial and final months.
- Also the closing_balances are negative, so make sure to use `abs(closing_balance)` whenever necessay while making calculations.

#### Final Query

```` sql
WITH CTE AS (
    SELECT 
        customer_id,
        month,
        SUM(net_value_per_month) OVER (PARTITION BY customer_id ORDER BY month) AS closing_balance
    FROM (
        SELECT 
            EXTRACT(MONTH FROM txn_date) AS month,
            customer_id,
            SUM(CASE 
                WHEN txn_type = 'deposit' THEN txn_amount
                ELSE -txn_amount
            END) AS net_value_per_month
        FROM 
            data_bank.customer_transactions
        GROUP BY 
            1, 2
        ORDER BY 
            2, 1
    ) temp
),
CTE2 AS (
    SELECT 
        customer_id,
        month,
        LEAD(month) OVER (PARTITION BY customer_id ORDER BY month) AS next_month,
        closing_balance
    FROM 
        CTE
),
closing_balance_table AS (
    SELECT 
        customer_id,
        GENERATE_SERIES(
            month::INTEGER,
            CASE 
                WHEN next_month IS NULL THEN 4
                ELSE next_month::INTEGER - 1
            END,
            1
        ) AS month,
        closing_balance
    FROM 
        CTE2
)
SELECT 
    ROUND(
        100.0 * SUM(
            CASE 
                WHEN final_closing_balance > (closing_balance + (0.05 * ABS(closing_balance))) 
                THEN 1 
                ELSE 0 
            END
        ) / NULLIF(SUM(CASE WHEN final_closing_balance IS NOT NULL THEN 1 ELSE 0 END), 0), 
        2
    ) AS perc_of_customers
FROM (
    SELECT 
        t.*,
        CASE 
            WHEN month = 1 
            THEN LEAD(closing_balance) OVER (PARTITION BY customer_id ORDER BY month) 
        END AS final_closing_balance
    FROM 
        closing_balance_table t
    WHERE 
        month IN (1, 4)
) temp;
````

#### Output Table

 | perc_of_customers |
| ----------------- |
| 33.20             |

---

## Data Allocation Challenge

- When I first heard this question, I had no idea what it actually meant. I actually thought of skipping this because it made no sense to me. Then I re-read the introduction part and I found this sentence "Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts".
-  I'm guessing the cloud storage data is directly proportional to the amount of balance in the account? I'm moving forward with this assumption.
-  Like for every 100$ in your balance, you will get 1GB of cloud storage.



### running customer balance column that includes the impact each transaction

- We need impact of each transaction, so instead of grouping by month, we won't do any grouping.

```` sql
SELECT 
    customer_id,
    txn_date,
    txn_type,
    txn_amount,
    SUM(
        CASE 
            WHEN txn_type = 'deposit' THEN txn_amount 
            ELSE -txn_amount 
        END
    ) OVER (
        PARTITION BY customer_id 
        ORDER BY txn_date
    ) AS current_balance
FROM 
    data_bank.customer_transactions;
````

- Only including data for 3 customers.

| customer_id | txn_date                 | txn_type   | txn_amount | current_balance |
| ----------- | ------------------------ | ---------- | ---------- | --------------- |
| 1           | 2020-01-02T00:00:00.000Z | deposit    | 312        | 312             |
| 1           | 2020-03-05T00:00:00.000Z | purchase   | 612        | -300            |
| 1           | 2020-03-17T00:00:00.000Z | deposit    | 324        | 24              |
| 1           | 2020-03-19T00:00:00.000Z | purchase   | 664        | -640            |
| 2           | 2020-01-03T00:00:00.000Z | deposit    | 549        | 549             |
| 2           | 2020-03-24T00:00:00.000Z | deposit    | 61         | 610             |
| 3           | 2020-01-27T00:00:00.000Z | deposit    | 144        | 144             |
| 3           | 2020-02-22T00:00:00.000Z | purchase   | 965        | -821            |
| 3           | 2020-03-05T00:00:00.000Z | withdrawal | 213        | -1034           |
| 3           | 2020-03-19T00:00:00.000Z | withdrawal | 188        | -1222           |
| 3           | 2020-04-12T00:00:00.000Z | deposit    | 493        | -729            |

- Based on this we need to do data allocation on a monthly basis.
- Look the only thing that I know is data allocation limit is directly proportional to the balance in the account. So I'm assuming the fact that whenever current_balance is negative no data allocation is required for those customers.

### Data Allocation Done Real Time

- In this case data allocation is done real time, so it doesn't mean that right after txn is done data is updated, but it is updated for each day.
- So we gotta use `generate_series` to get the current_balance of each customer on every day. However before their first_transaction we are assuming current_balance to be 0, so no need for allocation of data.
- After generating the data for every day for each customer, we will see how much data is required for each day.

```` sql
WITH CTE AS (
    SELECT 
        customer_id,
        generate_series(
            txn_date,
            CASE 
                WHEN lead(txn_date) OVER (PARTITION BY customer_id ORDER BY txn_date) IS NULL 
                    THEN '2020-04-30' 
                ELSE lead(txn_date) OVER (PARTITION BY customer_id ORDER BY txn_date) - 1 
            END,
            '1 day'
        ) AS cur_date,
        SUM(
            CASE 
                WHEN txn_type = 'deposit' THEN txn_amount 
                ELSE -txn_amount 
            END
        ) OVER (PARTITION BY customer_id ORDER BY txn_date) AS current_balance
    FROM data_bank.customer_transactions
)
SELECT 
    cur_date,
    SUM(CASE WHEN current_balance < 0 THEN 0 ELSE current_balance END) AS data_allocated
FROM CTE
GROUP BY cur_date
ORDER BY 1;
````
- Only showing first 5 days. In original ouput, the data would be of 4 months.

| cur_date                 | data_allocated |
| ------------------------ | -------------- |
| 2020-01-01T00:00:00.000Z | 12270          |
| 2020-01-02T00:00:00.000Z | 17830          |
| 2020-01-03T00:00:00.000Z | 25435          |
| 2020-01-04T00:00:00.000Z | 36350          |
| 2020-01-05T00:00:00.000Z | 42574          |

- Now that we have each days data_allocation required, and we are needed to do analysis on monthly.
- Here for monthly we can take the avg(data_allocated), that would be fair, but I want to take max(data_allocated) because our cloud servers must be capable of handling huge data which exceeds average limit.

```` sql
SELECT extract(month from cur_date) as month,max(data_allocated) as max_data_allocated FROM (SELECT 
    cur_date,
    SUM(CASE WHEN current_balance < 0 THEN 0 ELSE current_balance END) AS data_allocated
FROM CTE
GROUP BY cur_date
ORDER BY 1) temp
GROUP BY 1;
````
| month | max_data_allocated |
| ----- | ------------------ |
| 1     | 236157             |
| 2     | 274699             |
| 3     | 275616             |
| 4     | 266913             |

- The conversion rate could be like for every 100 $, 1GB of cloud data is allocated. So for month 1 January, 2361 GB of cloud data storage is allocated overall for all customers.

---

### customer balance at the end of each month

- We have already done this, so directly jumping to the query of data allocation

### Data Allocation done based on previous month's balance.

```` sql
WITH CTE AS (
    SELECT 
        customer_id,
        generate_series(
            extract(month from txn_date)::integer,
            CASE 
                WHEN lead(extract(month from txn_date)) OVER (PARTITION BY customer_id ORDER BY txn_date) IS NULL 
                    THEN 4 
                ELSE (lead(extract(month from txn_date)) OVER (PARTITION BY customer_id ORDER BY txn_date) - 1 )::integer
            END,
            1
        ) AS month,
        SUM(
            CASE 
                WHEN txn_type = 'deposit' THEN txn_amount 
                ELSE -txn_amount 
            END
        ) OVER (PARTITION BY customer_id ORDER BY txn_date) AS current_balance
    FROM data_bank.customer_transactions
)
SELECT month,sum(case when current_balance <0 then 0 else current_balance end) as total_data_allocation FROM CTE GROUP BY month ORDER BY month;

````

| month | total_data_allocation |
| ----- | --------------------- |
| 1     | 235595                |
| 2     | 261508                |
| 3     | 260971                |
| 4     | 264857                |

- Also I'd like to point out that in real-time allocation, for month 1 the data allocated is for that month only. Where as here in month 1, the calculated data will be assigned to the next month i.e month 2 and so on.

---

### minimum, average and maximum values of the running balance for each customer

- I'd like to point out that min and max values can be calculated directly, but avg cannot be calculated directly.
- Lets say I have 100$ at the beginning of January. And at the last day of January I deposited 400$ to make my balance equal to 500 $ total. Min is 100$ and max is 500$, but what is average? It is should be close to 100$, not (100+500)/2.
- So we need to do these calculations on another table which has each and every day generated including from the starting of year 2020.
```` sql
WITH CTE AS (
    SELECT 
        customer_id,
        generate_series(
            txn_date,
            CASE 
                WHEN lead(txn_date) OVER (PARTITION BY customer_id ORDER BY txn_date) IS NULL 
                    THEN '2020-04-30' 
                ELSE lead(txn_date) OVER (PARTITION BY customer_id ORDER BY txn_date) - 1 
            END,
            '1 day'
        ) AS cur_date,
        SUM(
            CASE 
                WHEN txn_type = 'deposit' THEN txn_amount 
                ELSE -txn_amount 
            END
        ) OVER (PARTITION BY customer_id ORDER BY txn_date) AS current_balance
    FROM data_bank.customer_transactions
)
SELECT * FROM (
SELECT customer_id,GENERATE_SERIES('2020-01-01',min(cur_date)-'1 day'::interval,'1 day') as cur_date,0 as current_balance FROM CTE GROUP BY customer_id
UNION ALL
SELECT * FROM CTE) temp ORDER BY customer_id,cur_date;
````
- This query gives each and every day's data for every customer without missing a day. Now on this we need to do monthly basis for each customer by doing min,max and avg.

```` sql
WITH CTE AS (
    SELECT 
        customer_id,
        generate_series(
            txn_date,
            CASE 
                WHEN lead(txn_date) OVER (PARTITION BY customer_id ORDER BY txn_date) IS NULL 
                    THEN '2020-04-30' 
                ELSE lead(txn_date) OVER (PARTITION BY customer_id ORDER BY txn_date) - 1 
            END,
            '1 day'
        ) AS cur_date,
        SUM(
            CASE 
                WHEN txn_type = 'deposit' THEN txn_amount 
                ELSE -txn_amount 
            END
        ) OVER (PARTITION BY customer_id ORDER BY txn_date) AS current_balance
    FROM data_bank.customer_transactions
),
CTE2 as(
SELECT * FROM (
SELECT customer_id,GENERATE_SERIES('2020-01-01',min(cur_date)-'1 day'::interval,'1 day') as cur_date,0 as current_balance FROM CTE GROUP BY customer_id
UNION ALL
SELECT * FROM CTE) temp ORDER BY customer_id,cur_date)
SELECT customer_id,extract(month from cur_date) as month,min(current_balance) as min_bal,max(current_balance) as max_bal, avg(current_balance) as avg_bal FROM CTE2 GROUP BY 1,2 ORDER BY 1,2;
````
- Showing data for 5 customers only.

 customer_id | month | min_bal | max_bal | avg_bal                |
| ----------- | ----- | ------- | ------- | ---------------------- |
| 1           | 1     | 0       | 312     | 301.9354838709677419   |
| 1           | 2     | 312     | 312     | 312.0000000000000000   |
| 1           | 3     | -640    | 312     | -342.7096774193548387  |
| 1           | 4     | -640    | -640    | -640.0000000000000000  |
| 2           | 1     | 0       | 549     | 513.5806451612903226   |
| 2           | 2     | 549     | 549     | 549.0000000000000000   |
| 2           | 3     | 549     | 610     | 564.7419354838709677   |
| 2           | 4     | 610     | 610     | 610.0000000000000000   |
| 3           | 1     | 0       | 144     | 23.2258064516129032    |
| 3           | 2     | -821    | 144     | -122.2068965517241379  |
| 3           | 3     | -1222   | -821    | -1085.3548387096774194 |
| 3           | 4     | -1222   | -729    | -909.7666666666666667  |
| 4           | 1     | 0       | 848     | 507.7419354838709677   |
| 4           | 2     | 848     | 848     | 848.0000000000000000   |
| 4           | 3     | 655     | 848     | 804.4193548387096774   |
| 4           | 4     | 655     | 655     | 655.0000000000000000   |
| 5           | 1     | 0       | 1780    | 689.4838709677419355   |
| 5           | 2     | 954     | 954     | 954.0000000000000000   |
| 5           | 3     | -1923   | 954     | 91.3870967741935484    |
| 5           | 4     | -2413   | -1923   | -2396.6666666666666667 |

### Data Allocation based on average balance in previous 30 days.

- I'd say the question contradicts itself.
- If we are doing data allocation based on avg balance of previous 30 days, then it means that it is being updated real time for every day.
- Then again they wanted us to do analysis on monthly basis.
- So I'm gonna solve this like I solved in the 1st case, i.e taking the maximum data allocated out of each day for every month.

```` sql
WITH CTE AS (
    SELECT 
        customer_id,
        generate_series(
            txn_date,
            CASE 
                WHEN lead(txn_date) OVER (PARTITION BY customer_id ORDER BY txn_date) IS NULL 
                    THEN '2020-04-30' 
                ELSE lead(txn_date) OVER (PARTITION BY customer_id ORDER BY txn_date) - 1 
            END,
            '1 day'
        ) AS cur_date,
        SUM(
            CASE 
                WHEN txn_type = 'deposit' THEN txn_amount 
                ELSE -txn_amount 
            END
        ) OVER (PARTITION BY customer_id ORDER BY txn_date) AS current_balance
    FROM data_bank.customer_transactions
),
CTE2 as(
SELECT * FROM (
SELECT customer_id,GENERATE_SERIES('2020-01-01',min(cur_date)-'1 day'::interval,'1 day') as cur_date,0 as current_balance FROM CTE GROUP BY customer_id
UNION ALL
SELECT * FROM CTE) temp ORDER BY customer_id,cur_date)
SELECT customer_id,cur_date,avg(current_balance) OVER(PARTITION BY customer_id ORDER BY cur_date ROWS BETWEEN 30 PRECEDING AND CURRENT ROW) as avg_bal FROM CTE2;
````
- The above query gives us avg_bal in last 30 days for everyday between january and april for each customer. Now all we gotta do is based on this value, find the data_allocated for each day and take out max for each month.
- Also don't forget, when avg_bal < 0 , data allocated will be 0.

```` sql
WITH CTE AS (
    SELECT 
        customer_id,
        generate_series(
            txn_date,
            CASE 
                WHEN lead(txn_date) OVER (PARTITION BY customer_id ORDER BY txn_date) IS NULL 
                    THEN '2020-04-30' 
                ELSE lead(txn_date) OVER (PARTITION BY customer_id ORDER BY txn_date) - 1 
            END,
            '1 day'
        ) AS cur_date,
        SUM(
            CASE 
                WHEN txn_type = 'deposit' THEN txn_amount 
                ELSE -txn_amount 
            END
        ) OVER (PARTITION BY customer_id ORDER BY txn_date) AS current_balance
    FROM data_bank.customer_transactions
),
CTE2 AS (
    SELECT * FROM (
        SELECT customer_id, GENERATE_SERIES('2020-01-01', min(cur_date) - '1 day'::interval, '1 day') AS cur_date, 0 AS current_balance 
        FROM CTE 
        GROUP BY customer_id
        UNION ALL
        SELECT * FROM CTE
    ) temp 
    ORDER BY customer_id, cur_date
),
CTE3 AS (
    SELECT cur_date, ROUND(SUM(CASE WHEN avg_bal < 0 THEN 0 ELSE avg_bal END)) AS total_data_allocated 
    FROM (
        SELECT customer_id, cur_date, AVG(current_balance) OVER (PARTITION BY customer_id ORDER BY cur_date ROWS BETWEEN 30 PRECEDING AND CURRENT ROW) AS avg_bal 
        FROM CTE2
    ) temp
    GROUP BY 1 
    ORDER BY 1
)
SELECT 
    EXTRACT(month FROM cur_date) AS month, 
    MAX(total_data_allocated) AS data_allocated_for_every_customer 
FROM CTE3 
GROUP BY 1;
````
| month | data_allocated_for_every_customer |
| ----- | --------------------------------- |
| 1     | 124579                            |
| 2     | 243353                            |
| 3     | 256572                            |
| 4     | 257031                            |

- The data_allocated_values could change based on our assumption for first txn.
- What I assumed is that before first txn, the cur_bal will be 0. So when a customer deposited 100$ at the end of 1st month, the avg will be close to 0.
- However what if the customer just signed up for data_bank on last day of January and deposited 100$, then his average should be 100$ only, not 0$.
- Keep this in mind, if we have more information on this, we can calculate even more accurate answer. But for this question I'm moving forward with the first assumption that every customer is already part of data_bank and has 0 as their cur_balance from 1st of January, 2020.

---

### Extra Challenge and Extension Request
- For the extra challenge, I'm not really familiar with all the annual compound calculations etc, but I think they are just implementation of mathematical calculations and nothing tricky sql is needed, so I'm comfortable skipping it.
- Same goes with extension request, it is very open ended question so I'm assuming you can do it yourself, or skip it since it doesn't need any sql .

