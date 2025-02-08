# C. Ingredient Optimisation
### 1. What are the standard ingredients for each pizza?
**Steps:**
- `JOIN` `pizza_recipes` and `pizza_toppings` based on `toppings` and `topping_id`
- Use `STRING_AGG` to aggregate topping_name into standard_ingredients
```
SELECT
    pn.pizza_name,
    STRING_AGG(CONVERT(VARCHAR, pt.topping_name), ', ') AS standard_ingredients
FROM pizza_runner.pizza_recipes AS pr
JOIN pizza_runner.pizza_names AS pn
ON pr.pizza_id = pn.pizza_id
JOIN pizza_runner.pizza_toppings AS pt
    ON CHARINDEX(CAST(pt.topping_id AS VARCHAR), pr.toppings) > 0
GROUP BY pn.pizza_name;
```
**Output**  
![image](https://github.com/user-attachments/assets/95413921-e0f9-48dc-8dad-8d5e33fc90b5)
<br/>
<br/>
### 2. What was the most commonly added extra?
**Steps:**
- Use `CROSS APPLY` and `STRING_SPLIT` to split the topping_id within column `extras`
- Then, `JOIN` `customer_orders` with `pizza_toppings`
- `COUNT` frequency by `topping_name`
```
WITH added_toppings AS (
	SELECT c.order_id, t.topping_name AS extra_topping
	FROM (
		SELECT order_id, extras
		FROM pizza_runner.customer_orders
		WHERE extras IS NOT NULL) AS c
	CROSS APPLY
		STRING_SPLIT(CONVERT(VARCHAR, c.extras), ',') AS split_extras
	JOIN pizza_runner.pizza_toppings AS t
	ON CAST(split_extras.value AS INT) = t.topping_id
)
SELECT CONVERT(VARCHAR, extra_topping) AS extra_topping, COUNT(*) AS frequency
FROM added_toppings
GROUP BY CONVERT(VARCHAR, extra_topping);
```
**Output**  
![image](https://github.com/user-attachments/assets/b4d5fadb-f81e-4fd2-9eff-2202bb0e74f1)
<br/>
Bacon is the most frequently added extra topping.
<br/>
<br/>
### 3. What was the most common exclusion?
Same approach as previous question about extras.
```
WITH excluded_toppings AS (
	SELECT c.order_id, t.topping_name AS exclusion_topping
	FROM (
		SELECT order_id, exclusions
		FROM pizza_runner.customer_orders
		WHERE exclusions IS NOT NULL) AS c
	CROSS APPLY
		STRING_SPLIT(c.exclusions, ',') AS split_exclusions
	JOIN pizza_runner.pizza_toppings AS t
	ON CAST(split_exclusions.value AS INT) = t.topping_id
)
SELECT CONVERT(VARCHAR, exclusion_topping) AS exclusion_topping, COUNT(*) AS frequency
FROM excluded_toppings
GROUP BY CONVERT(VARCHAR, exclusion_topping)
ORDER BY frequency DESC;
```
**Output**  
![image](https://github.com/user-attachments/assets/640cd73b-5cf7-4d9b-bc7c-2217fa8ff24d)
<br/>
Cheese is the most common excluded topping.
<br/>
<br/>
### 4. Generate an ingredient list for each pizza order
**Steps:**
- Create a CTE for standard_ingredients
- Create a CTE for extra toppings
- Create a CTE for exclusion toppings
- Join all 3 CTEs, then add '2x' before extra toppings, and remove exclusion toppings
```
WITH standard_recipes AS (
    SELECT r.pizza_id,
           STRING_AGG(CONVERT(VARCHAR, t.topping_name), ', ') AS standard_ingredients
    FROM pizza_runner.pizza_recipes AS r
    JOIN pizza_runner.pizza_toppings AS t
        ON CHARINDEX(CAST(t.topping_id AS VARCHAR), r.toppings) > 0
    GROUP BY r.pizza_id
),
added_toppings AS (
    SELECT c.order_id, STRING_AGG(CONVERT(VARCHAR, t.topping_name), ', ') AS extra_topping
    FROM (
        SELECT order_id, extras
        FROM pizza_runner.customer_orders
        WHERE extras IS NOT NULL
    ) AS c
    CROSS APPLY STRING_SPLIT(c.extras, ',') AS split_extras
    JOIN pizza_runner.pizza_toppings AS t
        ON t.topping_id = CAST(split_extras.value AS INT)
	GROUP BY c.order_id
),
excluded_toppings AS (
    SELECT c.order_id, STRING_AGG(CONVERT(VARCHAR, t.topping_name), ', ') AS exclusion_topping
    FROM (
        SELECT order_id, exclusions
        FROM pizza_runner.customer_orders
        WHERE exclusions IS NOT NULL
    ) AS c
    CROSS APPLY STRING_SPLIT(c.exclusions, ',') AS split_exclusions
    JOIN pizza_runner.pizza_toppings AS t
        ON t.topping_id = CAST(split_exclusions.value AS INT)
	GROUP BY c.order_id
),
summary AS (
    SELECT c.order_id, c.pizza_id, c.exclusions, c.extras,
           sr.standard_ingredients, added_toppings.extra_topping, excluded_toppings.exclusion_topping
    FROM pizza_runner.customer_orders AS c
    JOIN standard_recipes AS sr
        ON c.pizza_id = sr.pizza_id
    LEFT JOIN added_toppings ON c.order_id = added_toppings.order_id
    LEFT JOIN excluded_toppings ON c.order_id = excluded_toppings.order_id
)
SELECT order_id, pizza_id,
	CASE WHEN extra_topping IS NOT NULL THEN REPLACE(standard_ingredients, extra_topping, '2x' + extra_topping)
	WHEN exclusion_topping IS NOT NULL THEN REPLACE(standard_ingredients, exclusion_topping, '')
	ELSE standard_ingredients END AS standard_ingredients
FROM summary;
```
**Output**  
![image](https://github.com/user-attachments/assets/1ffc6138-2414-4886-a520-ff3673073c60)
