# 🍕 Case Study #2 - Pizza Runner
## A. Pizza Metrics
### Data cleaning
  
  * Create a temporary table ```#customer_orders_temp``` from ```customer_orders``` table:
  	* Convert ```null``` values and ```'null'``` text values in ```exclusions``` and ```extras``` into blank ```''```.
  
  ```TSQL
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
  INTO #customer_orders_temp
  FROM customer_orders;
  
  SELECT *
  FROM #customer_orders_temp;
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
	* Cast ```pickup_time``` to DATETIME.
	* Cast ```distance``` to FLOAT.
	* Cast ```duration``` to INT.
  ```TSQL
  SELECT 
    order_id,
    runner_id,
    CAST(
    	CASE WHEN pickup_time LIKE 'null' THEN NULL ELSE pickup_time END 
	    AS DATETIME) AS pickup_time,
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
INTO #runner_orders_temp
FROM runner_orders;
  
SELECT *
FROM #runner_orders_temp;

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
### Q1. How many pizzas were ordered?
```TSQL
SELECT 
	 count(pizza_id) as total_pizza_order
FROM pizza_runner.customer_orders
;
```
| total_pizza_order |
|--------------|
| 14           |

---
### Q2. How many pizzas were ordered?
```TSQL
SELECT 
	 count(distinct order_id) as total_pizza_order
FROM pizza_runner.customer_orders;
```
| total_pizza_order |
|--------------|
| 10           |

---
### Q3. How many successful orders were delivered by each runner?
```TSQL
WITH customer_orders as (
SELECT 
	 order_id, customer_id, pizza_id,
     CASE WHEN exclusions ='null' or exclusions =''  then  null else exclusions end exclusions ,
     CASE WHEN extras='null' or extras =''  then  null else extras end extras ,
     order_time
FROM pizza_runner.customer_orders
  ),
  
  runner_orders as (
  SELECT 
	 order_id, runner_id, 
     CASE WHEN pickup_time ='null'  then  null else pickup_time end pickup_time,
     CASE WHEN distance ='null'   then  null else distance end distance ,
     CASE WHEN duration='null'   then  null else duration end duration ,
     CASE WHEN cancellation ='null' or cancellation =''  then  null else cancellation end cancellation
FROM pizza_runner.runner_orders )
SELECT 
 		runner_id,
        count(distinct cu.order_id) orders
FROM
	customer_orders cu 
    JOIN runner_orders ru 
   	ON cu.order_id = ru.order_id
WHERE
	cancellation is null
GROUP BY  runner_id
ORDER BY  runner_id
;
```
| runner_id | orders             |
|-----------|--------------------|
| 1         | 4                  |
| 2         | 3                  |
| 3         | 1                  |

---
### Q4. How many successful orders were delivered by each runner?
Approach 1: Use subquery.
```TSQL
WITH customer_orders as (
SELECT 
	 order_id, customer_id, pizza_id,
     CASE WHEN exclusions ='null' or exclusions =''  then  null else exclusions end exclusions ,
     CASE WHEN extras='null' or extras =''  then  null else extras end extras ,
     order_time
FROM pizza_runner.customer_orders
  ),
  
  runner_orders as (
  SELECT 
	 order_id, runner_id, 
     CASE WHEN pickup_time ='null'  then  null else pickup_time end pickup_time,
     CASE WHEN distance ='null'   then  null else distance end distance ,
     CASE WHEN duration='null'   then  null else duration end duration ,
     CASE WHEN cancellation ='null' or cancellation =''  then  null else cancellation end cancellation
FROM pizza_runner.runner_orders )
SELECT 
 		pizza_name,
        count( cu.pizza_id) total_pizza
FROM
	customer_orders cu 
    JOIN pizza_runner.pizza_names pi 
   	ON cu.pizza_id = pi.pizza_id
    JOIN runner_orders ru
    ON cu.order_id=ru.order_id
WHERE
	cancellation is null
GROUP BY  pizza_name
ORDER BY  pizza_name

```


