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

Now I will give you 3 questions. The solutions will be just down below but hidden as spoilers. First try to answer these questions for yourself.\


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

### Building up the solution for Sample example table

- Look, when ever we have to deal with these type of problems where there would be multiple consecutive rows which needs to be treated as one, in general it is better to categorize them by different categories.
- First I will write a case statement to determine which type each row belongs to , namely Start, Middle, End and StandAlone which are pretty self explanatory.
- Start : When ever there is a chain of consecutive rows with same node, the first row would be labelled as start.
- End : When ever there is a chain of consecutive rows with same node, the last row would be labelled as end.
- Middle : When ever there is a chain of consecutive rows with same node, all the rows which are not either start or end will be labelled as middle.
- StandAlone : When a particular row's node doesn't match with either the previous or next node, then it will be labelled as standalone.

#### Final Query

#### Output Table
 
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
 
