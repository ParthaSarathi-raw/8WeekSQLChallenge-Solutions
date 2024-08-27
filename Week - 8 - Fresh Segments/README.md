# Case Study - 8 : Fresh Segments

All the data and questions that I've answered in this markdown can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-8/).

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/iRdsT76vaus813crPP8Ma4/10) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.


## Data Exploration and Cleaning

**1) Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month**

 #### Final Query
```` sql
ALTER TABLE fresh_segments.interest_metrics 
ALTER month_year TYPE DATE using to_date(month_year,'mm-yyyy');
SELECT month_year FROM fresh_segments.interest_metrics
````
 #### Output Table
 
| month_year               |
| ------------------------ |
| 2018-07-01T00:00:00.000Z |
| 2018-07-01T00:00:00.000Z |
| 2018-07-01T00:00:00.000Z |
| 2018-07-01T00:00:00.000Z |
| 2018-07-01T00:00:00.000Z |

- We can see that the data type has changed.

---


**2) What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?**

 #### Final Query
```` sql
SELECT month_year,count(*) FROM fresh_segments.interest_metrics 
GROUP BY 1 ORDER BY month_year 
NULLS FIRST;
````
 #### Output Table

 - Only showing first 5 rows.


| month_year               | count |
| ------------------------ | ----- |
|                          | 1194  |
| 2018-07-01T00:00:00.000Z | 729   |
| 2018-08-01T00:00:00.000Z | 767   |
| 2018-09-01T00:00:00.000Z | 780   |
| 2018-10-01T00:00:00.000Z | 857   |

---

**3) What do you think we should do with these null values in the fresh_segments.interest_metrics**

- Get rid of them.

 #### Final Query
```` sql
DELETE FROM fresh_segments.interest_metrics WHERE month_year is NULL;
SELECT month_year,count(*) FROM fresh_segments.interest_metrics 
GROUP BY 1 ORDER BY month_year 
NULLS FIRST;
````
 #### Output Table
 
| month_year               | count |
| ------------------------ | ----- |
| 2018-07-01T00:00:00.000Z | 729   |
| 2018-08-01T00:00:00.000Z | 767   |
| 2018-09-01T00:00:00.000Z | 780   |
| 2018-10-01T00:00:00.000Z | 857   |
| 2018-11-01T00:00:00.000Z | 928   |

- Now we can see that the null values are gone.

---

**4) How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?**

 #### Final Query
```` sql
SELECT SUM(CASE WHEN id IS NULL THEN 1 ELSE 0 END) AS ids_present_in_metrics_but_not_in_map
FROM fresh_segments.interest_metrics met
LEFT JOIN fresh_segments.interest_map map ON met.interest_id::integer = map.id;

SELECT SUM(CASE WHEN interest_id IS NULL THEN 1 ELSE 0 END) AS ids_present_in_map_but_not_in_metrics
FROM fresh_segments.interest_metrics met
RIGHT JOIN fresh_segments.interest_map map ON met.interest_id::integer = map.id;
````
 #### Output Table
 
| ids_present_in_metrics_but_not_in_map |
| ------------------------------------- |
| 0                                     |

| ids_present_in_map_but_not_in_metrics |
| ------------------------------------- |
| 7                                     |

---

**5) Summarise the id values in the fresh_segments.interest_map by its total record count in this table**

 #### Final Query
```` sql
SELECT id,interest_name,count(*) 
FROM fresh_segments.interest_metrics met
RIGHT JOIN fresh_segments.interest_map map ON met.interest_id::integer = map.id
GROUP BY 1,2
ORDER BY 3 DESC;
````
 #### Output Table

- Showing first 5 rows only.
  
| id    | interest_name                                        | count |
| ----- | ---------------------------------------------------- | ----- |
| 19622 | Quads and ATV Enthusiasts                            | 14    |
| 10008 | Japanese Luxury Car Enthusiasts                      | 14    |
| 5896  | Orthopedic Health Researchers                        | 14    |
| 4916  | Arthritis Sufferers                                  | 14    |
| 6171  | High-End Camera Shoppers                             | 14    |

---

**6) What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.**

- What type of join we are going to use depends on the order of the tables we are joning. As there are 7 ids which are in interest_map, but not present in interest metrics, there is no point in having them. So simple INNER JOIN would be sufficient I guess.

 #### Final Query
