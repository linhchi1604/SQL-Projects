The data needs cleaning since there are inconsistancy and missing values.

# Table 2: customer_orders
Problem: blank space, 'null' value in column exclusions and extras
- Update 'null' values as NULL
UPDATE pizza_runner.customer_orders
SET exclusions = REPLACE(exclusions, 'null', NULL)
WHERE exclusions = 'null';

UPDATE pizza_runner.customer_orders
SET extras = REPLACE(extras, 'null', NULL)
WHERE extras = 'null';

- Replace blank space with NULL
UPDATE pizza_runner.customer_orders
SET exclusions = REPLACE(exclusions, '', NULL)
WHERE exclusions = '';

UPDATE pizza_runner.customer_orders
SET extras = REPLACE(extras, '', NULL)
WHERE extras = '';