| pizza_name | total_pizza  |
|------------|----------------|
| Meatlovers | 9              |
| Vegetarian | 3              |

---
### Q5. How many Vegetarian and Meatlovers were ordered by each customer?
```TSQL
SELECT 
 		customer_id	,
        pizza_name,
        count( cu.pizza_id) pizzas
FROM
	customer_orders cu 
    LEFT JOIN pizza_runner.pizza_names pi 
   	ON cu.pizza_id = pi.pizza_id
    LEFT JOIN runner_orders ru
    ON cu.order_id=ru.order_id
GROUP BY  customer_id,pizza_name
ORDER BY  customer_id
;
```

| customer_id | pizza_name |
|-------------|------------|
| 101         | Meatlovers |
| 101         | Vegetarian | 
| 102         | Vegetarian |
| 102         | Meatlovers | 
| 103         | Meatlovers |
| 103         | Meatlovers |
| 103         | Vegetarian |
| 104         | Meatlovers |
| 105         | Vegetarian |

---
### Q6. What was the maximum number of pizzas delivered in a single order?
```TSQL

SELECT max(pizza_ordered) max_order
FROM(
SELECT 
 		cu.order_id,
        count( cu.pizza_id) pizza_ordered
        
FROM
	customer_orders cu 
    LEFT JOIN pizza_runner.pizza_names pi 
   	ON cu.pizza_id = pi.pizza_id
    LEFT JOIN runner_orders ru
    ON cu.order_id=ru.order_id
GROUP BY  cu.order_id
ORDER BY  cu.order_id ) pizza;
```

|max_order |
|------------|
| 3          |

---
### Q7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```TSQL
SELECT 
 		cu.customer_id,
       sum( CASE WHEN (exclusions is null and extras is not null) or (exclusions is not null and extras is null) then 1 else 0 end  ) has_changes,
       sum( CASE WHEN exclusions is null and extras is null then 1 else 0 end  ) no_changes
FROM
	customer_orders cu 
    LEFT JOIN pizza_runner.pizza_names pi 
   	ON cu.pizza_id = pi.pizza_id
    LEFT JOIN runner_orders ru
    ON cu.order_id=ru.order_id
GROUP BY  cu.customer_id
ORDER BY  cu.customer_id
;
```
| customer_id | has_changes | no_changes  |
|-------------|------------|------------|
| 101         | 0          | 2          |
| 102         | 0          | 3          |
| 103         | 3          | 0          |
| 104         | 2          | 1          |
| 105         | 1          | 0          |

---
### Q8. How many pizzas were delivered that had both exclusions and extras?
```TSQL
SELECT
       sum( CASE WHEN exclusions is not null and extras is not null then 1 else 0 end  )  change_both
FROM
	customer_orders cu 
    LEFT JOIN pizza_runner.pizza_names pi 
   	ON cu.pizza_id = pi.pizza_id
    LEFT JOIN runner_orders ru
    ON cu.order_id=ru.order_id
    WHERE cancellation is null;
```
| change_both  |
|--------------|
| 1            |

---
### Q9. What was the total volume of pizzas ordered for each hour of the day?
```TSQL
SELECT 
        hours ,
        count(pizza_id) pizza
from customer_orders
group by  hours
order by  hours;
```
| hour | pizza  |
|-------------|---------------|
| 11          | 1             |
| 13          | 3             |
| 18          | 3             |
| 19          | 1             |
| 21          | 3             |
| 23          | 3             |

---
### Q10. What was the volume of orders for each day of the week?
```TSQL
SELECT 
        DATENAME(weekday, order_time) day_name ,
        count(pizza_id) pizza
from customer_orders
group by  day_name
order by  day_name;
```
| day_name  | pizza |
|-----------|---------------|
| Friday    | 1             |
| Saturday  | 5             |
| Thursday  | 3             |
| Wednesday | 5             |
