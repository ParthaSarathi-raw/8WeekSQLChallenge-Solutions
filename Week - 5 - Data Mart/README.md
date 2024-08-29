
## Case Study - 5 : Data Mart
All the data and questions that I've answered in this markdown can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-5/).

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/8) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.

**You can directly skip to code block to directly view the solution. However if you are stuck understanding something I suggest you to read my approach on how I tackled the problem**\
**Also please run the code blocks youself on dbfiddle as you can look at the output and follow through my explanations when you are confused**

## Index

- [Entity Relationship Diagram](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#entity-relationship-diagram)
- [Data Cleansing Steps](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#data-cleansing-steps)
- [Data Exploration](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#data-exploration) \
[1](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#final-query-1)
[2](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#final-query-2)
[3](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#final-query-3)
[4](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#final-query-4)
[5](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#final-query-5)
[6](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#final-query-6)
[7](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#final-query-7)
[8](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#final-query-8)
[9](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#final-query-9)
- [Before and After Analysis](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#before-and-after-analysis) \
[1](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#final-query-10)
[2](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#final-query-11)
[3](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#final-query-1-1)
- [Bonus Question](https://github.com/ParthaSarathi-raw/8WeekSQLChallenge-Solutions/tree/main/Week%20-%205%20-%20Data%20Mart#bonus-question) 

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
    region,platform,customer_type,case when segment != 'null' then segment else 'unknown' end as segment,
    case when right(segment,1) = '1' then 'Young Adults' when right(segment,1) = '2' then 'Middle Aged' when right(segment,1) in ('3','4') then 'Retirees' else 'unknown' end as age_band,
    case when left(segment,1) = 'C' then 'Couples' when left(segment,1) = 'F' then 'Families' else 'unknown' end as demographic,
    transactions,sales,
    round(1.0*sales/transactions,2) as avg_transaction
FROM data_mart.weekly_sales)
SELECT * FROM CTE
;
````
#### Final Output

| week_date                | week_number | month_number | calender_year | region        | platform | customer_type | segment | age_band     | demographic | transactions | sales    | avg_transaction |
| ------------------------ | ----------- | ------------ | ------------- | ------------- | -------- | ------------- | ------- | ------------ | ----------- | ------------ | -------- | --------------- |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | ASIA          | Retail   | New           | C3      | Retirees     | Couples     | 120631       | 3656163  | 30.31           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | ASIA          | Retail   | New           | F1      | Young Adults | Families    | 31574        | 996575   | 31.56           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | USA           | Retail   | Guest         | unknown | unknown      | unknown     | 529151       | 16509610 | 31.20           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | EUROPE        | Retail   | New           | C1      | Young Adults | Couples     | 4517         | 141942   | 31.42           |
| 2020-08-31T00:00:00.000Z | 36          | 8            | 2020          | AFRICA        | Retail   | New           | C2      | Middle Aged  | Couples     | 58046        | 1758388  | 30.29           |


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

- total count of transactions : `sum(transactions)`
- for each platform : `GROUP BY platform`

#### Final Query
```` sql
SELECT platform,sum(transactions) as total_transactions FROM CTE 
GROUP BY platform;
````
#### Final Output

 | platform | total_transactions |
| -------- | ------------------ |
| Shopify  | 5925169            |
| Retail   | 1081934227         |

---

**6.	What is the percentage of sales for Retail vs Shopify for each month?**

- For each month : `GROUP BY month_number`
- For the calculation of percentages, we first use `case(statement)` to find the sales for retail as well as shopify for each month. Then from that we can easily calculate percentages.

#### Final Query
```` sql
SELECT 
    month_number,
    ROUND(100.0 * shopify_sales / (shopify_sales + retail_sales), 2) AS shopify_percentage,
    ROUND(100.0 * retail_sales / (shopify_sales + retail_sales), 2) AS retail_percentage
FROM (
    SELECT 
        month_number,
        SUM(CASE WHEN platform = 'Shopify' THEN sales END) AS shopify_sales,
        SUM(CASE WHEN platform = 'Retail' THEN sales END) AS retail_sales
    FROM CTE 
    GROUP BY month_number
    ORDER BY month_number
) temp;
````

#### Final Output


| month_number | shopify_percentage | retail_percentage |
| ------------ | ------------------ | ----------------- |
| 3            | 2.46               | 97.54             |
| 4            | 2.41               | 97.59             |
| 5            | 2.70               | 97.30             |
| 6            | 2.73               | 97.27             |
| 7            | 2.71               | 97.29             |
| 8            | 2.92               | 97.08             |
| 9            | 2.62               | 97.38             |

---

 
**7.	What is the percentage of sales by demographic for each year in the dataset?**

- Exactly similar to previous question, so directly jumping to query.

#### Final Query

````sql
SELECT 
    calender_year,
    ROUND(100.0 * couples_sales / (couples_sales + families_sales + unknown_sales), 2) AS couples_percentage,
    ROUND(100.0 * families_sales / (couples_sales + families_sales + unknown_sales), 2) AS families_percentage,
    ROUND(100.0 * unknown_sales / (couples_sales + families_sales + unknown_sales), 2) AS unknown_percentage
FROM (
    SELECT 
        calender_year,
        SUM(CASE WHEN demographic = 'Couples' THEN sales END) AS couples_sales,
        SUM(CASE WHEN demographic = 'Families' THEN sales END) AS families_sales,
  		SUM(CASE WHEN demographic = 'unknown' THEN sales END) as unknown_sales
    FROM CTE 
    GROUP BY 1
    ORDER BY 1
) temp;
````

#### Final Output

| calender_year | couples_percentage | families_percentage | unknown_percentage |
| ------------- | ------------------ | ------------------- | ------------------ |
| 2018          | 26.38              | 31.99               | 41.63              |
| 2019          | 27.28              | 32.47               | 40.25              |
| 2020          | 28.72              | 32.73               | 38.55              |

---


 
**8.	Which age_band and demographic values contribute the most to Retail sales?**

- We need data for Retial sales only : `WHERE platform = 'Retail'`
- We need age_band and demographic values : `SELECT age_band,demographic`
- Contributes the most : `ORDER BY sum(sales) DESC LIMIT 1`

#### Final Query
```` sql
SELECT age_band,demographic FROM cte 
WHERE platform = 'Retail'
GROUP BY 1,2 
ORDER BY sum(sales) DESC 
LIMIT 1;
````

#### Final Output
 
| age_band | demographic |
| -------- | ----------- |
| unknown  | unknown     |

---

**9.	Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?**

- Let us first find the avg transaction size for each year and platform in both methods.
- Method 1 : `avg(avg_transaction)`
- Method 2 : `sum(transactions)/sum(sales)`

#### Final Query
```` sql

    SELECT 
        calender_year,platform
        ,sum(sales)/sum(transactions) as overall_avg,
        round(avg(avg_transaction)) as avg_of_avg
    FROM CTE 
    GROUP BY 1,2
    ORDER BY 1,2
;
````
#### Final Output
 
| calender_year | platform | overall_avg | avg_of_avg |
| ------------- | -------- | ----------- | ---------- |
| 2018          | Retail   | 36          | 43         |
| 2018          | Shopify  | 192         | 188        |
| 2019          | Retail   | 36          | 42         |
| 2019          | Shopify  | 183         | 178        |
| 2020          | Retail   | 36          | 41         |
| 2020          | Shopify  | 179         | 175        |

- Again even though these values do not differ by that much, so I'd say both are fine.
- But again depending on specific use case, one will be more accurate than other.  

---

## Before and After Analysis

- One simple mistake to keep in mind is that we need the data for 4 weeks before and after 2020-06-15. So we need to do - '4 week'::interval and  +'4 week'::interval to consider the correct dates.
- Do not take week number for 2020-06-15 date and do -3 and +3 to week number. It is only right when 2020-06-15 is the exact start of the week.
- Also notice how I am going to use + 1 or - 1 day to get the correct week range.
 
**1) What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?**

#### Final Query

```` sql
,before_and_after_analysis AS (
    SELECT 
        SUM(CASE 
            WHEN week_date BETWEEN '2020-06-15'::date - '1 day'::interval - '4 week'::interval + '1 day'::interval AND '2020-06-15'::date - '1 day'::interval 
            THEN sales 
            END) AS before_sales,
        SUM(CASE 
            WHEN week_date BETWEEN '2020-06-15'::date AND '2020-06-15'::date + '4 week'::interval - '1 day'::interval
            THEN sales 
            END) AS after_sales
    FROM CTE
)
SELECT 
    before_sales,
    after_sales,
    after_sales - before_sales AS sales_variance,
    ROUND(100.0 * (after_sales - before_sales) / before_sales, 2) AS variance_percentage
FROM before_and_after_analysis;
````

#### Output Table

| before_sales | after_sales | sales_variance | variance_percentage |
| ------------ | ----------- | -------------- | ------------------- |
| 2345878357   | 2318994169  | -26884188      | -1.15               |

---

**2) What about the entire 12 weeks before and after?**

#### Final Query

```` sql
,before_and_after_analysis AS (
    SELECT 
        SUM(CASE 
            WHEN week_date BETWEEN '2020-06-15'::date - '1 day'::interval - '12 week'::interval + '1 day'::interval AND '2020-06-15'::date - '1 day'::interval 
            THEN sales 
            END) AS before_sales,
        SUM(CASE 
            WHEN week_date BETWEEN '2020-06-15'::date AND '2020-06-15'::date + '12 week'::interval - '1 day'::interval
            THEN sales 
            END) AS after_sales
    FROM CTE
)
SELECT 
    before_sales,
    after_sales,
    after_sales - before_sales AS sales_variance,
    ROUND(100.0 * (after_sales - before_sales) / before_sales, 2) AS variance_percentage
FROM before_and_after_analysis;
````

#### Output Table

| before_sales | after_sales | sales_variance | variance_percentage |
| ------------ | ----------- | -------------- | ------------------- |
| 7126273147   | 6973947753  | -152325394     | -2.14               |

---

**3) How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?**

- Checking the trend for 4 weeks before and after for each year.

#### Final Query 1

```` sql
,before_and_after_analysis AS (
    SELECT
  		calender_year,
        SUM(CASE 
            WHEN week_date BETWEEN (calender_year||'-06-15')::date - '1 day'::interval - '4 week'::interval + '1 day'::interval AND (calender_year||'-06-15')::date - '1 day'::interval 
            THEN sales 
            END) AS before_sales,
        SUM(CASE 
            WHEN week_date BETWEEN (calender_year||'-06-15')::date AND (calender_year||'-06-15')::date + '4 week'::interval - '1 day'::interval
            THEN sales 
            END) AS after_sales
    FROM CTE
  	GROUP BY calender_year
)
SELECT 
	calender_year,
    before_sales,
    after_sales,
    after_sales - before_sales AS sales_variance,
    ROUND(100.0 * (after_sales - before_sales) / before_sales, 2) AS variance_percentage
FROM before_and_after_analysis ORDER BY 1;
````
#### Output Table 1

| calender_year | before_sales | after_sales | sales_variance | variance_percentage |
| ------------- | ------------ | ----------- | -------------- | ------------------- |
| 2018          | 2125140809   | 2129242914  | 4102105        | 0.19                |
| 2019          | 2249989796   | 2252326390  | 2336594        | 0.10                |
| 2020          | 2345878357   | 2318994169  | -26884188      | -1.15               |

---

- Checking the trend for 12 weeks before and after for each year.

#### Final Query 2

```` sql
,before_and_after_analysis AS (
    SELECT
  		calender_year,
        SUM(CASE 
            WHEN week_date BETWEEN (calender_year||'-06-15')::date - '1 day'::interval - '12 week'::interval + '1 day'::interval AND (calender_year||'-06-15')::date - '1 day'::interval 
            THEN sales 
            END) AS before_sales,
        SUM(CASE 
            WHEN week_date BETWEEN (calender_year||'-06-15')::date AND (calender_year||'-06-15')::date + '12 week'::interval - '1 day'::interval
            THEN sales 
            END) AS after_sales
    FROM CTE
  	GROUP BY calender_year
)
SELECT 
	calender_year,
    before_sales,
    after_sales,
    after_sales - before_sales AS sales_variance,
    ROUND(100.0 * (after_sales - before_sales) / before_sales, 2) AS variance_percentage
FROM before_and_after_analysis;
````

#### Output Table 2

| calender_year | before_sales | after_sales | sales_variance | variance_percentage |
| ------------- | ------------ | ----------- | -------------- | ------------------- |
| 2018          | 6396562317   | 6500818510  | 104256193      | 1.63                |
| 2019          | 6883386397   | 6862646103  | -20740294      | -0.30               |
| 2020          | 7126273147   | 6973947753  | -152325394     | -2.14               |

---

## Bonus Question

**Which areas of the business have the highest negative impact in sales metrics performance in 2020 for the 12 week before and after period?**

- region
- platform
- age_band
- demographic
- customer_type

Very simple question, all we have to do is `GROUP BY metric` to analyse on that specific metric.

### Region Analysis
```` sql
,before_and_after_analysis AS (
    SELECT 
 region,
        SUM(CASE 
            WHEN week_date BETWEEN '2020-06-15'::date - '1 day'::interval - '12 week'::interval + '1 day'::interval AND '2020-06-15'::date - '1 day'::interval 
            THEN sales 
            END) AS before_sales,
        SUM(CASE 
            WHEN week_date BETWEEN '2020-06-15'::date AND '2020-06-15'::date + '12 week'::interval - '1 day'::interval
            THEN sales 
            END) AS after_sales
    FROM CTE GROUP BY region
)
SELECT 
region,
    before_sales,
    after_sales,
    after_sales - before_sales AS sales_variance,
    ROUND(100.0 * (after_sales - before_sales) / before_sales, 2) AS variance_percentage
FROM before_and_after_analysis
ORDER BY 5;
````
| region        | before_sales | after_sales | sales_variance | variance_percentage |
| ------------- | ------------ | ----------- | -------------- | ------------------- |
| ASIA          | 1637244466   | 1583807621  | -53436845      | -3.26               |
| OCEANIA       | 2354116790   | 2282795690  | -71321100      | -3.03               |
| SOUTH AMERICA | 213036207    | 208452033   | -4584174       | -2.15               |
| CANADA        | 426438454    | 418264441   | -8174013       | -1.92               |
| USA           | 677013558    | 666198715   | -10814843      | -1.60               |
| AFRICA        | 1709537105   | 1700390294  | -9146811       | -0.54               |
| EUROPE        | 108886567    | 114038959   | 5152392        | 4.73                |

---
### Platform Analysis

- I won't be showing the query for any of the following, but all we have to do is instead of region, we have to put our metric. I'll just show outputs.

| platform | before_sales | after_sales | sales_variance | variance_percentage |
| -------- | ------------ | ----------- | -------------- | ------------------- |
| Retail   | 6906861113   | 6738777279  | -168083834     | -2.43               |
| Shopify  | 219412034    | 235170474   | 15758440       | 7.18                |

---

### Age_band Analysis

| age_band     | before_sales | after_sales | sales_variance | variance_percentage |
| ------------ | ------------ | ----------- | -------------- | ------------------- |
| unknown      | 2764354464   | 2671961443  | -92393021      | -3.34               |
| Middle Aged  | 1164847640   | 1141853348  | -22994292      | -1.97               |
| Retirees     | 2395264515   | 2365714994  | -29549521      | -1.23               |
| Young Adults | 801806528    | 794417968   | -7388560       | -0.92               |

---
### Demographic Analysis

| demographic | before_sales | after_sales | sales_variance | variance_percentage |
| ----------- | ------------ | ----------- | -------------- | ------------------- |
| unknown     | 2764354464   | 2671961443  | -92393021      | -3.34               |
| Families    | 2328329040   | 2286009025  | -42320015      | -1.82               |
| Couples     | 2033589643   | 2015977285  | -17612358      | -0.87               |

---

### Customer_type Analysis


| customer_type | before_sales | after_sales | sales_variance | variance_percentage |
| ------------- | ------------ | ----------- | -------------- | ------------------- |
| Guest         | 2573436301   | 2496233635  | -77202666      | -3.00               |
| Existing      | 3690116427   | 3606243454  | -83872973      | -2.27               |
| New           | 862720419    | 871470664   | 8750245        | 1.01                |

---

### Please feel free to let me know if I have made any mistake or if you know a better approach to solve any question. If this helped you in anyway to improve your skills, just drop a message. It might not mean much to you, but it absolutely makes my day when I know that I’ve helped someone gain some knowledge.

### Anyways Happy Fiddling with the Data. See you in the next case study.
