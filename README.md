# 8 Week SQL Challenge

Thanks @DataWithDanny for the excellent SQL case studies! 👋🏻
* Find his challenge website on **[#8WeekSQLChallenge](https://8weeksqlchallenge.com)**
* Furthermore, I would recommend his course for anyone looking to get advanced SQL skills **[Serious-SQL](https://www.datawithdanny.com/courses/serious-sql)**

## Case Study #1: Danny's Diner 
<img src="https://8weeksqlchallenge.com/images/case-study-designs/1.png" alt="Image" width="450" height="450">

[Link to case study](https://8weeksqlchallenge.com/case-study-1/)

### Business Problem:
Danny just started a japonese food business. He wants to leverage the data that he collected by creating some Dataset and answer a few questions regarding his customer, their habits and whether to expand the customer loyalty program or not.

### Entity Relationship Diagram:

![Entity diagram](https://user-images.githubusercontent.com/89108170/165634684-2a08d263-90a8-48e7-8819-b7a3e0ea649f.png)

### Case Study Questions:
<details>
<summary>
Click here to expand!
</summary>

1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
10. What is the total items and amount spent for each member before they became a member?
11. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
12. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

### Solutions:
 
1)
SELECT customer_id, sum(dannys_diner.menu.price) as total_sale
FROM dannys_diner.sales
INNER JOIN dannys_diner.menu
ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY customer_id
ORDER BY sum(dannys_diner.menu.price) DESC

2)
SELECT customer_id, COUNT(DISTINCT order_date) as total_visit
FROM dannys_diner.sales
GROUP BY customer_id

3)
WITH ordered_sales_cte AS
(
 SELECT customer_id, order_date, product_name,
  DENSE_RANK() OVER(PARTITION BY s.customer_id
  ORDER BY s.order_date) AS rank
 FROM dannys_diner.sales AS s
 JOIN dannys_diner.menu AS m
  ON s.product_id = m.product_id
)

SELECT customer_id, product_name
FROM ordered_sales_cte
WHERE rank = 1
GROUP BY customer_id, product_name;

4)
SELECT dannys_diner.menu.product_name, count(dannys_diner.sales.product_id) as most_wanted
FROM dannys_diner.sales
join dannys_diner.menu 
ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
group by dannys_diner.menu.product_name, dannys_diner.sales.product_id
order by dannys_diner.sales.product_id desc
limit 1;

5)
WITH popular_each_cst as(
SELECT dannys_diner.sales.customer_id, dannys_diner.menu.product_name, count(dannys_diner.sales.product_id) as order_count,
DENSE_RANK() OVER(PARTITION BY dannys_diner.sales.customer_id
ORDER BY count(dannys_diner.sales.customer_id) desc) as rank
from dannys_diner.sales
join dannys_diner.menu
on dannys_diner.sales.product_id = dannys_diner.menu.product_id
group by dannys_diner.sales.customer_id, dannys_diner.menu.product_name, dannys_diner.sales.product_id
)

Select customer_id, product_name, order_count
from popular_each_cst
where rank = 1;

6)
with first_sold as(
SELECT dannys_diner.members.customer_id, dannys_diner.sales.order_date, dannys_diner.members.join_date, dannys_diner.menu.product_name,
DENSE_RANK() OVER(PARTITION BY dannys_diner.sales.customer_id
ORDER BY dannys_diner.sales.order_date) as rank
from dannys_diner.members
JOIN dannys_diner.sales 
ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
JOIN dannys_diner.menu
ON dannys_diner.menu.product_id =dannys_diner.sales.product_id
WHERE dannys_diner.sales.order_date >= dannys_diner.members.join_date
)

Select customer_id, to_char(order_date, 'DD/MM/YYYY'), product_name
FROM first_sold
WHERE RANK = 1;

7)
with first_sold as(
SELECT dannys_diner.members.customer_id, dannys_diner.sales.order_date, dannys_diner.members.join_date, dannys_diner.menu.product_name,
DENSE_RANK() OVER(PARTITION BY dannys_diner.sales.customer_id
ORDER BY dannys_diner.sales.order_date desc) as rank
from dannys_diner.members
JOIN dannys_diner.sales 
ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
JOIN dannys_diner.menu
ON dannys_diner.menu.product_id =dannys_diner.sales.product_id
WHERE dannys_diner.sales.order_date < dannys_diner.members.join_date
)

Select customer_id, to_char(order_date, 'DD/MM/YYYY') as order_date, product_name
FROM first_sold
where rank = 1;

8)
with first_sold as(
SELECT dannys_diner.members.customer_id AS customer_id, dannys_diner.sales.order_date, dannys_diner.members.join_date, dannys_diner.menu.product_name, dannys_diner.sales.product_id AS product_id, dannys_diner.menu.price AS price
from dannys_diner.members
JOIN dannys_diner.sales 
ON dannys_diner.sales.customer_id = dannys_diner.members.customer_id
JOIN dannys_diner.menu
ON dannys_diner.menu.product_id =dannys_diner.sales.product_id
WHERE dannys_diner.sales.order_date < dannys_diner.members.join_date
)

SELECT customer_id, count(distinct(product_id)), SUM(price)
FROM first_sold
Group by customer_id;

9)
WITH price_points AS
 (
 SELECT *, 
 CASE
  WHEN product_id = 1 THEN price * 20
  ELSE price * 10
  END AS points
 FROM dannys_diner.menu
 )
 
 
SELECT s.customer_id, SUM(p.points) AS total_points
FROM price_points AS p
JOIN dannys_diner.sales AS s
ON p.product_id = s.product_id
GROUP BY s.customer_id;

10)
WITH dates_cte AS 
(
 SELECT *, 
 dannys_diner.members.join_date + INTERVAL '6 day'  AS valid_date, 
 date_trunc('month', dannys_diner.members.join_date) + interval '1 month' -  interval '1 day' AS last_date
 FROM dannys_diner.members
)

SELECT d.customer_id, s.order_date, d.join_date, 
 d.valid_date, d.last_date, m.product_name, m.price,
 SUM(CASE
  WHEN m.product_name = 'sushi' THEN 2 * 10 * m.price
  WHEN s.order_date BETWEEN d.join_date AND d.valid_date THEN 2 * 10 * m.price
  ELSE 10 * m.price
  END) AS points
FROM dates_cte AS d
JOIN dannys_diner.sales AS s
 ON d.customer_id = s.customer_id
JOIN dannys_diner.menu AS m
 ON s.product_id = m.product_id
WHERE s.order_date < d.last_date
GROUP BY ROLLUP(d.customer_id, s.order_date, d.join_date, d.valid_date, d.last_date, m.product_name, m.price)
order by points desc;
