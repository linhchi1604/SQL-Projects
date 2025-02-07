### 1. How many runners signed up for each 1 week period?
```
SELECT DATEPART(week, registration_date) AS week_number, COUNT(runner_id) AS num_register
FROM pizza_runner.runners
GROUP BY DATEPART(week, registration_date);
```
**Output**  
![image](https://github.com/user-attachments/assets/12e2e9a6-d957-405b-aa83-32d28fd2e45b)
<br/>
<br/>
### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
**Steps:**
- Calculate the time difference between `order_time` and `pickup_time` for each runner
- Calculate `AVG` of all time difference
```
WITH time_diff AS (
	SELECT DISTINCT c.order_id, r.runner_id, c.order_time, r.pickup_time,
		DATEDIFF(MINUTE, c.order_time, r.pickup_time) AS arrival_time
	FROM pizza_runner.customer_orders AS c
	INNER JOIN pizza_runner.runner_orders AS r
	ON c.order_id = r.order_id
	WHERE r.pickup_time IS NOT NULL
)
SELECT runner_id, AVG(ARRIVAL_TIME) AS average_time
FROM time_diff
GROUP BY runner_id;
```
**Output**  
![image](https://github.com/user-attachments/assets/f1c91233-f1ee-4b61-8907-b2dee3f3f316)
<br/>
<br/>
### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
