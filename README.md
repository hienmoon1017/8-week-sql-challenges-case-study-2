# 8 Week SQL Challenges by Danny | Case Study #1 - Pizza Runner
### ERD for Database
![image](https://github.com/user-attachments/assets/baab67e2-5068-4808-8b52-036d4650077e)

## Result of Questions
### Data Preparation
```sql
-- for runner_orders table
ALTER TABLE pizza_runner.runner_orders
ADD COLUMN is_cancellation boolean; 

UPDATE pizza_runner.runner_orders
SET is_cancellation = case when pickup_time = 'null' then True  -- True represents no cancellation
		 					else False
							end; 
```
_Result:_

![image](https://github.com/user-attachments/assets/1b96f3f8-8fbc-4b17-8084-ed293b536a7e)

```sql
-- for customer_orders table
ALTER TABLE pizza_runner.customer_orders
ADD COLUMN is_changed_order boolean,
ADD COLUMN order_date_extracted date,
ADD COLUMN order_hour_extracted TIME;

UPDATE pizza_runner.customer_orders
SET order_date_extracted = substring(order_time,1,10)::date,
	order_hour_extracted = substring(order_time,11)::TIME;

-- clean null and NaN values
UPDATE pizza_runner.customer_orders
SET exclusions = case when exclusions isnull or exclusions = 'NaN' or exclusions ='null' then ' '
						else exclusions 
						end,
	extras = case when extras isnull or extras = 'NaN' or extras = 'null' then ' '
					else extras  
					end;
```
_Result:_

![image](https://github.com/user-attachments/assets/b8ca1339-a357-46ef-b20e-7b827c87b2d1)


### Q1. How many pizzas were ordered?
```sql
SELECT distinct count(*) as total_ordered_pizzas
FROM pizza_runner.customer_orders
;
```
_Result:_

![image](https://github.com/user-attachments/assets/eb1c6e8b-ed56-4b44-ab09-5755a6e35add)

### Q2. How many unique customer orders were made?
```sql
SELECT count(distinct customer_id) as unique_ordered_customers
FROM pizza_runner.customer_orders
;
```
_Result:_

![image](https://github.com/user-attachments/assets/c8d6993c-3235-4b19-8ef6-53a53931099c)

### Q3. How many successful orders were delivered by each runner?
```sql
SELECT runner_id
,count(*) as total_successful_orders
FROM pizza_runner.runner_orders
WHERE is_cancellation = false
GROUP BY 1
ORDER BY 1
;
```
_Result:_

![image](https://github.com/user-attachments/assets/48617cb7-5547-4fdb-98c3-ed8f693d1752)

### Q4. How many of each type of pizza was delivered?
```sql
SELECT co.pizza_id
,count(*) as total_delivered_pizza
FROM pizza_runner.runner_orders ro
LEFT JOIN pizza_runner.customer_orders co on co.order_id = ro.order_id
WHERE is_cancellation = false
GROUP BY 1
ORDER BY 1
;
```
_Result:_

![image](https://github.com/user-attachments/assets/2abf3e4a-2925-4ce2-b82b-53ebbb39aeb6)

### Q5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT  co_dis.customer_id
,pz.pizza_name
,count(*) as total_pizzas
FROM
(
	SELECT distinct *
	FROM pizza_runner.customer_orders
) co_dis
JOIN pizza_runner.pizza_names pz on pz.pizza_id = co_dis.pizza_id
GROUP BY 1,2
ORDER BY 1,2
;
```
_Result:_

![image](https://github.com/user-attachments/assets/6d244643-e84c-442e-8a9c-e24e92d28277)

### Q6. What was the maximum number of pizzas delivered in a single order?
```sql
WITH delivered_pizzas as
(
	SELECT ro.order_id
	,count(*) as number_of_delievered_pizzas
	FROM pizza_runner.runner_orders ro
	JOIN pizza_runner.customer_orders co on ro.order_id = co.order_id
	WHERE ro.is_cancellation = false
	GROUP BY 1
)
SELECT max(number_of_delievered_pizzas) as max_pizzas_delivered
FROM delivered_pizzas dp
;
```
_Result:_

![image](https://github.com/user-attachments/assets/4b82c28d-9266-472c-bd8b-c07a874c6dd7)

### Q7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
WITH order_change as
(
	SELECT customer_id, order_id
	,case when exclusions isnull or extras isnull then 'no change'
			else 'at least 1 change'
			end as changed_order
	FROM 
	(
		SELECT distinct *
		FROM pizza_runner.customer_orders
	)
)
SELECT oc.customer_id, oc.changed_order
,count(*)
FROM order_change oc
JOIN pizza_runner.runner_orders ro on ro.order_id = oc.order_id
WHERE ro.is_cancellation = false
GROUP BY 1,2
ORDER BY 1,2
;
```
_Result:_

![image](https://github.com/user-attachments/assets/ee457de3-e784-4183-83b2-9e3107425aad)

### Q8. How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT count(*) as total_delivered_pizzas_both_exclusions_and_extras
FROM pizza_runner.customer_orders co
JOIN pizza_runner.runner_orders ro on ro.order_id = co.order_id
WHERE ro.is_cancellation = false
	and co.exclusions != ' ' and co.extras != ' '
;
```
_Result:_

![image](https://github.com/user-attachments/assets/cc9aa1ce-5cf9-4860-b913-0ebca6aeb2a9)

### Q9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT extract(hour from order_hour_extracted) as order_hour
,count(*) as total_order
FROM
(
	SELECT distinct *
	FROM pizza_runner.customer_orders
)
GROUP BY 1
ORDER BY 1
;
```
_Result:_

![image](https://github.com/user-attachments/assets/6affe042-ff83-4be5-a821-625f86fc4b98)

### Q10. What was the volume of orders for each day of the week?
```sql
SELECT to_char(order_date_extracted, 'Day') as day_of_week
,count(*)
FROM
(
	SELECT distinct *
	FROM pizza_runner.customer_orders
)
GROUP BY 1
ORDER BY 1
;
```
_Result:_

![image](https://github.com/user-attachments/assets/b4346ce7-d9c2-4d8a-9da4-cdc30f7f9d8a)

Thank you for stopping by, and I'm pleased to connect with you, my new friend!

Please do not forget to FOLLOW and star ⭐ the repository if you find it valuable.

I wish you a day filled with happiness and energy!

Warm regards,

Hien Moon
