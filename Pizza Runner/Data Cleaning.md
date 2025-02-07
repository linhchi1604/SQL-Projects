The data needs cleaning since there are inconsistancy and missing values.

## Table 2: customer_orders
**Problem:** blank space, 'null' value in column `exclusions` and `extras`
- Update 'null' values as NULL
```
UPDATE pizza_runner.customer_orders
SET exclusions = REPLACE(exclusions, 'null', NULL)
WHERE exclusions = 'null';
```

```
UPDATE pizza_runner.customer_orders
SET extras = REPLACE(extras, 'null', NULL)
WHERE extras = 'null';
```

- Replace blank space with NULL
```
UPDATE pizza_runner.customer_orders
SET exclusions = REPLACE(exclusions, '', NULL)
WHERE exclusions = '';
```

```
UPDATE pizza_runner.customer_orders  
SET extras = REPLACE(extras, '', NULL)  
WHERE extras = '';
```

## Table 3: runner_orders
**Problem:** blank space, 'null' value, inconsistant format
- Column `distance`: Remove 'km', replace 'null' with NULL, convert data type to FLOAT
```
UPDATE pizza_runner.runner_orders
SET distance = 
	CASE WHEN CHARINDEX('k', distance) > 0 
       THEN SUBSTRING(distance, 1, CHARINDEX('k', distance) - 1)
       ELSE distance END; -- If 'k' is not found, return the whole distance value
```

```
UPDATE pizza_runner.runner_orders
SET distance = REPLACE(distance, 'null', NULL)
WHERE distance = 'null';
```

```
UPDATE pizza_runner.runner_orders
SET distance = CONVERT(FLOAT, distance);
```

- Column `duration`: Remove 'm', replace 'null' with NULL, convert data type to FLOAT
```
UPDATE pizza_runner.runner_orders
SET duration = 
	CASE WHEN CHARINDEX('m', duration) > 0 
       THEN SUBSTRING(duration, 1, CHARINDEX('m', duration) - 1)
       ELSE duration END; -- If 'm' is not found, return the whole distance value
```

```
UPDATE pizza_runner.runner_orders
SET duration = REPLACE(duration, 'null', NULL)
WHERE duration = 'null';
```

```
UPDATE pizza_runner.runner_orders
SET distance = CONVERT(FLOAT, distance);
```

- Column `cancellation`: Replace 'null' and blank space with NULL
```
UPDATE pizza_runner.runner_orders
SET cancellation = REPLACE(cancellation, 'null', NULL)
WHERE cancellation = 'null';
```

```
UPDATE pizza_runner.runner_orders
SET cancellation = REPLACE(cancellation, '', NULL)
WHERE cancellation = '';
```
