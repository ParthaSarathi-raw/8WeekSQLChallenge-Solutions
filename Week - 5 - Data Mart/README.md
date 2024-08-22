
## Case Study - 5 : Data Mart
All the data and questions that I've answered in this markdown can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-5/).

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/8) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.

**You can directly skip to code block to directly view the solution. However if you are stuck understanding something I suggest you to read my approach on how I tackled the problem**\
**Also please run the code blocks youself on dbfiddle as you can look at the output and follow through my explanations when you are confused**

## Entity Relationship Diagram

<img width="248" alt="image" src="https://github.com/user-attachments/assets/ba2308e4-f7bd-4e06-8ee0-13da746b041f">

Well there is only a single table, but there is a lot of data cleansing to be done. Let's get to it shall we?

## Data Cleansing Steps

There is a lot of data cleansing required and the requirement is to do this in a single query. So I will explain how to cleanse for each column and we will combine all these to clean everything in a single query.

- convert the week_date to date format : `to_date(week_date,'dd/mm/yy')`
- add week number as 2nd column : `extract(week from to_date(week_date,'dd/mm/yy'))`
- add month number as 3rd column : `extract(month from to_date(week_date,'dd/mm/yy'))`
- add calender year as 4th column : `extract(year from to_date(week_date,'dd/mm/yy'))`
- Add a new column called age_band after the original segment column using mapping mentioned in question : `case statement`
- Add a new demographic column using the mapping mentioned in the question : `case statement`
- Ensure all null string values with an "unknown" in segment,age_band and demographic : `case statement`
- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places : `round(1.0*sales/transactions,2)`

#### Final Query

```` sql
WITH CTE as (SELECT to_date(week_date,'dd/mm/yy') as week_date,
	extract(week from to_date(week_date,'dd/mm/yy')) as week_number,
    extract(month from to_date(week_date,'dd/mm/yy')) as month_number,
    extract(year from to_date(week_date,'dd/mm/yy')) as calender_year,
    region,platform,case when segment != 'null' then segment else 'unknown' end as segment,
    case when right(segment,1) = '1' then 'Young Adults' when right(segment,1) = '2' then 'Middle Aged' when right(segment,1) in ('3','4') then 'Retirees' else 'unknown' end as age_band,
    case when left(segment,1) = 'C' then 'Couples' when left(segment,1) = 'F' then 'Families' else 'unknown' end as demographic,
    transactions,sales,
    round(1.0*sales/transactions,2) as avg_transaction
FROM data_mart.weekly_sales)
SELECT * FROM CTE
;
````
#### Final Output

| week_date                | week_number | month_number | calender_year | region        | platform | segment | age_band     | demographic | transactions | sales    | avg_transaction |
| ------------------------ | ----------- | ------------ | ------------- | ------------- | -------- | ------- | ------------ | ----------- | ------------ | -------- | --------------- |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | ASIA          | Retail   | C3      | Retirees     | Couples     | 120631       | 3656163  | 30.31           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | ASIA          | Retail   | F1      | Young Adults | Families    | 31574        | 996575   | 31.56           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | USA           | Retail   | unknown | unknown      | unknown     | 529151       | 16509610 | 31.20           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | EUROPE        | Retail   | C1      | Young Adults | Couples     | 4517         | 141942   | 31.42           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | AFRICA        | Retail   | C2      | Middle Aged  | Couples     | 58046        | 1758388  | 30.29           |

- From now we will use this table as CTE and perform calculations on this base table only.

---

## Data Exploration


**1.	What day of the week is used for each week_date value?**

- Simply use `to_char` function to get day name from date.

#### Final Query
```` sql
SELECT distinct(to_char(week_date,'day')) as day_of_week FROM CTE;
````
#### Final Output
 
| day_of_week |
| ----------- |
| monday      |

---

**2.	What range of week numbers are missing from the dataset?**

- We will use `generate_series` to get week numbers 1 to 52 because in a year max weeks are 52 only.
- On that we will filter the numbers which are not present in our data.

#### Final Query

````sql
,cte2 as(SELECT GENERATE_SERIES(1,52) as week_numbers_missing) 
SELECT week_numbers_missing FROM cte2 
WHERE week_numbers_missing 
NOT IN 
(SELECT distinct week_number FROM cte);
````

#### Final Output

-Only showing 10 rows output. Original table had 28 rows/week numbers which are missing

| week_numbers_missing |
| -------------------- |
| 1                    |
| 2                    |
| 3                    |
| 4                    |
| 5                    |
| 6                    |
| 7                    |
| 8                    |
| 9                    |
| 10                   |
| 11                   |
| 12                   |

---
 
**3.	How many total transactions were there for each year in the dataset?**

- For each year : `GROUP BY caldender_year`
- Total transactions : `sum(transactions)`

#### Final Query

```` sql
SELECT calender_year,sum(transactions) as total_transactions 
FROM CTE 
GROUP BY calender_year
ORDER BY 1;
````

#### Final Output

| calender_year | total_transactions |
| ------------- | ------------------ |
| 2018          | 346406460          |
| 2019          | 365639285          |
| 2020          | 375813651          |

---

**4.	What is the total sales for each region for each month?**

- Total Sales : `sum(sales)`
- For each regoin for each month : `GROUP BY region,month`

#### Final Query

```` sql
SELECT region,month_number,sum(sales) as total_sales
FROM CTE 
GROUP BY 1,2
ORDER BY 1,2;
````

#### Final Output
 - Only showing output for Africa Region, original table has data for all regions

| region        | month_number | total_sales |
| ------------- | ------------ | ----------- |
| AFRICA        | 3            | 567767480   |
| AFRICA        | 4            | 1911783504  |
| AFRICA        | 5            | 1647244738  |
| AFRICA        | 6            | 1767559760  |
| AFRICA        | 7            | 1960219710  |
| AFRICA        | 8            | 1809596890  |
| AFRICA        | 9            | 276320987   |

---

**5.	What is the total count of transactions for each platform**

#### Final Query

#### Final Output
 
**6.	What is the percentage of sales for Retail vs Shopify for each month?**

#### Final Query

#### Final Output
 
**7.	What is the percentage of sales by demographic for each year in the dataset?**

#### Final Query

#### Final Output
 
**8.	Which age_band and demographic values contribute the most to Retail sales?**

#### Final Query

#### Final Output
 
**9.	Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?**

#### Final Query

#### Final Output
 

