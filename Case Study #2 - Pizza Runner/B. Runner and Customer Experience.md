# 🍕 Case Study #2 - Pizza Runner
## B. Runner and Customer Experience
### Q1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```TSQL
SELECT 
  DATEPART(week, registration_date) AS week_period,
  COUNT(*) AS runner_count
FROM runners
GROUP BY DATEPART(week, registration_date);
```
| week_period | runner_count  |
|-------------|---------------|
| 1           | 1             |
| 2           | 2             |
| 3           | 1             |

---
### Q2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```TSQL
SELECT 
    ROUND(AVG((EXTRACT(MINUTE FROM pickup_time - order_time) + EXTRACT(SECOND FROM pickup_time - order_time)/60)::numeric), 2) AS average_time_in_minutes
FROM
    customer_orders cu
LEFT JOIN pizza_runner.pizza_names pi ON cu.pizza_id = pi.pizza_id
LEFT JOIN runner_orders ru ON cu.order_id = ru.order_id;
```
| average_time_in_minutes  |
|---------------|
| 18.59            |


---
### Q3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```TSQL
WITH pizzaPrepration AS (
  SELECT
    c.order_id, 
    c.order_time, 
    r.pickup_time,
    DATEDIFF(MINUTE, c.order_time, r.pickup_time) AS prep_time,
    COUNT(c.pizza_id) AS pizza_count
  FROM #customer_orders_temp AS c
  JOIN #runner_orders_temp AS r
    ON c.order_id = r.order_id
  WHERE r.cancellation IS NULL
  GROUP BY c.order_id, c.order_time, r.pickup_time, 
           DATEDIFF(MINUTE, c.order_time, r.pickup_time)
)

SELECT 
  pizza_count,
  AVG(prep_time) AS avg_prep_time
FROM pizzaPrepration
GROUP BY pizza_count;
```
| pizza_count | avg_prep_time  |
|-------------|----------------|
| 1           | 12.36          |
| 2           | 18.38          |
| 3           | 29.28          |

* More pizzas, longer time to prepare. 
* 2 pizzas took 6 minutes more to prepare, 3 pizza took 12 minutes more to prepare.
* On average, it took 6 * (number of pizzas - 1) minutes more to prepare the next pizza.

---
### Q4. What was the average distance travelled for each customer?
```TSQL
SELECT
  c.customer_id,
  ROUND(AVG(r.distance), 1) AS average_distance
FROM #customer_orders_temp AS c
JOIN #runner_orders_temp AS r
  ON c.order_id = r.order_id
GROUP BY c.customer_id;
```
| customer_id | average_distance  |
|-------------|-------------------|
| 101         | 20                |
| 102         | 16.7              |
| 103         | 23.4              |
| 104         | 10                |
| 105         | 25                |

---
### Q5. What was the difference between the longest and shortest delivery times for all orders?
```TSQL
SELECT MAX(duration) - MIN(duration) AS time_difference
FROM runner_orders;
```
| time_difference|
|----------------|
| 30             |

---
### Q6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```TSQL
SELECT 
  r.runner_id,
  c.order_id,
  r.distance,
  r.duration AS duration_min,
  COUNT(c.order_id) AS pizza_count, 
  ROUND(AVG(r.distance/r.duration*60), 1) AS avg_speed
FROM runner_orders r
JOIN customer_orders c
  ON r.order_id = c.order_id
WHERE r.cancellation IS NULL
GROUP BY r.runner_id, c.order_id, r.distance, r.duration;
```
| runner_id | order_id | distance | duration_min | pizza_count | avg_speed  |
|-----------|----------|----------|--------------|-------------|------------|
| 1         | 1        | 20       | 32           | 1           | 37.5       |
| 1         | 2        | 20       | 27           | 1           | 44.4       |
| 1         | 3        | 13.4     | 20           | 2           | 40.2       |
| 1         | 10       | 10       | 10           | 2           | 60         |
| 2         | 4        | 23.4     | 40           | 3           | 35.1       |
| 2         | 7        | 25       | 25           | 1           | 60         |
| 2         | 8        | 23.4     | 15           | 1           | 93.6       |
| 3         | 5        | 10       | 15           | 1           | 40         |

* Runner ```1``` had the average speed from 37.5 km/h to 60 km/h
* Runner ```2``` had the average speed from 35.1 km/h to 93.6 km/h. With the same distance (23.4 km), order ```4``` was delivered at 35.1 km/h, while order ```8``` was delivered at 93.6 km/h. There must be something wrong here!
* Runner ```3``` had the average speed at 40 km/h

---
### Q7. What is the successful delivery percentage for each runner?
```TSQL

SELECT 
  runner_id,
  COUNT(distance) AS delivered,
  COUNT(order_id) AS total,
  100 * COUNT(distance) / COUNT(order_id) AS successful
FROM #runner_orders_temp
GROUP BY runner_id;
```
| runner_id | delivered | total | successful  |
|-----------|-----------|-------|-----------------|
| 1         | 4         | 4     | 100             |
| 2         | 3         | 4     | 75              |
| 3         | 1         | 2     | 50              |