```` sql
SELECT *
FROM fresh_segments.interest_metrics met
JOIN fresh_segments.interest_map map ON met.interest_id::integer = map.id
WHERE id = 21246;
````

 #### Output Table

| _month | _year | month_year               | interest_id | composition | index_value | ranking | percentile_ranking | id    | interest_name                    | interest_summary                                      | created_at               | last_modified            |
| ------ | ----- | ------------------------ | ----------- | ----------- | ----------- | ------- | ------------------ | ----- | -------------------------------- | ----------------------------------------------------- | ------------------------ | ------------------------ |
| 7      | 2018  | 2018-07-01T00:00:00.000Z | 21246       | 2.26        | 0.65        | 722     | 0.96               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 8      | 2018  | 2018-08-01T00:00:00.000Z | 21246       | 2.13        | 0.59        | 765     | 0.26               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 9      | 2018  | 2018-09-01T00:00:00.000Z | 21246       | 2.06        | 0.61        | 774     | 0.77               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 10     | 2018  | 2018-10-01T00:00:00.000Z | 21246       | 1.74        | 0.58        | 855     | 0.23               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 11     | 2018  | 2018-11-01T00:00:00.000Z | 21246       | 2.25        | 0.78        | 908     | 2.16               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 12     | 2018  | 2018-12-01T00:00:00.000Z | 21246       | 1.97        | 0.7         | 983     | 1.21               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 1      | 2019  | 2019-01-01T00:00:00.000Z | 21246       | 2.05        | 0.76        | 954     | 1.95               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 2      | 2019  | 2019-02-01T00:00:00.000Z | 21246       | 1.84        | 0.68        | 1109    | 1.07               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 3      | 2019  | 2019-03-01T00:00:00.000Z | 21246       | 1.75        | 0.67        | 1123    | 1.14               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |
| 4      | 2019  | 2019-04-01T00:00:00.000Z | 21246       | 1.58        | 0.63        | 1092    | 0.64               | 21246 | Readers of El Salvadoran Content | People reading news from El Salvadoran media sources. | 2018-06-11T17:50:04.000Z | 2018-06-11T17:50:04.000Z |

---

 
**7) Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?**

 #### Final Query
```` sql
SELECT count(*)
FROM fresh_segments.interest_metrics met
JOIN fresh_segments.interest_map map ON met.interest_id::integer = map.id
WHERE month_year < created_at;

SELECT count(*)
FROM fresh_segments.interest_metrics met
JOIN fresh_segments.interest_map map ON met.interest_id::integer = map.id
WHERE month_year = date_trunc('month',created_at);
````
 #### Output Table
 
- Query : 1

| count |
| ----- |
| 188   |

- Query : 2
  
| count |
| ----- |
| 188   |

- Yes there are 188 queries whose month_year < created_date. But this is because month_year is the first date of that particular month. We have confirmed this by running the second query which gives the same exact count of 188.
- Hence there are correct values only, no need to remove them.

---
 ## Interest Analysis

**1) Which interests have been present in all month_year dates in our dataset?**

 #### Final Query
```` sql
SELECT interest_name,id FROM (SELECT interest_name,id,month_year FROM CTE GROUP BY 1,2,3) temp
GROUP BY 1,2
HAVING count(*) = (SELECT count(distinct month_year) FROM CTE);
````
 #### Output Table

- Only showing 5 rows, the actual output has 480 rows, which represent every interest that is present in all months.

| interest_name                                        | id    |
| ---------------------------------------------------- | ----- |
| Ski House Second Home Owners                         | 6107  |
| Foreman and Construction Managers                    | 7529  |
| Los Angeles Trip Planners                            | 6302  |
| North Carolina Travel Researchers                    | 7540  |
| Food Industry Professionals                          | 7461  |

--- 

**2) Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?**

- I might be using the below CTE table in other questions as well.

 #### Final Query
```` sql
WITH CTE as (
SELECT *
FROM fresh_segments.interest_metrics met
JOIN fresh_segments.interest_map map ON met.interest_id::integer = map.id),
CTE2 AS (
    SELECT no_of_months, COUNT(id) 
    FROM (
        SELECT interest_name, id, COUNT(*) AS no_of_months 
        FROM (
            SELECT interest_name, id, month_year 
            FROM CTE 
            GROUP BY interest_name, id, month_year
        ) AS temp1
        GROUP BY interest_name, id
        ORDER BY no_of_months DESC
    ) AS temp2
    GROUP BY no_of_months
    ORDER BY no_of_months DESC
)

