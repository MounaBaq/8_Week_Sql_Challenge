# 8_Week_Sql_Challenge
This repository represents the solutions for the 8 case studies in #8WeekSQLChallenge!  Thank you  @DataWithDanny for the excellent SQL case studies! 
## 📚 Table of Contents
- [Case Study #1: Danny's Diner](#case-study-1-dannys-diner)
- [Case Study #2: Pizza Runner](#case-study-2-pizza-runner)
- [Case Study #3: Foodie-Fi](#case-study-3-foodie-fi)
- [Case Study #4: Data Bank](#case-study-4-data-bank)
- [Case Study #5: Data Mart](#case-study-5-data-mart)
- [Case Study #6: Clique Bait](#case-study-6-clique-bait)
- [Case Study #8: Fresh Segments](#case-study-8-fresh-segments)

***

## Case Study #1: Danny's Diner 
<img src="https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png" alt="Image" width="500" height="520">

View the case study [here](https://8weeksqlchallenge.com/case-study-1/) and my **solution** [here](https://github.com/katiehuangx/8-Week-SQL-Challenge/tree/main/Case%20Study%20%231%20-%20Danny's%20Diner).

### Business Task
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. 

### Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)

### Solution

***
## 1. What is the total amount each customer spent at the restaurant?

````sql
Select s.customer_id,Sum(m.price) TotalAmountSpent
from dbo.sales as S
Join dbo.menu as M
on m.product_id=s.product_id
group by s.customer_id
````
#### Steps:
- Use **JOIN** to merge `dbo.sales` and `dbo.menu` tables as `sales.customer_id` and `menu.price` are from both tables.
- Use **SUM** to calculate the total sales contributed by each customer.
- Group the aggregated results by `s.customer_id`. 

#### Answer:
| customer_id | total_sales |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.

***

### 2. How many days has each customer visited the restaurant?

````sql
Select customer_id,count(distinct(order_date)) as Visit_count
From dbo.sales
group by customer_id
````

#### Steps:
- To determine the unique number of visits for each customer, utilize **COUNT(DISTINCT `order_date`)**.
- It's important to apply the **DISTINCT** keyword while calculating the visit count to avoid duplicate counting of days. For instance, if Customer A visited the restaurant twice on '2021–01–07', counting without **DISTINCT** would result in 2 days instead of the accurate count of 1 day.

#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

***
### 3. What was the first item from the menu purchased by each customer?

````sql
With sales_ordered_cte
as
(
select customer_id,order_date,product_name,
 dense_rank() over (partition by s.customer_id order by s.order_date) as rank
from dbo.sales s
join dbo.menu m
on m.product_id=s.product_id
)

Select customer_id,product_name
from sales_ordered_cte
where rank=1
group by customer_id,product_name
````

#### Steps:
- Create a Common Table Expression (CTE) named `sales_ordered_cte`. Within the CTE, create a new column `rank` and calculate the row number using **DENSE_RANK()** window function. The **PARTITION BY** clause divides the data by `customer_id`, and the **ORDER BY** clause orders the rows within each partition by `order_date`.
- In the outer query, select the appropriate columns and apply a filter in the **WHERE** clause to retrieve only the rows where the rank column equals 1, which represents the first row within each `customer_id` partition.
- Use the GROUP BY clause to group the result by `customer_id` and `product_name`.

#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

- Customer A placed an order for both curry and sushi simultaneously, making them the first items in the order.
- Customer B's first order is curry.
- Customer C's first order is ramen.

***
### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
Select  TOP 1 count(s.product_id) as most_purchased, m.product_name
from dbo.sales s
join dbo.menu m
on s.product_id=m.product_id
group by m.product_name,s.product_id
order by most_purchased desc;
````

#### Steps:
- Perform a **COUNT** aggregation on the `product_id` column and **ORDER BY** the result in descending order using `most_purchased` field.
- Apply the **TOP 1**  clause to filter and retrieve the highest number of purchased items.

#### Answer:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |


- Ramen is the most purchased item on the menu as it was purchased over 8 times.

***
### 5. Which item was the most popular for each customer?

````sql
With fav_item_cte 
as
(
Select  m.product_name,s.customer_id,count(s.product_id) as countitem,
dense_rank() over(partition by s.customer_id order by count (s.customer_id) desc ) as rank
from dbo.menu m
join dbo.sales s
on m.product_id=s.product_id
group by s.customer_id,m.product_name
)
select customer_id,product_name,countitem
from fav_item_cte
where rank=1
````

*Each user may have more than 1 favourite item.*

#### Steps:
- Create a CTE named `fav_item_cte` and within the CTE, join the `menu` table and `sales` table using the `product_id` column.
- Group results by `sales.customer_id` and `menu.product_name` and calculate the count of `menu.product_id` occurrences for each group. 
- Utilize the **DENSE_RANK()** window function to calculate the ranking of each `sales.customer_id` partition based on the count of orders **COUNT(`sales.customer_id`)** in descending order.
- In the outer query, select the appropriate columns and apply a filter in the **WHERE** clause to retrieve only the rows where the rank column equals 1, representing the rows with the highest order count for each customer.

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customer A and customer C both enjoy Ramen very much, 
- While customer B has a big appetite and loves every item on the menu 

***
