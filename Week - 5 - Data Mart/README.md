
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
