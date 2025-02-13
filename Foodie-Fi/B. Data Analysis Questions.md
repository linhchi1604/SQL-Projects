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
