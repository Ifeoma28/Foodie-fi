# Foodie-fi

## Overview 
Subscription based businesses are super popular and Danny realized that there was a large gap in the market – he wanted to create a new streaming service that only had food related contents. With the help of a few smart friends Danny launched his new start-up Foodie-FI in 2020 and started selling monthly and annual subscriptions giving their customers something like Netflix but with only cooking shows. This case study focuses on using subscription style digital data to answer important business questions.

### Problem Statement
-	We want to understand each customer’s onboarding journey.
-	We want to create a payments table for the Year 2020 that includes amounts paid by each customer in the subscriptions table.
-	The rate of growth for Foodie-fi and the customer churn rate.
-	Key metrics for Foodie-fi management to track performance of their overall business.

### Data source
This data was sourced from Danny Ma.

### Methodology
The table was created in Microsoft sql server and there were no errors in the dataset.
Sql was used for answering important business questions.
Power bi was used for creating visuals.

### Business questions
Using a sample data set of 8 customers, we want to have a short glimpse of our data
```
SELECT s.customer_id,p.plan_name,p.price,s.start_date
FROM subscriptions s
LEFT JOIN plans p ON
s.plan_id = p.plan_id
WHERE customer_id <= 8
```
-- 8 customers upgrade their plans after using the free trial
-- no customer canceled after the trial period
-- 6 customers upgraded their plans once
-- 2 customers upgraded their plans twice
-- 2 customers churned after upgrading their plan once

- Now we want to know how many customers Foodie-fi has ever had ?
```
SELECT COUNT(DISTINCT customer_id) AS total_customers
FROM subscriptions;
```
This shows a total of 1000 customers for Foodie-fi.

- What is the monthly distribution of trial plan start_date values for our dataset - using the start of the month as the group by value
```
SELECT count(p.plan_name) AS monthly_distribution,DATEFROMPARTS(YEAR(s.start_date),MONTH(s.start_date),01)  AS startmonth
FROM subscriptions s
INNER JOIN plans p ON
s.plan_id = p.plan_id
WHERE p.plan_name ='trial'
GROUP BY DATEFROMPARTS(YEAR(s.start_date),MONTH(s.start_date),01)
ORDER BY monthly_distribution DESC;
```
 March has the highest distribution of free trial users and February has the least

- What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
```
SELECT COUNT(*) AS customer_count,plan_name FROM
(SELECT  YEAR(start_date) AS year,customer_id,s.plan_id,start_date,p.plan_name
FROM subscriptions s
INNER JOIN plans p ON s.plan_id = p.plan_id
WHERE YEAR(start_date) > 2020) AS plans_after_2020
GROUP BY plan_name;
```
 After the year 2020, 8 customers subscribed to basic monthly, 71 customers churned
63 customers subscribed to pro annual
 60 customers subscribed to pro monthly.

- What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```
SELECT ((CAST (churned_customers AS FLOAT))/1000)*100  AS customer_churned_percent
			FROM 
				(SELECT COUNT(DISTINCT customer_id) AS churned_customers,plan_id
					FROM subscriptions
						WHERE plan_id = 4
						GROUP BY plan_id) AS churned_table;
```
307 customers have churned   with a percentage of 30.7 

- How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

 how many customers have churned straight after their initial free trial
here we have to use CTEs 
 one table for  for their next plan and a row number to rank all the customers according to their id

```
WITH next_plan AS (
SELECT customer_id,start_date, ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date) AS rnk,plan_name
FROM subscriptions s
INNER JOIN plans p ON s.plan_id = p.plan_id 
)
SELECT *,COUNT(*) OVER() AS churned_after_free_trial,
ROUND((CAST( (COUNT(*) OVER ()) AS FLOAT)/307)*100,2) AS churned_after_trial_percent,
-- i divided by 307 because we have 307 churned out customers
ROUND((CAST( (COUNT(*) OVER ()) AS FLOAT)/1000)*100,2) AS churned_after_trial_total_customers
FROM next_plan 
WHERE 
 rnk = 2 AND plan_name = 'churn';
```
92 customers churned out after free trial	
29.97 percent of churned out customers churned out after the free trial
9.2 percent of total customers churned out after the free trial



