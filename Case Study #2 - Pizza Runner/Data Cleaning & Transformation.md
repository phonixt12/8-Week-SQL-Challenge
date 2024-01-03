
### Data cleaning
  
  * Create a temporary table ```#customer_orders_temp``` from ```customer_orders``` table:
  	* Convert ```null``` values and ```'null'``` text values in ```exclusions``` and ```extras``` into blank ```''```.
  
  ```TSQL
  with customer_orders as (
 SELECT 
    order_id,
    customer_id,
    pizza_id,
    CASE 
    	WHEN exclusions IS NULL OR exclusions LIKE 'null' THEN ''
      	ELSE exclusions 
      	END AS exclusions,
    CASE 
    	WHEN extras IS NULL OR extras LIKE 'null' THEN ''
      	ELSE extras 
      	END AS extras,
    order_time
 
  FROM pizza_runner.customer_orders )
  
  SELECT *
  FROM customer_orders
  ```
| order_id | customer_id | pizza_id | exclusions | extras | order_time               |
|----------|-------------|----------|------------|--------|--------------------------|
| 1        | 101         | 1        |            |        | 2020-01-01 18:05:02.000  |
| 2        | 101         | 1        |            |        | 2020-01-01 19:00:52.000  |
| 3        | 102         | 1        |            |        | 2020-01-02 23:51:23.000  |
| 3        | 102         | 2        |            |        | 2020-01-02 23:51:23.000  |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46.000  |
| 4        | 103         | 1        | 4          |        | 2020-01-04 13:23:46.000  |
| 4        | 103         | 2        | 4          |        | 2020-01-04 13:23:46.000  |
| 5        | 104         | 1        |            | 1      | 2020-01-08 21:00:29.000  |
| 6        | 101         | 2        |            |        | 2020-01-08 21:03:13.000  |
| 7        | 105         | 2        |            | 1      | 2020-01-08 21:20:29.000  |
| 8        | 102         | 1        |            |        | 2020-01-09 23:54:33.000  |
| 9        | 103         | 1        | 4          | 1, 5   | 2020-01-10 11:22:59.000  |
| 10       | 104         | 1        |            |        | 2020-01-11 18:34:49.000  |
  
  
  * Create a temporary table ```#runner_orders_temp``` from ```runner_orders``` table:
  	* Convert ```'null'``` text values in ```pickup_time```, ```duration``` and ```cancellation``` into ```null``` values. 
	* Cast ```pickup_time``` to DATE.
	* Cast ```distance``` to FLOAT.
	* Cast ```duration``` to INT.
  
  ```TSQL
 with runner_orders as (
SELECT 
    order_id,
    runner_id,
    CAST(
    	CASE WHEN pickup_time LIKE 'null' THEN NULL ELSE pickup_time END 
	    AS DATE) AS pickup_time,
    CAST(
    	CASE WHEN distance LIKE 'null' THEN NULL
	      WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
	      ELSE distance END
      AS FLOAT) AS distance,
    CAST(
    	CASE WHEN duration LIKE 'null' THEN NULL
	      WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
	      WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
	      WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
	      ELSE duration END
      AS INT) AS duration,
    CASE WHEN cancellation IN ('null', 'NaN', '') THEN NULL 
        ELSE cancellation
        END AS cancellation 

FROM pizza_runner.runner_orders )
  
SELECT *
FROM runner_orders;

```
| order_id | runner_id | pickup_time             | distance | duration | cancellation             |
|----------|-----------|-------------------------|----------|----------|--------------------------|
| 1        | 1         | 2020-01-01 18:15:34.000 | 20       | 32       | NULL                     |
| 2        | 1         | 2020-01-01 19:10:54.000 | 20       | 27       | NULL                     |
| 3        | 1         | 2020-01-03 00:12:37.000 | 13.4     | 20       | NULL                     |
| 4        | 2         | 2020-01-04 13:53:03.000 | 23.4     | 40       | NULL                     |
| 5        | 3         | 2020-01-08 21:10:57.000 | 10       | 15       | NULL                     |
| 6        | 3         | NULL                    | NULL     | NULL     | Restaurant Cancellation  |
| 7        | 2         | 2020-01-08 21:30:45.000 | 25       | 25       | NULL                     |
| 8        | 2         | 2020-01-10 00:15:02.000 | 23.4     | 15       | NULL                     |
| 9        | 2         | NULL                    | NULL     | NULL     | Customer Cancellation    |
  
--- 

