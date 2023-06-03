# 🍕 Case Study #2 Pizza Runner

<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">
## 📚 Table of Contents
- [Business Task](#business-task)
- [Entity Relationship Diagram](#entity-relationship-diagram)
- Solution
  - [Data Cleaning and Transformation](#-data-cleaning--transformation)
  - [A. Pizza Metrics](#a-pizza-metrics)
  - [B. Runner and Customer Experience](#b-runner-and-customer-experience)
  - [C. Ingredient Optimisation](#c-ingredient-optimisation)
  - [D. Pricing and Ratings](#d-pricing-and-ratings)

## Business Task
Danny is expanding his new Pizza Empire and at the same time, he wants to Uberize it, so Pizza Runner was launched!

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers. 

## Entity Relationship Diagram

![Pizza Runner](https://github.com/katiehuangx/8-Week-SQL-Challenge/assets/81607668/78099a4e-4d0e-421f-a560-b72e4321f530)

## 🧼 Data Cleaning & Transformation

Looking at the `customer_orders` table below, we can see that there are
- In the `exclusions` column, there are missing/ blank spaces ' ' and null values. 
- In the `extras` column, there are missing/ blank spaces ' ' and null values.

![image](https://github.com/MounaBaq/8_Week_Sql_Challenge/assets/102436799/12210920-c547-4261-8966-70b23c17e0dd)

Our course of action to clean the table:
- Create a temporary table with all the columns
- Remove null values in `exlusions` and `extras` columns and replace with blank space ' '.

````sql
  select order_id,customer_id,pizza_id,order_time,
  case 
  when exclusions is null or exclusions like 'null' then ' ' else exclusions
  END AS exclusions,
  iif((extras is NULL) or (extras like 'null'), ' ', extras) as extras
  INTO #customer_orders -- create TEMP TABLE
  FROM customer_orders
````

This is how the clean `customers_orders_temp` table looks like and we will use this table to run all our queries.

![image](https://github.com/MounaBaq/8_Week_Sql_Challenge/assets/102436799/a0537460-8143-454d-8ca7-e94fc1e38e99)

****

### 🔨 Table: runner_orders

Looking at the `runner_orders` table below, we can see that there are
- In the `exclusions` column, there are missing/ blank spaces ' ' and null values. 
- In the `extras` column, there are missing/ blank spaces ' ' and null values

![image](https://github.com/MounaBaq/8_Week_Sql_Challenge/assets/102436799/8e401ffd-bc50-4387-9958-ffc2be93ad20)

Our course of action to clean the table:
- In `pickup_time` column, remove nulls and replace with blank space ' '.
- In `distance` column, remove "km" and nulls and replace with blank space ' '.
- In `duration` column, remove "minutes", "minute" and nulls and replace with blank space ' '.
- In `cancellation` column, remove NULL and null and and replace with blank space ' '.

````sql
drop table if  exists #runner_orders
select order_id,runner_id,
Case when distance like 'null'then ' ' 
     when distance like '%km' then trim ('km' from distance)
     else distance end as distance,
Case When cancellation is null then ' '  
     When cancellation like'null' then ' '
     else cancellation end as cancellation,
Case when pickup_time like 'null' then ' '
     else pickup_time end as pickup_time,
Case when duration is null then ' ' 
     when duration like 'null' then ' ' 
     When duration like '%minutes' then trim ('%minutes' from duration)
	   When duration like '%minute' then trim ('%minute' from duration)
     When duration like '%mins' then trim ('%mins' from duration)
     else duration end as duration
into #runner_orders
from runner_orders
````

Then, we alter the `pickup_time`, `distance` and `duration` columns to the correct data type.

````sql
Alter table #runner_orders
Alter column pickup_time DATETIME
Alter table #runner_orders
Alter column distance float
Alter table #runner_orders
Alter column duration int;
````

This is how the clean `runner_orders_temp` table looks like and we will use this table to run all our queries.

![image](https://github.com/MounaBaq/8_Week_Sql_Challenge/assets/102436799/8b6f05b7-482f-42cb-a3b7-16cc84aeb084)

## Solution

## A. Pizza Metrics

### 1. How many pizzas were ordered?

````sql
select count(order_id) As Pizza_order_count
 from #customer_orders
````

**Answer:**

![image](https://github.com/MounaBaq/8_Week_Sql_Challenge/assets/102436799/6d18a2da-ac7a-4f73-8913-182d9257e707)

- Total of 14 pizzas were ordered.

### 2. How many unique customer orders were made?

````sql
select  count(distinct order_id) As Pizza_order_unicount
 from #customer_orders
````

**Answer:**

![image](https://github.com/MounaBaq/8_Week_Sql_Challenge/assets/102436799/1a6daa6e-c615-47e1-8375-b1b531741431)

- There are 10 unique customer orders.

### 3. How many successful orders were delivered by each runner?

````sql
select runner_id,count ( order_id) as delivered
 from #runner_orders
 where cancellation=' '
 group by runner_id,cancellation
````

**Answer:**

![image](https://github.com/MounaBaq/8_Week_Sql_Challenge/assets/102436799/9a3e9d2c-5c17-40ff-b308-dd5c8c8b7083)

- Runner 1 has 4 successful delivered orders.
- Runner 2 has 3 successful delivered orders.
- Runner 3 has 1 successful delivered order.

### 4. How many of each type of pizza was delivered?

````sql
 select count( n.pizza_id) as count_pizza,cast(n.pizza_name as nvarchar(100)) as pizza_name
 from #customer_orders o
 join pizza_names n
 on o.pizza_id=n.pizza_id
 join #runner_orders ro
 on o.order_id=ro.order_id
 where ro.cancellation = ' '
 group by cast(n.pizza_name as nvarchar(100))
````

**Answer:**

![image](https://github.com/MounaBaq/8_Week_Sql_Challenge/assets/102436799/957b204b-de63-4696-b8e9-9aab68f206f2)

- There are 9 delivered Meatlovers pizzas and 3 Vegetarian pizzas.

### 5. How many Vegetarian and Meatlovers were ordered by each customer?**

````sql
DROP TABLE IF EXISTS pizza_names;
CREATE TABLE pizza_names (
  "pizza_id" INTEGER,
  "pizza_name" nvarchar(100)
);
INSERT INTO pizza_names
  ("pizza_id", "pizza_name")
VALUES
  (1, 'Meatlovers'),
  (2, 'Vegetarian');
````
````sql
select co.customer_id,count(pn.pizza_name) order_count,(pn.pizza_name) pizza_name
 from #customer_orders co
 join pizza_names pn
 on co.pizza_id= pn.pizza_id
 group by  co.customer_id ,pn.pizza_name
 order by co.customer_id
````

**Answer:**

![image](https://github.com/MounaBaq/8_Week_Sql_Challenge/assets/102436799/db6d13f3-589b-4ac9-b9c6-719279d6b074)

- Customer 101 ordered 2 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 102 ordered 2 Meatlovers pizzas and 2 Vegetarian pizzas.
- Customer 103 ordered 3 Meatlovers pizzas and 1 Vegetarian pizza.
- Customer 104 ordered 3 Meatlovers pizza.
- Customer 105 ordered 1 Vegetarian pizza.

### 6. What was the maximum number of pizzas delivered in a single order?

````sql
select max(z.countpizza ) as Max_pizza_ordered
from 
(select co.order_id,count(co.pizza_id) countpizza
from #customer_orders co
join #runner_orders ro
on co.order_id= ro.order_id
where ro.cancellation =' ' 
group by co.order_id) as z
````

**Answer:**

![image](https://github.com/MounaBaq/8_Week_Sql_Challenge/assets/102436799/6d0b4b19-f6d9-46a5-a051-29350d02b1c6)

- Maximum number of pizza delivered in a single order is 3 pizzas.

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

````sql
select co.customer_id,
SUM(case 
when  exclusions = ' ' and extras = ' '
then 1 else 0 end) as nochange,
SUM(case 
when exclusions <> ' ' or extras <> ' ' 
then 1 else 0 end) as atleast1change
from #customer_orders co
join #runner_orders ro
on co.order_id= ro.order_id
where ro.cancellation =' ' 
group by co.customer_id
````

**Answer:**

![image](https://github.com/MounaBaq/8_Week_Sql_Challenge/assets/102436799/0fcd8463-40cf-43bb-abf2-8a1d31e09cee)

- Customer 101 and 102 likes his/her pizzas per the original recipe.
- Customer 103, 104 and 105 have their own preference for pizza topping and requested at least 1 change (extra or exclusion topping) on their pizza.

### 8. How many pizzas were delivered that had both exclusions and extras?

````sql
select count(t.customer_id) as countpizzas
from (
select co.customer_id,
SUM(case 
when  exclusions = ' ' and extras = ' '
then 1 else 0 end) as nochange,
SUM(case 
when exclusions <> ' ' or extras <> ' ' 
then 1 else 0 end) as atleast1change
from #customer_orders co
join #runner_orders ro
on co.order_id= ro.order_id
where ro.cancellation =' '
group by co.customer_id
) as t
where t.nochange > 0 and t.atleast1change > 0
````

**Answer:**

![image](https://user-images.githubusercontent.com/81607668/129738278-dd3e7056-309d-42fc-a5e3-00f7b5d4609e.png)

- Only 1 pizza delivered that had both extra and exclusion topping. That’s one fussy customer!
