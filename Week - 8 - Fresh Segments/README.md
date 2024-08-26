# Case Study - 8 : Fresh Segments

All the data and questions that I've answered in this markdown can be found at  [Dannys Website](https://8weeksqlchallenge.com/case-study-8/).

And all my solutions have been executed at [DB Fiddle](https://www.db-fiddle.com/f/iRdsT76vaus813crPP8Ma4/10) using POSTGRE SQL V 13. If you get any kind of errors when you are executing my queries given below, you might be using some other dialect of SQL other than postgre sql v 13. Please look up alternate ways to achieve the same thing on your SQL dialect. However I do prefer just practising these questions on DB Fiddle itself.


## Data Exploration and Cleaning

**1) Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month**

 #### Final Query

 #### Output Table
  
**2) What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?**

 #### Final Query

 #### Output Table
 
**3) What do you think we should do with these null values in the fresh_segments.interest_metrics**

 #### Final Query

 #### Output Table
 
**4) How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?**

 #### Final Query

 #### Output Table
 
**5) Summarise the id values in the fresh_segments.interest_map by its total record count in this table**

 #### Final Query

 #### Output Table
 
**6) What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.**

 #### Final Query

 #### Output Table
 
**7) Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?**

 #### Final Query

 #### Output Table

 ## Interest Analysis

**1) Which interests have been present in all month_year dates in our dataset?**

 #### Final Query

 #### Output Table
 
**2) Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?**

 #### Final Query

 #### Output Table
 
**3) If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?**

 #### Final Query

 #### Output Table
 
**4) Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.**

 #### Final Query

 #### Output Table
 
**5) After removing these interests - how many unique interests are there for each month?**

 #### Final Query

 #### Output Table
 
## Segment Analysis
**1) Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year**

 #### Final Query

 #### Output Table
 
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
 
