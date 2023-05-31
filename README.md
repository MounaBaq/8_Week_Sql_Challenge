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

View the case study [here](https://8weeksqlchallenge.com/case-study-1/) and my **solution** [here](https://github.com/MounaBaq/8_Week_Sql_Challenge#case-study-1-dannys-diner).

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
### 6. Which item was purchased first by the customer after they became a member?

```sql
With members_sales_cte
as
(
Select s.customer_id,s.order_date,m.join_date,s.product_id,
dense_rank() over (partition by s.customer_id order by s.product_id) as rank
from dbo.sales s
join dbo.members m
on s.customer_id= m.customer_id
where s.order_date>=m.join_date
)

select s.customer_id,s.order_date,s.product_id,m2.product_name
from members_sales_cte as s
join dbo.menu m2
on s.product_id=m2.product_id
where rank=1

```

#### Steps:
- In this CTE, we filter order_date to be on or after their join_date and then rank the product_id by the order_date.
- Next, we filter the table by rank = 1 to show first item purchased by customer.

#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | ramen        |
| B           | sushi        |

- Customer A's first order as a member is ramen.
- Customer B's first order as a member is sushi.

***
### 7. Which item was purchased just before the customer became a member?

````sql
With prior_mem_pursh_cte
as
(
Select s.customer_id,s.order_date,m.join_date,s.product_id,
dense_rank() over (partition by s.customer_id order by s.order_date desc) as rank
from dbo.sales s
join dbo.members m
on s.customer_id= m.customer_id
where s.order_date < m.join_date
)

select s.customer_id,s.order_date,s.product_id,m2.product_name
from prior_mem_pursh_cte as p
join dbo.menu m2
on s.product_id = m2.product_id
where rank=1
````

#### Steps:
- Create new column rank by partitioning customer_id by DESC order_date to find out the order_date just before the customer became member
Filter order_date before join_date.
-Then, pull table to show the last item ordered by customer before becoming member.
#### Answer:
| customer_id | product_name |
| ----------- | ---------- |
| A           | sushi        |
| A           | curry        |
| B           | sushi        |

- Customer A’s order before he/she became member is sushi and curry and Customer B’s order is sushi. That must have been a real good sushi!
***
### 8. What is the total items and amount spent for each member before they became a member?

```sql

Select s.customer_id,sum(m.price) as sumsales,count (distinct s.product_id) total_sales
from dbo.sales s
join dbo.menu m
on s.product_id= m.product_id
join dbo.members mm
on s.customer_id=mm.customer_id
where  s.order_date < mm.join_date
group by s.customer_id
```

#### Steps:
- First, filter order_date before their join_date. Then, COUNT unique product_id and SUM the prices total spent before becoming member.

#### Answer:
| customer_id | total_items | total_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 3 |  40       |

Before becoming members,
- Customer A spent $25 on 2 items.
- Customer B spent $40 on 3 items.

***

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

```sql
With points_cte as
( select *, case when m.product_name='sushi' then m.price *20  else m.price*10 end as points
from dbo.menu m
)
select s.customer_id, sum(m.points) points
from points_cte m
join dbo.sales s
on m.product_id=s.product_id
group by s.customer_id

```

#### Steps:
Let's break down the question to understand the point calculation for each customer's purchases.
- Each $1 spent = 10 points. However, `product_id` 1 sushi gets 2x points, so each $1 spent = 20 points.
- Here's how the calculation is performed using a conditional CASE statement:
	- If product_id = 1, multiply every $1 by 20 points.
	- Otherwise, multiply $1 by 10 points.
- Then, calculate the total points for each customer.

#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Total points for Customer A is $860.
- Total points for Customer B is $940.
- Total points for Customer C is $360.

***

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?

```sql
With Interval as
(
select *,
dateadd ( day, 6, mm.join_date ) as weekafter,
eomonth ( '2021-01-31') as Endofjanuary
from dbo.members mm
)
Select i.customer_id,
sum (case 
 when m.product_name = 'sushi' 
then m.price*2*10 
 when s.order_date between  i.join_date  and i.weekafter
then m.price*2*10
 else m.price *10 end) points
From Interval i
Join dbo.sales s
on i.customer_id = s.customer_id
join dbo.menu m 
on s.product_id = m.product_id
where i.weekafter < i.Endofjanuary
group by i.customer_id

```

#### Clarifications:
- On Day -X to Day 1 (the day a customer becomes a member), each $1 spent earns 10 points. However, for sushi, each $1 spent earns 20 points.
- From Day 1 to Day 7 (the first week of membership), each $1 spent for any items earns 20 points.
- From Day 8 to the last day of January 2021, each $1 spent earns 10 points. However, sushi continues to earn double the points at 20 points per $1 spent.

#### Steps:
- Find out customer’s validity date (which is 6 days after join_date and inclusive of join_date) and last day of Jan 2021 (‘2021–01–21’).
- Then, use CASE WHEN to allocate points by dates and product_name.



#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1370 |
| B           | 820 |

- Total points for Customer A is 1,370.
- Total points for Customer B is 820.

***

## BONUS QUESTIONS

### Join All The Things - Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)

```sql

select s.customer_id,s.order_date,m.product_name,price,
case
when s.order_date < mm.join_date
then 'N' 
when s.order_date > = mm.join_date
then 'Y'
else 'N'
end as member
from  dbo.sales s
left join dbo.members mm
on s.customer_id = mm.customer_id
join dbo.menu m
on s.product_id = m.product_id
```
 
#### Answer: 
| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | -------------| ----- | ------ |
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

***

### Rank All The Things - Danny also requires further information about the ```ranking``` of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ```ranking``` values for the records when customers are not yet part of the loyalty program.

```sql

with member_cte 
as 
(select s.customer_id,s.order_date,m.product_name,price,
case
when s.order_date < mm.join_date
then 'N' 
when s.order_date > = mm.join_date
then 'Y'
else 'N'
end as member
from  dbo.sales s
left join dbo.members mm
on s.customer_id = mm.customer_id
join dbo.menu m
on s.product_id = m.product_id
)
select * , 
case 
when member = 'N'
Then NUll 
Else  
rank() over (partition by mm2.customer_id,mm2.member order by mm2.order_date )
end as ranking
from member_cte mm2
```

#### Answer: 
| customer_id | order_date | product_name | price | member | ranking | 
| ----------- | ---------- | -------------| ----- | ------ |-------- |
| A           | 2021-01-01 | sushi        | 10    | N      | NULL
| A           | 2021-01-01 | curry        | 15    | N      | NULL
| A           | 2021-01-07 | curry        | 15    | Y      | 1
| A           | 2021-01-10 | ramen        | 12    | Y      | 2
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| A           | 2021-01-11 | ramen        | 12    | Y      | 3
| B           | 2021-01-01 | curry        | 15    | N      | NULL
| B           | 2021-01-02 | curry        | 15    | N      | NULL
| B           | 2021-01-04 | sushi        | 10    | N      | NULL
| B           | 2021-01-11 | sushi        | 10    | Y      | 1
| B           | 2021-01-16 | ramen        | 12    | Y      | 2
| B           | 2021-02-01 | ramen        | 12    | Y      | 3
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-01 | ramen        | 12    | N      | NULL
| C           | 2021-01-07 | ramen        | 12    | N      | NULL

***
