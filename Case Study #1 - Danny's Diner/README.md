## Case Study #1: Danny's Diner
<p align="center">
<img src="https://github.com/hatrang12/8weeksqlchallenge.com/blob/main/1.png" align="center" width="400" height="400" >


## Table of Content
1. [Business Task](#Business-Task)
2. [Entity Relationship Diagram](#entity-relationship-diagram)
3. [Case Study Questions](#Question-and-Solution)

If you are interest in this case study you can find it in this [link](https://8weeksqlchallenge.com/case-study-1/)
## Business Task 
Danny wants to analyze customer data to gain insights into their visiting patterns, spending habits, and favorite menu items. He aims to establish a deeper connection with his customers to enhance their experience. Specifically, he plans to use these insights to decide whether to expand the current customer loyalty program. To facilitate the analysis, Danny needs help generating basic datasets that his team can easily inspect without using SQL directly. Despite privacy concerns, he has provided a sample of customer data for the purpose of creating fully functioning SQL queries to address his questions

## Entity Relationship Diagram
![entity_relationship_diagram](https://github.com/hatrang12/8weeksqlchallenge.com/assets/107136018/b2325eb0-4927-495d-9530-82e32dc05af5)

## Question and Solution
#### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT 
	  customer_id,
      sum(price) as total_amount    
FROM
	 dannys_diner.sales sa
JOIN
	 dannys_diner.menu me
ON 
	sa.product_id=me.product_i
GROUP BY customer_id
ORDER BY
	customer_id;
````
#### Steps:
- JOIN two table dannys_diner.sales and dannys_diner.menu by using product_id.
- Use SUM to calculate the total sales contributed by each customer.
- Group the aggregated results by sales.customer_id 
- Order it by customer_id

|  customer_id | total_pay  |
|---|---|
|A	|76|
|B	|74|
|C	|36|


#### Result
- Customer A : $76.
- Customer B : $74.
- Customer C : $36.
#### 2. How many days has each customer visited the restaurant?

````sql
SELECT 
	  customer_id,
      count(distinct order_date) visited_days
FROM
	 dannys_diner.sales sa
  
GROUP BY customer_id;
````
#### Steps:
- The order_date column is not unique value so we need to count distinct in this question, utilize **count(distinct `order_date`)**.
- It will avoid duplicate counting of days.
  
|  customer_id | visit_count  |
|---|---|
|A	|4|
|B	|6|
|C	|2|

#### Result 
- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

#### 3. What was the first item from the menu purchased by each customer?

 ````sql

with pre_data as (
  
SELECT 
	  customer_id,
  	  product_name,
  	  order_date,
      dense_rank() over (partition by customer_id order by order_date) ranks 
  FROM
	 dannys_diner.sales sa
JOIN
	 dannys_diner.menu me
ON 
	sa.product_id=me.product_id
 ) 
SELECT 
	  customer_id,
      product_name
FROM
	  pre_data   
WHERE
	   ranks=1
GROUP BY customer_id,product_name

````
#### Steps:
- Create a Common Table Expression (CTE) named `pre_data`. Within the CTE, create a new column `ranks` and calculate the row number using **dense_rank()** window function. in the the **partition by** clause divides the data by `customer_id`, and the **order by** by `order_date` in ascending.
- In the outer query, choose a suitable value by filltering with the **where** clause to retrieve  rows where the rank column equals 1, which represents the first row within each `customer_id` partition.
- Use the **group by** to group the result by `customer_id` and `product_name`

| customer_id | order_date | product_name |
|---|------------|-------|
| A | 2021-01-01 | curry |
| A | 2021-01-01 | sushi |
| B | 2021-01-01 | curry |
| C | 2021-01-01 | ramen |


#### Result 
- Customer A placed an order for both curry and sushi simultaneously, making them the first items in the order.
- Customer B's first order is curry.
- Customer C's first order is ramen


#### 4.What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql

with pre_data as
(SELECT 
      product_name,
      count(sa.product_id) buying_times
      
FROM
	 dannys_diner.sales sa
 JOIN
	 dannys_diner.menu me
ON 
	sa.product_id=me.product_id
GROUP BY product_name
)


SELECT 
	  product_name,
      max(buying_times) times
FROM 
	  pre_data
 GROUP BY product_name
 LIMIT 1
````

#### Steps:
- Use a **count** function on the `product_id` column and **order by** the result in descending order using `times` field.
- Apply the **limit** 1 clause to filter and retrieve the highest number of purchased items


| product_name | times      |
|--------------|------------|
| ramen        | 8          |

#### Result 
The most popular product is ramen which is 8 times

#### 5. Which item was the most popular for each customer?

````sql
with pre_data as
(SELECT 
	  customer_id,
      product_name,
      count(sa.product_id) buying_times,
      dense_rank() over (partition by customer_id order by  count(sa.product_id) desc) ranks
FROM
	 dannys_diner.sales sa
 JOIN
	 dannys_diner.menu me
ON 
	sa.product_id=me.product_id
GROUP BY customer_id,product_name
)
SELECT 
	  customer_id,
	  product_name,
      buying_times
FROM 
	  pre_data
 WHERE ranks = 1
````
#### Steps:
- Create a CTE named `pre_data` and within the CTE, join the `menu` table and `sales` table by using the `product_id` .
  
- Group results by `customer_id` and `product_name` and calculate the count of `product_id` .
  
- Use the **dense_rank()** window function to calculate the ranking of each `sales.customer_id` partition based on the  **count(`sales.customer_id`)** in descending order.
  
- In the outer query, select the columns by filltering with in the **where** clause to retrieve only the rows where the rank column equals 1, representing the rows with the highest order count for each customer

| customer_id  |product_name|
|--------------|------------|
| A            | curry      |
| A            | sushi      |
| B	       | curry      |
| C	       | ramen      |


#### Result 
- Customer A and C's favourite item is ramen.
- Customer B like all items on the menu

#### 6. Which item was purchased first by the customer after they became a member?

````sql
with pre_data as (
SELECT
  	sa.customer_id,
    product_name,
    dense_rank() over (partition by sa.customer_id order by order_date) ranks
FROM 
	 dannys_diner.sales sa
JOIN 
	 dannys_diner.members mem
ON 
	 sa.customer_id = mem.customer_id
JOIN 
	 dannys_diner.menu me 
ON 
 	 sa.product_id = me.product_id
WHERE 
	 order_date > join_date
 )
SELECT
	  customer_id,
	  product_name
FROM
	  pre_data
WHERE 
	  ranks =1
````

#### Steps:
- Create a Common Table Expression (CTE) named joined_as_member. In this CTE, select the necessary columns and employ the row_number() window function. Partition the data by members.customer_id, and order the rows within each members.customer_id partition by sales.order_date.
  
-Join the tables dannys_diner.members and dannys_diner.sales on the customer_id column. Include a condition to only include sales that occurred after the member's join_date (sales.order_date > members.join_date).

-In the outer query, join the joined_as_member CTE with the dannys_diner.menu on the product_id column.


-In the WHERE clause, filter the results to include only rows where the row_num column equals 1, indicating the first row within each customer_id partition.
- Order the result set by customer_id in ascending order
  
| customer_id | product_name |
|-------------|--------------|
| A           | curry        | 
| B           | sushi        | 


#### Result
- Customer A's first order as a member is ramen.
- Customer B's first order as a member is sushi

#### 7. Which item was purchased just before the customer became a member?

````sql
with pre_data as (
SELECT
  	sa.customer_id,
    product_name,
    row_number() over (partition by sa.customer_id order by order_date desc) ranks
FROM 
	 dannys_diner.sales sa
JOIN 
	 dannys_diner.members mem
ON 
	 sa.customer_id = mem.customer_id
JOIN 
	 dannys_diner.menu me 
ON 
 	 sa.product_id = me.product_id
WHERE 
	 order_date < join_date
 )
SELECT 
	  customer_id,
	  product_name
FROM
	  pre_data   
WHERE 
	  ranks =1
````

#### Steps:
-Establish a Common Table Expression (CTE) named purchased_prior_member.

-Within the CTE, carefully select the pertinent columns and employ the ROW_NUMBER() window function to determine the ranking. This ranking is computed based on the order dates of sales, arranged in descending order within each customer's group.

-Perform a join between the dannys_diner.members table and the dannys_diner.sales table, utilizing the customer_id column. Ensure inclusion of only those sales that transpired prior to the customer's membership initiation (sales.order_date < members.join_date).

-Employ the purchased_prior_member CTE in conjunction with the dannys_diner.menu table, using the product_id column for the join operation.

-Apply a filtering condition to the result set, retaining only the rows where the rank is 1. This signifies the earliest purchase made by each customer before their enrollment as a member.

-Arrange the final result in ascending order based on the customer_id

| customer_id | product_name |
|-------------|--------------|
| A           | sushi        | 
| B           | sushi        | 

#### Result 
- Both customers' last order before becoming members are sushi.
  
#### 8.What is the total items and amount spent for each member before they became a member?
````sql
with pre_data as (
SELECT
  	sa.customer_id,
    count(sa.product_id) total_items,
	sum(price) total_amount
FROM 
	 dannys_diner.sales sa
JOIN 
	 dannys_diner.members mem
ON 
	 sa.customer_id = mem.customer_id
JOIN 
	 dannys_diner.menu me 
ON 
 	 sa.product_id = me.product_id
WHERE 
	 order_date < join_date
  
GROUP BY sa.customer_id
 )
SELECT 
	  customer_id,
	  total_items,
      total_amount
FROM
	  pre_data
ORDER BY customer_id
````

#### Steps
--Column Selection and Calculation:
--Choose the columns sales.customer_id.
--Compute the count of sales.product_id as total_items for each customer.
--Determine the sum of menu.price as total_sales.
--Table Joining:

--Utilize the dannys_diner.sales table.
--Join it with the dannys_diner.members table on the customer_id column.
--Ensure that sales.order_date precedes members.join_date (sales.order_date < members.join_date).
--Additional Join Operation:

--Further join the result with the dannys_diner.menu table on the product_id column.
--Grouping:

--Group the results by sales.customer_id.
--Sorting:

--Order the result by sales.customer_id in ascending order

| customer_id | total_items | total_amount|
|-------------|-------------|-------------|
| A           | 2           | 25          |
| B           | 3           | 40          |


#### Result 
Before becoming members,
- Customer A spent $25 on 2 items.
- Customer B spent $40 on 3 items

  
9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
````sql
with pre_data as (
SELECT
  	sa.customer_id,
 	 product_name,
	sum(price) total_amount
FROM 
	 dannys_diner.sales sa
LEFT JOIN 
	 dannys_diner.members mem
ON 
	 sa.customer_id = mem.customer_id
JOIN 
	 dannys_diner.menu me 
ON 
 	 sa.product_id = me.product_id
GROUP BY sa.customer_id,me.product_name
 )

SELECT   
      customer_id,
	  sum (case when product_name = 'sushi' then total_amount *20
      else
     total_amount *10 END) points
FROM
	  pre_data
GROUP BY customer_id    
ORDER BY customer_id
````

--Point Calculation Logic:

--Establish the point conversion rate: Each $1 spent equals 10 points.
--For product_id 1 (sushi), apply a special condition: Each $1 spent is equivalent to 20 points.
--Calculation Using CASE Statement:

--Implement a conditional CASE statement for the calculation:
--If product_id equals 1, multiply every $1 by 20 points.
--For other product_id values, multiply $1 by the standard rate of 10 points.

--Total Points Calculation:
--Calculate the total points for each customer based on the adjusted point values

| customer_id | total_points |
|-------------|--------------|
| A           | 860          |
| B           | 940          |
| C           | 360          |


#### Result 
- Total points for Customer A is $860.
- Total points for Customer B is $940.
- Total points for Customer C is $360.
- 
10.In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
````sql
WITH programDates AS (
  SELECT 
    customer_id, 
    join_date,
    DATEADD(d, 6, join_date) AS valid_date, 
    EOMONTH('2021-01-01') AS last_date
  FROM members
)

SELECT 
  p.customer_id,
  SUM(CASE 
      	WHEN s.order_date BETWEEN p.join_date AND p.valid_date THEN m.price*20
      	WHEN m.product_name = 'sushi' THEN m.price*20
      ELSE m.price*10 END) AS total_points
FROM sales s
JOIN programDates p 
  ON s.customer_id = p.customer_id
JOIN menu m 
  ON s.product_id = m.product_id
WHERE s.order_date <= last_date
GROUP BY p.customer_id;
````
| customer_id | total_points |
|-------------|--------------|
| A           | 1370         |
| B           | 820          |          

### Join All The Things 
**Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)**

````sql
SELECT
  	sa.customer_id,
  
  	order_date,
    product_name,
    price,
    CASE WHEN sa.customer_id=mem.customer_id and  order_date > join_date THEN 'Y'
    ELSE 'N' END as member
FROM 
	 dannys_diner.sales sa
LEFT JOIN 
	 dannys_diner.members mem
ON 
	 sa.customer_id = mem.customer_id
JOIN 
	 dannys_diner.menu me 
ON 
 	 sa.product_id = me.product_id

````
| customer_id | order_date | product_name | price | member |
|-------------|------------|--------------|-------|--------|
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |


### Rank All The Things
**Danny also requires about the ranking of customer products, but he does not want to rank the non-membe so he expects null ranking values for the records when customers are not memeber in the loyalty program.**


````sql
WITH pre_data AS (
  SELECT
    sa.customer_id,
    order_date,
    product_name,
    price,
    CASE WHEN sa.customer_id = mem.customer_id AND order_date > join_date THEN 'Y'
         ELSE 'N' END AS member
  FROM 
    dannys_diner.sales sa
  LEFT JOIN 
    dannys_diner.members mem ON sa.customer_id = mem.customer_id
  JOIN 
    dannys_diner.menu me ON sa.product_id = me.product_id
)
SELECT *,
  CASE WHEN member = 'Y' 
    THEN DENSE_RANK() OVER(PARTITION BY customer_id, member ORDER BY order_date)
    ELSE null END AS ranking
FROM pre_data;

````
| customer_id | order_date | product_name | price | member | ranking |
|-------------|------------|--------------|-------|--------|---------|
| A           | 2021-01-01 | sushi        | 10    | N      | NULL    |
| A           | 2021-01-01 | curry        | 15    | N      | NULL    |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      | NULL    |
| B           | 2021-01-02 | curry        | 15    | N      | NULL    |
| B           | 2021-01-04 | sushi        | 10    | N      | NULL    |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      | NULL    |
| C           | 2021-01-01 | ramen        | 12    | N      | NULL    |
| C           | 2021-01-07 | ramen        | 12    | N      | NULL    |

