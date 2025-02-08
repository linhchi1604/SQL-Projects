# D. Pricing and Ratings
### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes. How much money has Pizza Runner made so far if there are no delivery fees?
```
WITH revenue AS (
	SELECT c.order_id, c.pizza_id,
		CASE WHEN n.pizza_name LIKE '%Meatlovers%' THEN 12
		ELSE 10 END AS sales
	FROM pizza_runner.customer_orders AS c
	INNER JOIN pizza_runner.pizza_names AS n
	ON c.pizza_id = n.pizza_id
)
SELECT SUM(sales) AS total_revenue
FROM revenue;
```
**Output**  
![image](https://github.com/user-attachments/assets/41ff9cc0-819e-40b6-aafe-76238ad781fb)
<br/>
<br/>
### 2. What if there was an additional $1 charge for any pizza extras?
**Steps:**
- Create a CTE with basic price and additional costs
- Use `LEN` to calculate the number of extras in each order
```
WITH revenue AS (
	SELECT
		CASE WHEN n.pizza_name LIKE 'Meatlovers' THEN 12
		ELSE 10 END AS sales,
		CASE WHEN c.extras IS NOT NULL THEN LEN(c.extras) - LEN(REPLACE(c.extras, ',', '')) + 1
		ELSE 0 END AS extra_costs
	FROM pizza_runner.customer_orders AS c
	INNER JOIN pizza_runner.pizza_names AS n
	ON c.pizza_id = n.pizza_id
)
SELECT SUM(sales) + SUM(extra_costs) AS total_revenue
FROM revenue;
```
**Output**  
![image](https://github.com/user-attachments/assets/45259dd8-7a23-446b-a3ce-c189ea5a268e)
<br/>
<br/>
### 3. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled. How much money does Pizza Runner have left over after these deliveries?
**Steps:**
- Create a CTE to calculate sales
- Create a CTE to calculate delivery costs
- Join these two CTEs, group by order_id
- Profit = Sales - Delivery costs
```
WITH revenue AS (
	SELECT order_id,
		CASE WHEN n.pizza_name LIKE 'Meatlovers' THEN 12
		ELSE 10 END AS sales
	FROM pizza_runner.customer_orders AS c
	INNER JOIN pizza_runner.pizza_names AS n
	ON c.pizza_id = n.pizza_id
),
costs AS (
	SELECT order_id, CONVERT(FLOAT, distance) * 0.3 AS delivery_costs
	FROM pizza_runner.runner_orders
),
summary AS (
	SELECT revenue.order_id, SUM(sales) AS sales, SUM(delivery_costs) AS delivery_costs
	FROM revenue
	INNER JOIN costs ON revenue.order_id = costs.order_id
	GROUP BY revenue.order_id
)
SELECT order_id, sales, delivery_costs,
	CASE WHEN delivery_costs IS NOT NULL THEN sales - delivery_costs
	ELSE NULL END AS profit
FROM summary;
```
**Output**  
![image](https://github.com/user-attachments/assets/09827968-7022-4a4a-9d02-b845848d9aab)
