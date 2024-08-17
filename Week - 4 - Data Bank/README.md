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
    (1+6+8+4+9)+6/2 = 17
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
    (1+6)+(6)+(8+4+9)/3 = 11.33 -> In this case first 2 rows should be treated as one because he is still on same node. Similarly last 3 rows should be treated as one beacause they all are same node.
</details>
<details>
  <summary>Query (Click me to see answer)</summary>
    Even though this might seem like a simple question, the query to solve this is a bit complex, we will build the solution to this question from ground up.
</details>

- I don't know what's up with everyone, but every online resource I looked up answered with solution 1 or solution 2 for question 3 which is just wrong in my opinion. 
- I am too poor to buy danny's official course, but from what I heard from others who had the official course, even the official solution provied by danny is also wrong. 
- Maybe I am stupid and I don't understand proper english, but when it is clearly mentioned **reallocated to a different node** I genuinly don't think either solution 1 or solution 2 is right for that question.

---

### Note : In the above answers that I've given for Q1,Q2 and Q3, there is a small flaw. Notice that date_diff is not same as no. of days stayed on that node. For no. of days stayed on that node, the correct answer would be date_diff + 1. But for keeping it simplcity, I'm ignoring the  " + 1 " in the calculation for now, but this can easily be fixed at the end by adding + 1 to date_diff. I just wanted you to know if you're wondering why am I complaining about wrong solutions available on the internet and I myself am giving wrong solutions lol.  

---


## Approach 1 : Creating the combined table and then finding the answer (Lenghty process to create combined table)

- By combined table what I mean is this:-

| node_id | start_date               | end_date                 | date_diff |
| ------- | ------------------------ | ------------------------ | --------- |
| 1       | 2020-01-02T00:00:00.000Z | 2020-01-10T00:00:00.000Z | 7         |
| 2       | 2020-01-11T00:00:00.000Z | 2020-01-17T00:00:00.000Z | 6         |
| 1       | 2020-01-18T00:00:00.000Z | 2020-02-10T00:00:00.000Z | 21        |


 
Here we can see all the rows which are adjacent and having the same node_id in the sample table I've given got combined to form a single row with start_date being min(start_dates),end_date being max(start_dates) and date_diff being sum(date_diffs) for all the adjacent rows.
**Note : I'd say just know the process of creating this combined table because, the logic that we use to combine is applicable anywhere.**
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
- If you are really interested in this topic, I suggest you to go through **Simpson's Paradox (Weighted Average Dilemma)**, where you can gain even deeper understand of this problem and decide which type of avg depending on the specific need.
**5.	What is the median, 80th and 95th percentile for this same reallocation days metric for each region?**

#### Final Query

#### Output Table
 

## Customer Transactions Solutions

**1.	What is the unique count and total amount for each transaction type?**

#### Final Query

#### Output Table
 
**2.	What is the average total historical deposit counts and amounts for all customers?**

#### Final Query

#### Output Table
 
**3.	For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**

#### Final Query

#### Output Table
 
**4.	What is the closing balance for each customer at the end of the month?**

#### Final Query

#### Output Table
 
**5.	What is the percentage of customers who increase their closing balance by more than 5%?**

#### Final Query

#### Output Table
 