SELECT * 
FROM (
    SELECT no_of_months,
           ROUND(100.0 * SUM(count) OVER (ORDER BY no_of_months DESC) / SUM(count) OVER (), 2) AS cumulative_percentage
    FROM CTE2
) AS temp
WHERE cumulative_percentage > 90;
````
 #### Output Table
 
| no_of_months | cumulative_percentage |
| ------------ | --------------------- |
| 6            | 90.85                 |
| 5            | 94.01                 |
| 4            | 96.67                 |
| 3            | 97.92                 |
| 2            | 98.92                 |
| 1            | 100.00                |

---

**3) If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?**

- This question really didn't make any sense to me. So I'm assuming we have to remove all the interest_id values which have no_of_months < 6.
- If you do understand what this question means, please let me know and I will make necessary changes.

 #### Final Query
```` sql
SELECT COUNT(interest_id) 
FROM cte 
WHERE interest_id IN (
    SELECT interest_id 
    FROM cte 
    GROUP BY interest_id 
    HAVING COUNT(DISTINCT month_year) < 6
);
````
 #### Output Table
 
| count |
| ----- |
| 400   |

---

**4) Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.**

- These interests have < 6 months of data, so we can't really use this less data to draw any meaning conclusion, so it kinda makes sense to avoid these while doing our analysis.
- And we are also only just removing a small percentage from the data, so no problem.
  
 #### Final Query
```` sql
, cte2 AS (
    SELECT 
        month_year,
        SUM(CASE 
                WHEN interest_id NOT IN (
                    SELECT interest_id 
                    FROM cte 
                    GROUP BY interest_id 
                    HAVING COUNT(DISTINCT month_year) < 6
                ) 
                THEN 1 
            END) AS not_removed,
        SUM(CASE 
                WHEN interest_id IN (
                    SELECT interest_id 
                    FROM cte 
                    GROUP BY interest_id 
                    HAVING COUNT(DISTINCT month_year) < 6
                ) 
                THEN 1 
            END) AS removed
    FROM cte 
    GROUP BY month_year
    ORDER BY month_year
)

SELECT 
    month_year,
    not_removed,
    removed,
    100.0 * removed / (removed + not_removed) AS perc_removed
FROM cte2;
````
 #### Output Table

| month_year               | not_removed | removed | perc_removed           |
| ------------------------ | ----------- | ------- | ---------------------- |
| 2018-07-01T00:00:00.000Z | 709         | 20      | 2.7434842249657064     |
| 2018-08-01T00:00:00.000Z | 752         | 15      | 1.9556714471968709     |
| 2018-09-01T00:00:00.000Z | 774         | 6       | 0.76923076923076923077 |
| 2018-10-01T00:00:00.000Z | 853         | 4       | 0.46674445740956826138 |
| 2018-11-01T00:00:00.000Z | 925         | 3       | 0.32327586206896551724 |
| 2018-12-01T00:00:00.000Z | 986         | 9       | 0.90452261306532663317 |
| 2019-01-01T00:00:00.000Z | 966         | 7       | 0.71942446043165467626 |
| 2019-02-01T00:00:00.000Z | 1072        | 49      | 4.3710972346119536     |
| 2019-03-01T00:00:00.000Z | 1078        | 58      | 5.1056338028169014     |
| 2019-04-01T00:00:00.000Z | 1035        | 64      | 5.8234758871701547     |
| 2019-05-01T00:00:00.000Z | 827         | 30      | 3.5005834305717620     |
| 2019-06-01T00:00:00.000Z | 804         | 20      | 2.4271844660194175     |
| 2019-07-01T00:00:00.000Z | 836         | 28      | 3.2407407407407407     |
| 2019-08-01T00:00:00.000Z | 1062        | 87      | 7.5718015665796345     |

---


**5) After removing these interests - how many unique interests are there for each month?**

 #### Final Query
```` sql
SELECT month_year,count(distinct id) as distinct_interests 
FROM cte 
WHERE interest_id NOT IN (
    SELECT interest_id 
    FROM cte 
    GROUP BY interest_id 
    HAVING COUNT(DISTINCT month_year) < 6
)
GROUP BY month_year;
````
 #### Output Table

