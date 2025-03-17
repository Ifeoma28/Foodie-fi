# Foodie-fi
![foodie fi dashboard](https://github.com/Ifeoma28/Foodie-fi/blob/4cf085d4ce31b30dec1b4ec94d1d9fb7a4ab9b4b/foodie-fi%20dashboard%20details.png
)
## Overview 
Subscription based businesses are super popular and Danny realized that there was a large gap in the market – he wanted to create a new streaming service that only had food related contents. With the help of a few smart friends Danny launched his new start-up Foodie-FI in 2020 and started selling monthly and annual subscriptions giving their customers something like Netflix but with only cooking shows. This case study focuses on using subscription style digital data to answer important business questions.

### Problem Statement
- We want to understand each customer’s onboarding journey.
- We want to calculate the total revenue  for the Year 2020 and 2021 that includes amounts paid by each customer in the subscriptions table.
- The rate of growth for Foodie-fi and the customer churn rate.
- Key metrics for Foodie-fi management to track performance of their overall business.

### Data source
This data was sourced from [Danny Ma](https://www.linkedin.com/in/datawithdanny)

### Methodology
The table was created in Microsoft sql server and there were no errors in the dataset.
- Sql was used for answering important business questions.
- Power bi was used for creating visuals.

### About Dataset 
Customers can choose which plans to join Foodie-Fi when they first sign up. The plans table comprises of;

- Basic plan customers have limited access and can only stream their videos and is only available monthly at $9.90

- Pro plan customers have no watch time limits and are able to download videos for offline viewing. Pro plans start at $19.90 a month or $199 for an annual subscription.

- Customers can sign up to an initial 7 day free trial will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.

- When customers cancel their Foodie-Fi service - they will have a churn plan record with a null price but their plan will continue until the end of the billing period.
![subscription price](https://github.com/Ifeoma28/Foodie-fi/blob/dba4ecdcefa4ecfffd7d3c7bcb2f0f867d9cfdfa/subscription%20prices.png)

Next is the subscriptions table,
Customer subscriptions show the exact date where their specific plan_id starts.

- If customers downgrade from a pro plan or cancel their subscription - the higher plan will remain in place until the period is over - the **start_date** in the subscriptions table will reflect the date that the actual plan changes.

- When customers upgrade their account from a basic plan to a pro or annual pro plan - the higher plan will take effect straightaway.

- When customers churn - they will keep their access until the end of their current billing period but the start_date will be technically the day they decided to cancel their service.
  
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

![Free trial distribution](https://github.com/Ifeoma28/Foodie-fi/blob/2f8a062e84f51f3c8b1defe6563a88cb8e8cd232/free%20trial%20users.png)
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
![after year 2020](https://github.com/Ifeoma28/Foodie-fi/blob/b367769d385fc9217e082f6f80cdb6fa6aa51f4f/2021%20subscribers.png)
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
92 customers churned out after free trial.	
29.97 percent of churned out customers churned out after the free trial.
9.2 percent of the total customers churned out after the free trial.

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
![breakdown at 31st December 2020](https://github.com/Ifeoma28/Foodie-fi/blob/4f0f7f04307a2c12d464d6358427b61f79643724/subscribers%20end%20of%202020.png)


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
![pro annual users](https://github.com/Ifeoma28/Foodie-fi/blob/c4708a20323c15daabcfb2475cb8e7e5f1cd4359/pro%20annual%20users.png)
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

## Analysis 
- Churn outs occur every month but mostly in March and November.
![churn out](https://github.com/Ifeoma28/Foodie-fi/blob/efe3c2eee7620a2dc5403207b59167abda5bddc4/churn%20rate.png)
- Active subscribers dropped from 136 to 126.
![Active](https://github.com/Ifeoma28/Foodie-fi/blob/c6c0ea8fabb3177bd69f47b17b226f05c2fbb95a/active%20subscribers.png)

- Pro monthly users in the month of November is the same as the number of churned out customers in that month.
![customer behavior](https://github.com/Ifeoma28/Foodie-fi/blob/f0a7f24f2f556eae9e36372099a27ad742a9baef/customer%20behaviour.png)

- The churn out rate (30.7) is greater than the rate at which customers upgrade to pro annual (25.8)
![churn out vs pro annual](https://github.com/Ifeoma28/Foodie-fi/blob/6afec9c226c7bc08ee437720d7f8218b182c6210/pro%20annual%20vs%20churn.png)

## Key Insights
- March has the highest distribution of free trials while February has the least
- Out of 1000 customers, 307 customers have churned (did not renew their subscription) with a churn rate of 30.7
- Only 92 out of 307 churned out customers churned after their initial free trial.
- The highest number of times a customer renewed their plans was 3 times.
- The minimum subscription duration  to annual plan from the day they joined Foodie-fi is 7days and the maximum  is 346 days.
- None of the customers downgraded their plans to a lesser plan which shows that customers had no issues with the price but rather the food content. 
- The percentage churn rate and the number of Active subscribers are a useful metric in tracking the performance of Foodie-fi.
- From August to September was the most active period comprising more of pro monthly and basic users.

## Recommendations
- An exit survey should be created for customers who wish to leave after the free trial
- Open ended questions like; How was your streaming experience on Foodie-fi, What can be done to improve your experience,What features did you like or dislike the most?,Would you consider subscribing in the future, if no, why and also give a rating based on your free trial.
- Also personalized offers and discounts for customers who rated their free trial highly without subscribing.
- This would help us to analyze further customer churn out after the free trial and thereby reduce the churn out rate
- The number of Active subscribers monthly is a useful metric in tracking the performance of Foodie-fi.
- A rating systems should be incorporated for active subscribers to help personalize content and improve user experience.
- Lastly, invest in advertising campaigns to increase visibility and referrals or affiliates programs for past users.


