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
 