| month_year               | distinct_interests |
| ------------------------ | ------------------ |
| 2018-07-01T00:00:00.000Z | 709                |
| 2018-08-01T00:00:00.000Z | 752                |
| 2018-09-01T00:00:00.000Z | 774                |
| 2018-10-01T00:00:00.000Z | 853                |
| 2018-11-01T00:00:00.000Z | 925                |
| 2018-12-01T00:00:00.000Z | 986                |
| 2019-01-01T00:00:00.000Z | 966                |
| 2019-02-01T00:00:00.000Z | 1072               |
| 2019-03-01T00:00:00.000Z | 1078               |
| 2019-04-01T00:00:00.000Z | 1035               |
| 2019-05-01T00:00:00.000Z | 827                |
| 2019-06-01T00:00:00.000Z | 804                |
| 2019-07-01T00:00:00.000Z | 836                |
| 2019-08-01T00:00:00.000Z | 1062               |

---

## Segment Analysis
- Segment Analysis needs to be done on the interests which have more than or equal to 6 months worth of data. So for this segment , we will be using new CTE2 which has all the data with < 6 months removed.

```` sql
ALTER TABLE fresh_segments.interest_metrics 
ALTER month_year TYPE DATE using to_date(month_year,'mm-yyyy');
DELETE FROM fresh_segments.interest_metrics WHERE month_year is NULL;

WITH CTE as (
SELECT *
FROM fresh_segments.interest_metrics met
JOIN fresh_segments.interest_map map ON met.interest_id::integer = map.id),
CTE2 as (
SELECT *
FROM cte 
WHERE interest_id NOT IN (
    SELECT interest_id 
    FROM cte 
    GROUP BY interest_id 
    HAVING COUNT(DISTINCT month_year) < 6
))
SELECT * FROM CTE2;
````

**1) Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year**

 #### Final Query
```` sql
CTE3 as (
SELECT * FROM CTE2 JOIN (SELECT id,max(composition) FROM CTE2 GROUP BY id) temp on 
CTE2.id = temp.id and CTE2.composition = temp.max)

SELECT interest_name, month_year FROM CTE3 ORDER BY composition DESC LIMIT 10; -- Top 10
````
```` sql
SELECT interest_name, month_year FROM CTE3 ORDER BY composition ASC LIMIT 10; -- Bottom 10
````
 #### Output Table
 
- Top 10

| interest_name                     | month_year               |
| --------------------------------- | ------------------------ |
| Work Comes First Travelers        | 2018-12-01T00:00:00.000Z |
| Gym Equipment Owners              | 2018-07-01T00:00:00.000Z |
| Furniture Shoppers                | 2018-07-01T00:00:00.000Z |
| Luxury Retail Shoppers            | 2018-07-01T00:00:00.000Z |
| Luxury Boutique Hotel Researchers | 2018-10-01T00:00:00.000Z |
| Luxury Bedding Shoppers           | 2018-12-01T00:00:00.000Z |
| Shoe Shoppers                     | 2018-07-01T00:00:00.000Z |
| Cosmetics and Beauty Shoppers     | 2018-07-01T00:00:00.000Z |
| Luxury Hotel Guests               | 2018-07-01T00:00:00.000Z |
| Luxury Retail Researchers         | 2018-07-01T00:00:00.000Z |

- Bottom 10

| interest_name                     | month_year               |
| --------------------------------- | ------------------------ |
| Astrology Enthusiasts             | 2018-08-01T00:00:00.000Z |
| Medieval History Enthusiasts      | 2018-10-01T00:00:00.000Z |
| Dodge Vehicle Shoppers            | 2019-03-01T00:00:00.000Z |
| Xbox Enthusiasts                  | 2018-07-01T00:00:00.000Z |
| Camaro Enthusiasts                | 2018-10-01T00:00:00.000Z |
| League of Legends Video Game Fans | 2019-01-01T00:00:00.000Z |
| Budget Mobile Phone Researchers   | 2019-08-01T00:00:00.000Z |
| Super Mario Bros Fans             | 2018-07-01T00:00:00.000Z |
| Oakland Raiders Fans              | 2019-08-01T00:00:00.000Z |
| Budget Wireless Shoppers          | 2018-07-01T00:00:00.000Z |

---

**2) Which 5 interests had the lowest average ranking value?**

 #### Final Query

 #### Output Table
 
**3) Which 5 interests had the largest standard deviation in their percentile_ranking value?**

 #### Final Query

 #### Output Table
 
**4) For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?**

 #### Final Query

 #### Output Table
 
**5) How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?**

 #### Final Query

 #### Output Table
 
