# 🍜 Case Study #1: Danny's Diner
## Solution
### 1. What is the total amount each customer spent at the restaurant?
````sql
SELECT 
sum(price) total_sales, customer_id
FROM menu
INNER JOIN
sales ON menu.product_id = sales.product_id
GROUP BY customer_id; 
````
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
SELECT customer_id, count(distinct(order_date)) visit_count 
from sales 
group by customer_id;
````
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
WITH sales_order_cte as
(
	  select customer_id,product_name,order_date,
    dense_rank() over(partition by customer_id order by order_date) as rnk
    FROM sales s inner join menu m on
    s.product_id = m.product_id
)
SELECT customer_id, product_name 
from sales_order_cte 
where rnk = 1
group by customer_id,product_name;
````
#### Answer:
| customer_id | product_name | 
| ----------- | ----------- |
| A           | curry        | 
| A           | sushi        | 
| B           | curry        | 
| C           | ramen        |

- Customer A's first orders are curry and sushi.
- Customer B's first order is curry.
- Customer C's first order is ramen.

***
### 4.What is the most purchased item on the menu and how many times was it purchased by all customers?
````sql
SELECT  (COUNT(s.product_id)) AS most_purchased, product_name
FROM sales AS s
JOIN menu AS m
ON s.product_id = m.product_id
GROUP BY s.product_id, product_name
ORDER BY most_purchased DESC limit 1;
````

#### Answer:
| most_purchased | product_name | 
| ----------- | ----------- |
| 8       | ramen |


- Most purchased item on the menu is ramen which is 8 times. Yummy!

***
### 5.Which item was the most popular for each customer?
````sql
WITH most_fav_cte as
(
	SELECT s.customer_id,m.product_name,count(m.product_id) as order_count,
    DENSE_RANK() OVER(PARTITION BY s.customer_id order by count(s.customer_id) desc) as rnk
    from menu m join sales s
    on m.product_id = s.product_id
    group by customer_id,product_name
)
SELECT customer_id,product_name,order_count from most_fav_cte where rnk =1;
````
#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

- Customer A and C's favourite item is ramen.
- Customer B enjoys all items on the menu. He/she is a true foodie, sounds like me!

***
### 6.Which item was purchased first by the customer after they became a member?
````sql
WITH members_sales_cte as 
(
	SELECT s.customer_id,s.product_id,s.order_date,m.join_date,
    dense_rank() over(partition by s.customer_id order by s.order_date) as rnk
    from sales s join members m on
    s.customer_id =m.customer_id
    where s.order_date >= m.join_date
)
SELECT s.customer_id,s.order_date,m2.product_name
FROM members_sales_cte s join menu m2
on s.product_id = m2.product_id
where rnk=1;
````
#### Answer:
| customer_id | order_date  | product_name |
| ----------- | ---------- |----------  |
| A           | 2021-01-07 | curry        |
| B           | 2021-01-11 | sushi        |

- Customer A's first order as member is curry.
- Customer B's first order as member is sushi.

***
### 7.Which item was purchased just before the customer became a member?
````sql
with before_members_cte as
(
	select s.customer_id,s.order_date,m.join_date,s.product_id,
    dense_rank() over(partition by s.customer_id order by s.order_date desc) as rnk
    from sales s join members m on
    s.customer_id = m.customer_id
    where s.order_date < m.join_date
)

select s.customer_id,m2.product_name,s.order_date
from before_members_cte s join menu m2
on s.product_id = m2.product_id
where rnk =1;
````
#### Answer:
| customer_id | order_date  | product_name |
| ----------- | ---------- |----------  |
| A           | 2021-01-01 |  sushi        |
| A           | 2021-01-01 |  curry        |
| B           | 2021-01-04 |  sushi        |

- Customer A’s last order before becoming a member is sushi and curry.
- Whereas for Customer B, it's sushi. That must have been a real good sushi!

***
### 8.What is the total items and amount spent for each member before they became a member?
````sql
SELECT s.customer_id,sum(m2.price) total_sales,count(distinct(s.product_id)) unique_menu_item
from sales s 
inner join members m 
on s.customer_id = m.customer_id
join menu m2 
on s.product_id = m2.product_id
where s.order_date < m.join_date
group by s.customer_id;
````
#### Answer:
| customer_id | unique_menu_item | total_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 2 |  40       |

Before becoming members,
- Customer A spent $ 25 on 2 items.
- Customer B spent $40 on 2 items.

***
### 9.If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?
````sql
WITH all_points_cte as
(
	SELECT *,
    CASE
		  WHEN m.product_id = 1 THEN price * 20
      ELSE price * 10
      END AS points
      FROM menu m
)
SELECT s.customer_id, sum(p.points) total_points
from all_points_cte p 
join sales s on
s.product_id = p.product_id
group by s.customer_id;
````
#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 860 |
| B           | 940 |
| C           | 360 |

- Total points for Customer A is 860.
- Total points for Customer B is 940.
- Total points for Customer C is 360.

***
### 10.In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi 
— how many points do customer A and B have at the end of January?
````sql
with joined_table as (
  SELECT 
    sales.customer_id, 
    Menu.product_name, 
    Menu.price, 
    sales.order_date, 
    members.join_date, 
    join_date +6 as one_week
  FROM dannys_diner.sales 
   JOIN dannys_diner.menu 
    ON sales.product_id = menu.product_id
   JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
)
select 
  sum(case
    when order_date >= join_date and order_date <= one_week then price* 20
    when  product_name = 'sushi' then price*20
    else price* 10
  end) as total_counts,
  customer_id
from joined_table
where order_date <= '2021-01-31'
group by customer_id;
````
#### Answer:
| customer_id | total_points | 
| ----------- | ---------- |
| A           | 1370 |
| B           | 820 |

- Total points for Customer A is 1,370.
- Total points for Customer B is 820.

***
