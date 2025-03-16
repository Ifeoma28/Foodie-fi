# Foodie-fi
![foodie fi dashboard](https://github.com/Ifeoma28/Foodie-fi/blob/32ba8e7cc98c610fa76c5306d445be6169f670a4/FOODIE-FI%20dashboard.png)

## Overview 
Subscription based businesses are super popular and Danny realized that there was a large gap in the market – he wanted to create a new streaming service that only had food related contents. With the help of a few smart friends Danny launched his new start-up Foodie-FI in 2020 and started selling monthly and annual subscriptions giving their customers something like Netflix but with only cooking shows. This case study focuses on using subscription style digital data to answer important business questions.

### Problem Statement
-	We want to understand each customer’s onboarding journey.
-	We want to calculate the total revenue  for the Year 2020 and 2021 that includes amounts paid by each customer in the subscriptions table.
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
1. 8 customers upgrade their plans after using the free trial
2. no customer canceled after the trial period
3. 6 customers upgraded their plans once
4. 2 customers upgraded their plans twice
5. 2 customers churned after upgrading their plan once

- Now we want to know how many customers Foodie-fi has ever had ?
```
SELECT COUNT(DISTINCT customer_id) AS total_customers
FROM subscriptions;
```
This shows a total of 1000 customers for Foodie-fi.
Below is the number of customers that subscribed for each plans
![plans breakdown](https://github.com/Ifeoma28/Foodie-fi/blob/78145d163d637ff3c0038f0958f1016a5eec1b96/number%20of%20people%20that%20subscribed%20for%20various%20plans.png)
All the customers started with a free trial plan 
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

![Free trial distribution](https://github.com/Ifeoma28/Foodie-fi/blob/cb726360d58d7e1a355a7e5e59e4dbc6dec4b4a7/Free%20trial%20distribution.png)
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

![after year 2020](https://github.com/Ifeoma28/Foodie-fi/blob/42dee59193f3c6c42469e9bc7e16b2ba2520562a/2021%20breakdown%20of%20customer%20plans.png)
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

 To know how many customers have churned straight after their initial free trial
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

- What is the number and percentage of customer plans after their initial free trial?
```
WITH next_plan AS (
SELECT customer_id,start_date, ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date) AS rnk,plan_name
FROM subscriptions s
INNER JOIN plans p ON s.plan_id = p.plan_id 
)
SELECT COUNT(customer_id) AS customer_count,plan_name AS no_of_plans,
ROUND((CAST (COUNT(plan_name) AS FLOAT)/1000)*100,2) AS customer_plans_percent
FROM next_plan 
WHERE rnk = 2
GROUP BY plan_name
;
```
- What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
```
WITH last_plan AS (
	SELECT *,ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY start_date DESC ) AS rnk
	FROM subscriptions 
	WHERE start_date <= '2020-12-31'
	)
	SELECT COUNT(customer_id) AS customer_count,plan_name,((CAST (COUNT(plan_name) AS FLOAT)) /1000)*100 AS customer_plans_percent
	-- i divided by 1000 because we want a percentage of the total customers
	FROM last_plan l
	INNER JOIN plans p ON l.plan_id = p.plan_id
	WHERE rnk = 1
GROUP BY plan_name
;
```
![breakdown at 31st December 2020](https://github.com/Ifeoma28/Foodie-fi/blob/9538d38d82a78ad6805cd8e8babf5750fab59f75/end%20of%202020%20subscribers.png)

- How many customers have upgraded to an annual plan in 2020?
```
SELECT COUNT(*) FROM
	(SELECT s.customer_id,s.plan_id,p.plan_name,s.start_date
	FROM subscriptions s
	INNER JOIN plans p ON
	s.plan_id = p.plan_id
	WHERE s.plan_id = 3 AND s.start_date <= '2020-12-31') AS annual_plans;
```
195 customers upgraded to annual plans in 2020
![pro annual users](https://github.com/Ifeoma28/Foodie-fi/blob/05f49937837aa44439696d8cf97881306efaaa3f/Pro%20annual%20users.png)
This shows the growth of annual users monthly 
- How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```
WITH free_trial_users AS (
	SELECT customer_id,start_date
	FROM subscriptions 
	WHERE plan_id = 0),
	next_plan AS (
	SELECT s.customer_id,s.plan_id,s.start_date
	FROM subscriptions s
	JOIN free_trial_users t ON s.customer_id = t.customer_id
	WHERE  s.plan_id = 3 AND s.start_date > t.start_date
	GROUP BY s.customer_id,s.plan_id,s.start_date
	)
	SELECT t.customer_id,(DATEDIFF(DAY,t.start_date,n.start_date)) AS no_of_days,
	AVG(DATEDIFF(DAY,t.start_date,n.start_date)) OVER() AS average_no_days
	FROM free_trial_users t
	LEFT JOIN next_plan n ON t.customer_id = n.customer_id
    GROUP BY t.customer_id,n.start_date,t.start_date
	ORDER BY no_of_days DESC
    ;
    -- giving an average of 104
```
- How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```
 WITH pro_monthly_users AS (
	SELECT customer_id,start_date
	FROM subscriptions 
	WHERE plan_id = 2 AND (start_date BETWEEN '2020-01-01'AND'2020-12-31')
    ),
	next_plan AS (
	SELECT s.customer_id,s.plan_id,s.start_date
	FROM subscriptions s
	JOIN pro_monthly_users p ON s.customer_id = p.customer_id
	WHERE  s.start_date > p.start_date AND plan_id = 1 AND s.start_date <= '2020-12-31'
	)
	SELECT p.customer_id,COUNT(*) AS customer_count
	FROM pro_monthly_users p
	INNER JOIN next_plan n ON p.customer_id = n.customer_id
    GROUP BY p.customer_id;
```
no customers downgraded their plans from pro monthly to a basic monthly in 2020

## Key Insights
- March has the highest distribution of free trials while February has the least
- Out of 1000 customers, 307 customers have churned (did not renew their subscription) with a churn rate of 30.7
- Only 92 out of 307 churned out customers churned after their initial free trial.
- The highest number of times a customer renewed their plans was 3 times.
- The minimum number of days it took for a customer to subscribe to annual plan from the day they joined Foodie-fi is 7days and the maximum number is 346 days.
- None of the customers downgraded their plans to a lesser plan which indicates an arithmetic growth for Foodie-fi.
- The percentage churn rate and the upgrade rate are a useful metric in tracking the performance of Foodie-fi.
- The customer count after 2020-12-31 showed an increase in customer count


