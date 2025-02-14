# B. Data Analysis Questions
### 1. How many customers has Foodie-Fi ever had?
```
SELECT COUNT(DISTINCT customer_id) AS total_customers
FROM foodie_fi.subscriptions;
```
**Output**  
![image](https://github.com/user-attachments/assets/bc5c178e-7bfb-4e7b-8709-5fc38b859d23)
<br/>
<br/>
### 2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the grouyp by value
**Steps:**
- Extract month from `start_date`
- Filter for `plan_name` = 'trial'
- `COUNT` `plan_id` by month
```
SELECT DATENAME(MONTH, s.start_date) AS month_name,
	COUNT(s.plan_id) AS trial_monthly_distribution
FROM foodie_fi.subscriptions AS s
INNER JOIN foodie_fi.plans AS p
ON s.plan_id = p.plan_id
WHERE p.plan_name = 'trial'
GROUP BY DATENAME(MONTH, start_date)
ORDER BY trial_monthly_distribution DESC;
```
**Output**  
![image](https://github.com/user-attachments/assets/9da56359-1464-4640-ac53-3442ef2bbc1a)
<br/>
<br/>
### 3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown count of events for each plan_name
**Steps:**
- Filter for plans with `start_date` from `2021-01-01`
- `GROUP BY` `plan_name` and `COUNT` number of plans
```
SELECT p.plan_name, COUNT(s.start_date) AS num
FROM foodie_fi.subscriptions AS s
INNER JOIN foodie_fi.plans AS p
ON s.plan_id = p.plan_id
WHERE start_date >= '2021-01-01'
GROUP BY p.plan_name;
```
**Output**  
![image](https://github.com/user-attachments/assets/014640a7-b53a-487e-aca0-e7dff9553983)
<br/>
<br/>
### 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
**Steps:**
- To count number of churned customers: filter for plan_id = 4 and count
- To calculate percentage of churned customers: number of churned customers / number of total customers
```
SELECT COUNT(*) AS num_churn,
	ROUND((CAST(COUNT(*) AS FLOAT) * 100 / (SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions)), 1) AS pct_churn
FROM foodie_fi.subscriptions
WHERE plan_id = 4;
```
**Output**  
![image](https://github.com/user-attachments/assets/da51579a-96af-4f45-a2e6-08463aab3a4b)
<br/>
<br/>
### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
**Steps:**
- Create a CTE with `current_plan` and `previous_plan` for each customer (i.e., using `LAG()` function)
- Filter for customers who churn right after trial (i.e., `current_plan = 4` and `previous_plan = 0`)
- Count number of customers and calculate percentage
```
WITH previous_plan_outline AS (
	SELECT customer_id, plan_id AS current_plan,
		LAG(plan_id, 1) OVER (PARTITION BY customer_id ORDER BY plan_id) AS previous_plan
	FROM foodie_fi.subscriptions
)
SELECT COUNT(previous_plan) AS num_churn_after_trial,
	(CAST(COUNT(*) AS INTEGER) * 100 / (SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions)) AS pct_churn_after_trial
FROM previous_plan_outline
WHERE current_plan = 4 AND previous_plan = 0;
```
**Output**  
![image](https://github.com/user-attachments/assets/08ca1484-5364-4397-921f-3a52de3bcd5c)
<br/>
<br/>
### 6. What is the number and percentage of customer plans after their initial free trial?
**Steps:**
- Create a CTE with `current_plan` and `previous_plan` for each customer (i.e., using `LAG()` function)
- Filter for `previous_plan = 0` (i.e, free trial plan) and examine the comparable `current_plan` to see which plan customers upgrade to after free trial
- `GROUP BY` `plan_name`, count number of customers and calculate percentage
```
WITH former_plan AS (
	SELECT customer_id, plan_id AS current_plan,
		LAG(plan_id, 1) OVER (PARTITION BY customer_id ORDER BY plan_id) AS previous_plan
	FROM foodie_fi.subscriptions
)
SELECT p.plan_name,
	COUNT(DISTINCT f.customer_id) AS num_customers,
	ROUND((CAST(COUNT(DISTINCT customer_id) AS FLOAT) * 100 / (SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions)), 2) AS pct_customers
FROM former_plan AS f
LEFT JOIN foodie_fi.plans AS p ON f.current_plan = p.plan_id
WHERE f.previous_plan = 0
GROUP BY plan_name;
```
**Output**  
![image](https://github.com/user-attachments/assets/3164616e-a845-4f04-9229-6ea86dcbf9af)
<br/>
<br/>
### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
**Steps:**
- Create a CTE with `current_plan` before `2020-12-31` of each customer
- `GROUP BY` `plan_name`, count number of customers and calculate percentage
```
WITH current_date_plan AS (
	SELECT customer_id, MAX(start_date) AS latest_date
	FROM foodie_fi.subscriptions
	WHERE start_date <= '2020-12-31'
	GROUP BY customer_id
)
SELECT plan_name,
	COUNT(DISTINCT cdp.customer_id) AS num_customers,
	ROUND(CAST(COUNT(DISTINCT cdp.customer_id) AS FLOAT) * 100 / (SELECT COUNT(DISTINCT customer_id) FROM foodie_fi.subscriptions), 2) AS pct_customers
FROM current_date_plan AS cdp
LEFT JOIN foodie_fi.subscriptions AS s
ON cdp.customer_id = s.customer_id AND cdp.latest_date = s.start_date
LEFT JOIN foodie_fi.plans AS p ON s.plan_id = p.plan_id
GROUP BY p.plan_name;
```
**Output**  
![image](https://github.com/user-attachments/assets/5cfddfb5-de23-4008-8218-fcebcf6f86ff)
<br/>
<br/>
### 8. How many customers have upgraded to annual plan in 2020?
**Steps:**
- Filter for customers with `annual plan` in `2020`
- Count number of customers
```
SELECT COUNT(DISTINCT s.customer_id) AS num_customers_annual_2020
FROM foodie_fi.subscriptions AS s
INNER JOIN foodie_fi.plans as p
ON s.plan_id = p.plan_id
WHERE p.plan_name = 'pro annual'
	AND DATEPART(YEAR, s.start_date) = 2020;
```
**Output**  
![image](https://github.com/user-attachments/assets/ed21890d-4cf6-49fc-9386-6519f0e1e362)
<br/>
<br/>
### 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
**Steps:**
- Create a CTE with `first date` that customers join Foodie-Fi
- Create a CTE with the date that customers `upgrade to annual plan`
- Calculate the time difference `DATEDIFF` between these two dates
- Calculate the average `AVG` converting time
```
WITH first_subscriptions AS (                          -- First date customers join Foodie-Fi
	SELECT customer_id, MIN(start_date) AS first_sub
	FROM foodie_fi.subscriptions
	GROUP BY customer_id
),
annual_subscriptions AS (                               -- Date customers upgrade to annual plan
	SELECT customer_id, start_date AS annual_sub
	FROM foodie_fi.subscriptions AS s
	INNER JOIN foodie_fi.plans AS p
	ON s.plan_id = p.plan_id
	WHERE p.plan_name = 'pro annual'
),
annual_plan_switch AS (                                  -- Calculate the time between first day and day upgrading to annual plan
	SELECT annual_subscriptions.customer_id, first_sub, annual_sub, DATEDIFF(DAY, first_sub, annual_sub) AS time_switch
	FROM annual_subscriptions
	LEFT JOIN first_subscriptions
	ON annual_subscriptions.customer_id = first_subscriptions.customer_id
)
SELECT AVG(time_switch) AS average_days_to_annual        -- Calculate average time
FROM annual_plan_switch;
```
**Output**  
![image](https://github.com/user-attachments/assets/c9ecef7f-56fe-4b62-9e48-9be04c517631)
<br/>
<br/>
### 10. Can you further breakdown this average value into 30 day periods (i.e., 0-30 days, 31-60 days, etc.)
**Steps:**
- Create a CTE with `first date` that customers join Foodie-Fi
- Create a CTE with the date that customers `upgrade to annual plan`
- Calculate the time difference `DATEDIFF` between these two dates
- Use `CASE WHEN` to categorize duration/time difference into 30-day periods
- Count number of customers and calculate `AVG` duration
```
WITH first_subscriptions AS (
	SELECT customer_id, MIN(start_date) AS first_sub
	FROM foodie_fi.subscriptions
	GROUP BY customer_id
),
annual_subscriptions AS (
	SELECT customer_id, start_date AS annual_sub
	FROM foodie_fi.subscriptions
	INNER JOIN foodie_fi.plans ON subscriptions.plan_id = plans.plan_id
	WHERE plans.plan_name = 'pro annual'
),
plan_switch AS (
	SELECT annual_subscriptions.customer_id, first_sub, annual_sub, DATEDIFF(DAY, first_sub, annual_sub) AS time_switch
	FROM annual_subscriptions
	LEFT JOIN first_subscriptions
	ON annual_subscriptions.customer_id = first_subscriptions.customer_id
)
SELECT
	duration, COUNT(*) AS num_customers, AVG(time_switch) AS average_days_to_annual
FROM (
	SELECT
		CASE WHEN time_switch <= 30 THEN ' 0-30 days'
		WHEN time_switch >= 31 AND time_switch <= 60 THEN ' 31-60 days'
		WHEN time_switch >= 61 AND time_switch <= 90 THEN ' 61-90 days'
		WHEN time_switch >= 91 AND time_switch <= 120 THEN ' 91-120 days'
		WHEN time_switch >= 120 AND time_switch <= 150 THEN '121-150 days'
		WHEN time_switch >= 151 AND time_switch <= 180 THEN '151-180 days'
		WHEN time_switch >= 181 AND time_switch <= 210 THEN '181-210 days'
		WHEN time_switch >= 211 AND time_switch <= 240 THEN '211-240 days'
		WHEN time_switch >= 241 AND time_switch <= 270 THEN '241-270 days'
		WHEN time_switch >= 271 AND time_switch <= 300 THEN '271-300 days'
		ELSE '301+ days' END AS duration,
		time_switch
	FROM plan_switch) AS group_duration
GROUP BY duration;
```
**Output**  
![image](https://github.com/user-attachments/assets/8494f071-3bda-4bad-825a-bdec6aa17748)
<br/>
<br/>
### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
**Steps:**
- Create a CTE with `pro monthly` customers in `2020`
- Create a CTE with `basic monthly` customers in `2020`
- Filter for cases where `pro monthly start_date < basic monthly start_date`
- Count number of customers
```
WITH pro_monthly AS (                      -- Date customers start pro monthly plan
	SELECT s.customer_id, s.start_date
	FROM foodie_fi.subscriptions AS s
	INNER JOIN foodie_fi.plans AS p ON s.plan_id = p.plan_id
	WHERE p.plan_name = 'pro monthly' AND DATEPART(YEAR, s.start_date) = 2020
),
basic_monthly AS (                         -- Date customers start basic monthly plan
	SELECT s.customer_id, s.start_date
	FROM foodie_fi.subscriptions AS s
	INNER JOIN foodie_fi.plans AS p ON s.plan_id = p.plan_id
	WHERE p.plan_name = 'basic monthly' AND DATEPART(YEAR, s.start_date) = 2020
),
downgraded_customers AS (                   -- Sort for cases where pro-monthly start date <  basic-monthly start date 
	SELECT pm.customer_id, pm.start_date AS pro_date, bm.start_date AS basic_date
	FROM pro_monthly AS pm
	INNER JOIN basic_monthly AS bm ON pm.customer_id = bm.customer_id
	WHERE pm.start_date < bm.start_date
)
SELECT COUNT(*) AS num_downraded_customers   -- Count number of customers downgrading from pro monthly to basic monthly plan
FROM downgraded_customers;
```
**Output**  
![image](https://github.com/user-attachments/assets/755a73f8-5597-45cc-8973-56a7aacbb954)
