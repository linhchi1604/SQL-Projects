### 1. How many pizzas were ordered?
```
SELECT COUNT(order_id) AS total_pizzas
FROM pizza_runner.customer_orders;
```
**Output**  
![image](https://github.com/user-attachments/assets/06826227-d975-45bb-8aa6-e7e1d5008154)
<br/>
<br/>
### 2. How many unique customers orders were made?
```
SELECT COUNT(DISTINCT order_id) AS unique_orders
FROM pizza_runner.customer_orders;
```
**Output**  
![image](https://github.com/user-attachments/assets/176ff4bf-dae3-44d5-b6ff-c2382422c54e)
<br/>
<br/>
### 3. How many successful orders were delivered by each runner?
```
SELECT runner_id, COUNT(DISTINCT order_id) AS successful_orders
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id;
```
**Output**  
![image](https://github.com/user-attachments/assets/a20055e1-6e6a-437d-9a32-2903ec081379)
<br/>
<br/>
### 4. How many of each type of pizza were delivered?
```
SELECT pizza_id, COUNT(*) AS num_pizzas_delivered
FROM pizza_runner.customer_orders
GROUP BY pizza_id;
```
**Output**  
![image](https://github.com/user-attachments/assets/cb5c0321-f57e-4657-8d1e-b064577977a1)
<br/>
<br/>
### 5. How many Vegetarian and Meatlovers were ordered by each customer?
```
SELECT c.customer_id, pizza_name, COUNT(*) AS num_orders
FROM pizza_runner.customer_orders AS c
INNER JOIN pizza_runner.pizza_names AS n
ON c.pizza_id = n.pizza_id
GROUP BY c.customer_id, pizza_name;
```
**Output**  
![image](https://github.com/user-attachments/assets/e26994f8-a5fa-4584-bfe8-7d19c6d559cc)
<br/>
<br/>
### 6. What was the maximum number of pizzas delivered in a single order?
```
SELECT TOP 1 order_id, COUNT(pizza_id) AS max_pizzas_delivered
FROM pizza_runner.customer_orders
GROUP BY order_id
ORDER BY max_pizzas_delivered DESC;
```
**Output**  
![image](https://github.com/user-attachments/assets/52b0fbb1-551f-4146-9de0-6f6613fc1bb9)
<br/>
<br/>
### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no change?
**Steps:**  
- Create a CTE with number of orders with at least 1 change
- Create a CTE with number of orders with no change
- UNION these two CTEs
```
WITH with_change AS (
	SELECT customer_id, COUNT(pizza_id) AS num_change, NULL AS num_no_change
	FROM pizza_runner.customer_orders
	WHERE exclusions IS NOT NULL OR extras IS NOT NULL
	GROUP BY customer_id
),
without_change AS (
	SELECT customer_id, NULL AS num_change, COUNT(pizza_id) AS num_no_change
	FROM pizza_runner.customer_orders
	WHERE exclusions IS NULL AND extras IS NULL
	GROUP BY customer_id
)
SELECT wc.customer_id, wc.num_change, wc.num_no_change
FROM with_change AS wc
UNION
SELECT woc.customer_id, woc.num_change, woc.num_no_change
FROM without_change AS woc;
```
**Output**  
![image](https://github.com/user-attachments/assets/d063e0a9-a9d3-4a34-aca6-371b373d843d)
<br/>
<br/>
### 8. How many pizzas were delivered that had both exclusions and extras?
**Approach:** Find orders where both `exclusions` and `extras` `IS NOT NULL`
```
SELECT COUNT(pizza_id) AS num_pizzas_excl_extra
FROM pizza_runner.customer_orders
WHERE exclusions IS NOT NULL AND extras IS NOT NULL;
```
**Output**  
![image](https://github.com/user-attachments/assets/ac583362-9ae7-43fe-a3ec-b2fcc8650b82)
<br/>
<br/>
### 9. What was the total volume of pizzas ordered for each hour of the day?
**Steps:**
- Extract hour from `order_time`
- `COUNT` number of pizzas `BY` hour
```
SELECT DISTINCT DATEPART(hour, order_time) AS order_hour, COUNT(pizza_id) AS total_pizzas
FROM pizza_runner.customer_orders
GROUP BY DATEPART(hour, order_time);
```
**Output**  
![image](https://github.com/user-attachments/assets/f58f2ba3-4e8e-48cf-93c3-708a5e3c398f)
<br/>
<br/>
### 10. What was the total volume of orders for each day of the week?
**Steps:**
- Extract day of week from `order_time`
- `COUNT` number of pizzas `BY` day of week
```
SELECT DISTINCT DATENAME(weekday, order_time) AS day_of_week, COUNT(DISTINCT order_id) AS total_orders
FROM pizza_runner.customer_orders
GROUP BY DATENAME(weekday, order_time);
```
**Output**  
![image](https://github.com/user-attachments/assets/9ab8871c-efdd-47e4-9775-42968609ffde)
