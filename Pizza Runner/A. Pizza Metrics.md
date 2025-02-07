## 1. How many pizzas were ordered?
```
SELECT COUNT(order_id) AS total_pizzas
FROM pizza_runner.customer_orders;
```
**Output**  
![image](https://github.com/user-attachments/assets/06826227-d975-45bb-8aa6-e7e1d5008154)

## 2. How many unique customers orders were made?
```
SELECT COUNT(DISTINCT order_id) AS unique_orders
FROM pizza_runner.customer_orders;
```
**Output**  
![image](https://github.com/user-attachments/assets/176ff4bf-dae3-44d5-b6ff-c2382422c54e)

## 3. How many successful orders were delivered by each runner?
```
SELECT runner_id, COUNT(DISTINCT order_id) AS successful_orders
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id;
```
**Output**  
![image](https://github.com/user-attachments/assets/a20055e1-6e6a-437d-9a32-2903ec081379)

## 4. How many of each type of pizza were delivered?
```
SELECT pizza_id, COUNT(*) AS num_pizzas_delivered
FROM pizza_runner.customer_orders
GROUP BY pizza_id;
```
**Output**  
![image](https://github.com/user-attachments/assets/cb5c0321-f57e-4657-8d1e-b064577977a1)
