# B. Runner and Customer Experience
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
**Steps:**
- Create a CTE `COUNT` number of pizzas of each order
- Calculate time to prepare of each order (time difference between `order_time` and `pickup_time`)
- Calculate `AVG` time to prepare, `GROUP BY` number of pizzas
```
WITH table_a AS (
	SELECT DISTINCT order_id, COUNT(pizza_id) AS num_pizzas
	FROM pizza_runner.customer_orders
	GROUP BY order_id
),
table_b AS (
	SELECT DISTINCT order_id, order_time
	FROM pizza_runner.customer_orders
),
table_c AS (
	SELECT order_id, pickup_time
	FROM pizza_runner.runner_orders
	WHERE pickup_time IS NOT NULL
),
table_d AS (
	SELECT table_a.order_id, num_pizzas, order_time, pickup_time,
		DATEDIFF(MINUTE, order_time, pickup_time) AS time_prepare
	FROM table_a
	INNER JOIN table_b ON table_a.order_id = table_b.order_id
	INNER JOIN table_c ON table_a.order_id = table_c.order_id
)
SELECT num_pizzas, AVG(time_prepare) AS avg_time_prepare
FROM table_d
GROUP BY num_pizzas;
```
**Output**  
![image](https://github.com/user-attachments/assets/a65fbe8a-2ddf-4a9a-aa78-34735f76c2fa)
<br/>
Yes, there is possibly a relationship between number of pizzas and time to prepare. When order has only 1 pizza, the average time to prepare is 12 minutes. When order has 2 or 3 pizzas, the average time to prepare increases, i.e., 18 and 30 minutes respectively.
<br/>
<br/>
### 4. What was the average distance travelled for each customer?
**Steps:**
- `LEFT JOIN` `customer_orders` with `runner_orders` to have `customer_id` and `distance` in the same table
- Calculate the `AVG` `distance`, `GROUP BY` `customer_id`
```
SELECT c.customer_id, ROUND(AVG(CONVERT(FLOAT, r.distance)), 2) AS average_distance
FROM pizza_runner.customer_orders AS c
LEFT JOIN pizza_runner.runner_orders AS r
ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL
GROUP BY c.customer_id;
```
**Output**  
![image](https://github.com/user-attachments/assets/042b667b-d093-41db-a351-a3fb9962e51c)
<br/>
<br/>
### 5. What was the difference between the longest and shortest delivery times for all orders?
```
SELECT MAX(CONVERT(FLOAT, duration)) - MIN(CONVERT(FLOAT, duration)) AS delivery_time_diff
FROM pizza_runner.runner_orders
WHERE duration IS NOT NULL;
```
**Output**  
![image](https://github.com/user-attachments/assets/3b1aa7b7-ab04-47ea-bd53-1c7c2e17b25a)
<br/>
<br/>
### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```
SELECT runner_id,
	ROUND(AVG(CONVERT(FLOAT, distance)), 2) AS avg_distance,
	ROUND(AVG(CONVERT(FLOAT, duration)), 2) AS avg_duration,
	ROUND(AVG(CONVERT(FLOAT, distance) / (CONVERT(FLOAT, duration))), 2) AS avg_speed
FROM pizza_runner.runner_orders
WHERE distance IS NOT NULL
GROUP BY runner_id;
```
**Output**  
![image](https://github.com/user-attachments/assets/090e2b20-7ee8-4e18-b96e-e2dbf71048be)
<br/>
As the average distance increases, the average duration also increaes.
<br/>
<br/>
### 7. What is the successful delivery percentage for each runner?
**Steps:**
- Create a CTE, within column `successful` assign `1` if order is successful (i.e., cancellation IS NULL) and assign `0` if order is cancelled (i.e., cancellation IS NOT NULL)
- Calculate average of column `successful`
```
WITH succeed AS (
SELECT runner_id,
	CASE WHEN cancellation IS NULL THEN 1
	ELSE 0 END AS successful
FROM pizza_runner.runner_orders
)
SELECT runner_id, ROUND(AVG(CONVERT(FLOAT, successful)) * 100, 2) AS pct_successful
FROM succeed
GROUP BY runner_id;
```
**Output**  
![image](https://github.com/user-attachments/assets/efab6e98-d092-4c54-a7f9-0f731c22cab6)
