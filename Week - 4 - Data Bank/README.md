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

#### Final Query

#### Output Table
 
**2.	What is the number of nodes per region?**

#### Final Query

#### Output Table
 
**3.	How many customers are allocated to each region?**

#### Final Query

#### Output Table
 
**4.	How many days on average are customers reallocated to a different node?**

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
 
