# SQL_CASE_STUDY

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.


![App Screenshot](https://user-images.githubusercontent.com/81607668/127727503-9d9e7a25-93cb-4f95-8bd0-20b87cb4b459.png)


## 📚Table of Contents
The Problem

The Approach

The Answers

## The Problem
Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!
## The Approach
1st : Fully Understand and grasp the business problem with it's full dimensions

2nd : obersve the Entity Relationship Diagram 

3rd : Assess the data quality

4th : Start the Analysis
## Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png)
## Business Questions

#### 1. What is the total amount each customer spent at the restaurant?

````sql
SELECT
 	 s.customer_id
  	, SUM(price) as total_amount
  from dannys_diner.sales as s 
  left JOIN dannys_diner.menu as M
  ON S.product_id = m.product_id
  Group by s.customer_id
  Order by total_amount DESC
````
#### Explaination:
- Use *SUM* and *GROUP BY* to find out total_amount contributed by each customer.
- Use *JOIN* to merge sales and menu tables as customer_id and price are from both tables.


| customer_id | total_amount |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

- Customer A spent $76.
- Customer B spent $74.
- Customer C spent $36.
*

#### 2. How many days has each customer visited the restaurant?


````sql
SELECT customer_id, count (DISTINCT order_date) as total_days 
  from dannys_diner.sales
  group BY customer_id
  order by total_days desc
````
#### Explaination:
- Use *DISTINCT* and merge with *COUNT* to find out the total_visit_days for each customer.
#### Answer:
| customer_id | visit_count |
| ----------- | ----------- |
| A           | 4          |
| B           | 6          |
| C           | 2          |

- Customer A visited 4 times.
- Customer B visited 6 times.
- Customer C visited 2 times.

*
#### 3. What was the first item from the menu purchased by each customer?
````sql
WITH sample_cte as 
(SELECT customer_id,
 product_name,
 order_date,DENSE_RANK()
 over(PARTITION by s.customer_id
      order BY s.order_date) AS "rank"
 from dannys_diner.sales as s 
 join dannys_diner.menu as m 
 using (product_id))

 SELECT customer_id,
 product_name as first_item_bought
 FROM sample_cte
 where "rank"=1
 GROUP BY customer_id,
 product_name;
````
#### Explaination: 
- Create a temp table sample_cte and use *Windows function* with *DENSE_RANK* to create a new column rank based on order_date.
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
*
### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

````sql
SELECT product_name,count(s.product_id) as total_purchases 
from dannys_diner.sales AS S
join dannys_diner.menu AS M 
on s.product_id = m.product_id
group by product_name
order by total_purchases desc LIMIT 1
````
#### Explaination:
- *COUNT* number of product_id and *ORDER BY* purchase_count by descending order. 
#### Answer:
| prodcut_name | purchase_count | 
| ----------- | ----------- |
| ramen       | 8 |

*

### 5. Which item was the most popular for each customer?
````sql
SELECT s.customer_id,product_name, count(s.product_id) as number_of_products 
from dannys_diner.menu as m 
left join dannys_diner.sales as s
on s.product_id = m.product_id
group by customer_id, product_name
order by number_of_products DESC, customer_id

WITH sample_cte as 
(SELECT customer_id,
 product_name,
 count(product_id),DENSE_RANK()
 over(PARTITION by s.customer_id
      order BY M.product_id) AS "rank"
 from dannys_diner.sales as s 
 join dannys_diner.menu as m 
 using (product_id))
````
 #### Explaination:
 - Create a fav_item and use *DENSE_RANK* to rank the order_count for each product by descending order for each customer.
- Generate results where product rank = 1 only as the most popular product for each customer.

#### Answer:
| customer_id | product_name | order_count |
| ----------- | ---------- |------------  |
| A           | ramen        |  3   |
| B           | sushi        |  2   |
| B           | curry        |  2   |
| B           | ramen        |  2   |
| C           | ramen        |  3   |

*
### 6. Which item was purchased first by the customer after they became a member?
````sql
WITH sample_CTE AS
(select customer_id, 
 product_id,
 join_date,
 order_date, 
 DENSE_RANK() OVER(PARTITION by customer_id order by order_date) as RANK
from dannys_diner.sales as s
JOIN dannys_diner.members as m 
 USING (customer_id)
 WHERE order_date >= join_date)
 
 
 SELECT s.customer_id, m2.product_name, s.order_date
 FROM  sample_CTE AS s 
 JOIN dannys_diner.menu as m2 
 USING (product_id)
where rank =1 
````
 #### Explaination:

- Create cte by using *windows function* and partitioning customer_id by ascending order_date. Then, filter order_date to be on or after join_date.
- Then, filter table by rank = 1 to show 1st item purchased by each customer.

#### Answer:
| customer_id | order_date  | product_name |
| ----------- | ---------- |----------  |
| A           | 2021-01-07 | curry        |
| B           | 2021-01-11 | sushi        |

* 

### 7. Which item was purchased just before the customer became a member?
````sql
WITH sample_CTE AS
(select customer_id, 
 product_id,
 join_date,
 order_date, 
 DENSE_RANK() OVER(PARTITION by customer_id order by order_date) as RANK
from dannys_diner.sales as s
JOIN dannys_diner.members as m 
 USING (customer_id)
 WHERE order_date < join_date)
 
 SELECT s.customer_id, m2.product_name, s.order_date
 FROM  sample_CTE AS s 
 JOIN dannys_diner.menu as m2 
 USING (product_id)
where rank =1 
````
  #### Explaination:
  - Create a cte to create new column rank by using *Windows function* and partitioning customer_id by descending order_date to find out the last order_date before customer becomes a member.
- Filter order_date < join_date.

#### Answer:
| customer_id | order_date  | product_name |
| ----------- | ---------- |----------  |
| A           | 2021-01-01 |  sushi        |
| A           | 2021-01-01 |  curry        |
| B           | 2021-01-04 |  sushi        |

*
### 8. What is the total items and amount spent for each member before they became a member?
````sql
SELECT 
s.customer_id,
count(s.product_id) as total_items,
sum(m.price) as total_sales
from dannys_diner.sales as s
JOIN dannys_diner.menu as m 
 USING (product_id)
JOIN dannys_diner.members as m2
 USING (customer_id)
WHERE order_date < join_date
GROUP by s.customer_id
order by s.customer_id
````
 #### Explaination:
 - Filter order_date < join_date and perform a *COUNT* *DISTINCT* on product_id and *SUM* the price before becoming member.

 #### Answer:
| customer_id | DISTINCT_product | SUM_sales |
| ----------- | ---------- |----------  |
| A           | 2 |  25       |
| B           | 2 |  40       |
