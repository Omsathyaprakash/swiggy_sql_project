![Swiggy](https://github.com/Omsathyaprakash/swiggy_sql_project/blob/main/swiggy%20photo.jpg)

# SQL Project: Data Analysis for Swiggy - A Food Delivery Company
## Overview
This project demonstrates my SQL problem-solving skills through the analysis of data for Swiggy, a popular food delivery company in India. The project involves setting up the database, importing data, handling null values, and solving a variety of business problems using complex SQL queries.

## Project Structure

- **Database Setup**: Creation of the `swiggy_db` database and the required tables.
- **Data Import**: Inserting sample data into the tables.
- **Data Cleaning**: Handling null values and ensuring data integrity.
- **Business Problems**: Solving 20 specific business problems using SQL queries.

![ERD]( https://github.com/Omsathyaprakash/swiggy_sql_project/blob/main/erd.png)

## Database Setup
```sql
CREATE DATABASE swiggy_db;
```

## 1. Dropping Existing Tables
```sql
DROP TABLE IF EXISTS deliveries;
DROP TABLE IF EXISTS Orders;
DROP TABLE IF EXISTS customers;
DROP TABLE IF EXISTS restaurants;
DROP TABLE IF EXISTS riders;
```

## 2. Creating Tables
```sql
CREATE TABLE restaurants (
    restaurant_id SERIAL PRIMARY KEY,
    restaurant_name VARCHAR(100) NOT NULL,
    city VARCHAR(50),
    opening_hours VARCHAR(50)
);

CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL,
    reg_date DATE
);

CREATE TABLE riders (
    rider_id SERIAL PRIMARY KEY,
    rider_name VARCHAR(100) NOT NULL,
    sign_up DATE
);

CREATE TABLE Orders (
    order_id SERIAL PRIMARY KEY,
    customer_id INT,
    restaurant_id INT,
    order_item VARCHAR(255),
    order_date DATE NOT NULL,
    order_time TIME NOT NULL,
    order_status VARCHAR(20) DEFAULT 'Pending',
    total_amount DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
);

CREATE TABLE deliveries (
    delivery_id SERIAL PRIMARY KEY,
    order_id INT,
    delivery_status VARCHAR(20) DEFAULT 'Pending',
    delivery_time TIME,
    rider_id INT,
    FOREIGN KEY (order_id) REFERENCES Orders(order_id),
    FOREIGN KEY (rider_id) REFERENCES riders(rider_id)
);
```

## Data Import

## Data Cleaning and Handling Null Values

Before performing analysis, I ensured that the data was clean and free from null values where necessary. For instance:

```sql
UPDATE orders
SET total_amount = COALESCE(total_amount, 0);
```

## Business Problems Solved

### 1. Write a query to find the top 5 most frequently ordered dishes by customer called "Arjun Mehta" in the last 1 year.
```sql
select o.order_item,count(o.order_id) as nooforders
from orders o 
join customers c on o.customer_id=c.customer_id
where c.customer_name='Arjun Mehta'
 		and
	  extract(year from o.order_date)
	  =(SELECT max(EXTRACT(YEAR FROM order_date))-1
		FROM orders)
group by 1
order by nooforders desc
limit 5;
```

### 2. Popular Time Slots
-- Question: Identify the time slots during which the most orders are placed. based on 2-hour intervals.
```sql
SELECT
	FLOOR(EXTRACT(HOUR FROM order_time)/2)*2 AS start_time,   --23/2=11.5 ,FLOOR(23/2)=11*2=22 >start_time & 22+2=24 >end_time
	FLOOR(EXTRACT(HOUR FROM order_time)/2)*2 + 2 AS end_time,
	COUNT(*) AS total_orders
FROM 
	orders
GROUP BY
	1, 2
ORDER BY
	3 DESC;
```
### 3. Order Value Analysis
-- Question: Find the average order value per customer who has placed more than 750 orders.
-- Return customer_name, and aov(average order value)

```sql
select c.customer_name,
		AVG(o.total_amount) AS aov
from orders o 
join customers c on o.customer_id=c.customer_id
group by 1
having count(order_id)>750;
```
### 4. High-Value Customers
-- Question: List the customers who have spent more than 100K in total on food orders.
-- return customer_name, and customer_id!

```sql
select c.customer_name,sum(o.total_amount) as totalspent
from orders o
join customers c on o.customer_id=c.customer_id
group by 1
having sum(o.total_amount) > 100000
```

### 5. Orders Without Delivery
-- Question: Write a query to find orders that were placed but not delivered. 
-- Return each restuarant name, city and number of not delivered orders 

```sql
select * from orders;

select r.restaurant_name,r.city,count(*) as "orders_not_delivered"
from orders o
join restaurants r on o.restaurant_id=r.restaurant_id
where o.order_status='Not Fulfilled'
group by 1,2
order by orders_not_delivered desc;
```
### 6. Restaurant Revenue Ranking: 
-- Rank restaurants by their total revenue from the last year, including their name, 
-- total revenue, and rank within their city.

```sql
select r.restaurant_name,r.city,sum(total_amount) as revenuegenerated,
rank() over(partition by r.city
			order by sum(total_amount) desc ) as rn
from orders o
join restaurants r on o.restaurant_id=r.restaurant_id
where extract(year from o.order_date)
	  =(SELECT max(EXTRACT(YEAR FROM order_date))-1
		FROM orders)
group by 1,2;
```

### 7. Most Popular Dish by City: 
-- Identify the most popular dish in each city based on the number of orders.

```sql

select * from
(select o.order_item,r.city,
		count(order_id) as nooforders,
		rank() over(partition by r.city
					order by count(order_id) desc) as rn
from orders o
join restaurants r on o.restaurant_id=r.restaurant_id
group by 1,2)A
where rn=1;
```
### 8. Customer Churn: 
-- Find customers who haven’t placed an order in 2024 but did in 2023.

```sql

select * 
from customers;

select distinct c.customer_id,c.customer_name
from orders o
join customers c on o.customer_id=c.customer_id
where extract(year from order_date)=2023
	and c.customer_id
			not in
	(select distinct c.customer_id
	from orders o
	join customers c on o.customer_id=c.customer_id
	where extract(year from order_date)=2024)
```

### 9. Cancellation Rate Comparison: 
-- Calculate and compare the order cancellation rate for each restaurant between the 
-- current year and the previous year.

```sql


with cancelratio23 as
(
select o.restaurant_id,count(o.order_id) as totalorders,
		count(case when d.delivery_id is null then 1 end) as notdelivered
from orders o
left join deliveries d on o.order_id=d.order_id
where extract(year from o.order_date)= 2023
group  by 1
),
cancelratio24 as
(
select o.restaurant_id,count(o.order_id) as totalorders,
		count(case when d.delivery_id is null then 1 end) as notdelivered
from orders o
left join deliveries d on o.order_id=d.order_id
where extract(year from o.order_date)= 2024
group  by 1
),
lastyeardata as
(
select restaurant_id,totalorders,notdelivered,
		round((notdelivered::numeric/totalorders)*100,2) as cancelratio
from cancelratio23
),
currentyeardata as
(
select restaurant_id,totalorders,notdelivered,
		round((notdelivered::numeric/totalorders)*100,2) as cancelratio
from cancelratio24
)
select c.restaurant_id,
		c.cancelratio,
		l.cancelratio
from currentyeardata c
join lastyeardata l 
on c.restaurant_id=l.restaurant_id;
```

### 10. Rider Average Delivery Time: 
-- Determine each rider's average delivery time.

```sql


SELECT
	o.order_id,
	o.order_time,
	d.delivery_time,
	d.rider_id,
	d.delivery_time - o.order_time AS time_difference,
	EXTRACT(
			EPOCH FROM (
						d.delivery_time - o.order_time +
						CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 DAY'
						ELSE INTERVAL '0 DAY' END))/60 AS time_diff_min
FROM 
	orders AS o
JOIN 
	deliveries AS d
ON
	o.order_id = d.delivery_id
WHERE
	d.delivery_status = 'Delivered'
ORDER BY
time_diff_min ASC;

-- EXTRACT(EPOCH FROM ...)> time interval into a number representing seconds. "Epoch" time format as the number of seconds 

```


### 11. Monthly Restaurant Growth Ratio: 
-- Calculate each restaurant's growth ratio based on the total number of delivered orders since its joining

```sql
with monthly_orders as
(select restaurant_id,
	TO_CHAR(o.order_date, 'mm-yy') AS monthname,
	count(o.order_id) as currentmonthorders,
	lag(count(o.order_id)) over(partition by restaurant_id
								order by TO_CHAR(order_date, 'mm-yy')) as prevmonthorders
from orders o
join deliveries d on o.order_id=d.order_id
where d.delivery_status='Delivered'
group by 1,2
order by 1,2)
select restaurant_id,monthname,prevmonthorders,currentmonthorders,
		ROUND((currentmonthorders::Numeric - prevmonthorders::Numeric)/currentmonthorders::Numeric * 100 , 2) AS growth_ratio
from monthly_orders;
```

### 12. Customer Segmentation: 
-- Customer Segmentation: Segment customers into 'Gold' or 'Silver' groups based on their total spending 
-- compared to the average order value (AOV). If a customer's total spending exceeds the AOV, 
-- label them as 'Gold'; otherwise, label them as 'Silver'. Write an SQL query to determine each segment's 
-- total number of orders and total revenue

```sql
with cte as
(select customer_id,sum(total_amount) as totalspent,
				count(order_id) as totalorders,
case when sum(total_amount)>(select avg(total_amount) from orders) then 'Gold'
	else 'Silver' end as customersegment
from orders
group by 1)
select customersegment,sum(totalorders) as nooforders,
		sum(totalspent) as totalrevenue
from cte
group by 1;

select *from orders;
```

### 13. Rider Monthly Earnings: 
-- Calculate each rider's total monthly earnings, assuming they earn 8% of the order amount.

```sql
select d.rider_id,TO_CHAR(o.order_date, 'mm-yy'),sum(o.total_amount)*0.08 as earnings
from orders o join deliveries d on o.order_id=d.order_id 
where d.delivery_status='Delivered'
group by 1,2
order by 1,2;
```

### Q.14 Rider Ratings Analysis: 
-- Find the number of 5-star, 4-star, and 3-star ratings each rider has.
-- riders receive this rating based on delivery time.
-- If orders are delivered less than 15 minutes of order received time the rider get 5 star rating,
-- if they deliver 15 and 20 minute they get 4 star rating 
-- if they deliver after 20 minute they get 3 star rating.

```sql
select rider_id,ratings,count(*) as totalratings
from
(select rider_Id,
case when time_diff_min<15 then '5-Star'
	 when time_diff_min between 15 and 20 then '4-Star'
	 else '3-Star' end as ratings
from
(select d.rider_id,
EXTRACT(
			EPOCH FROM (
						d.delivery_time - o.order_time +
						CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 DAY'
						ELSE INTERVAL '0 DAY' END))/60 AS time_diff_min

FROM 
	orders AS o
JOIN 
	deliveries AS d
ON
	o.order_id = d.delivery_id
WHERE
	d.delivery_status = 'Delivered') A)B
group by 1,2
order by 1,3 desc;

```

### 15. Q.15 Order Frequency by Day: 
-- Analyze order frequency per day of the week and identify the peak day for each restaurant.

```sql
SELECT
	restaurant_name,
	day,
	total_orders
FROM
(
	SELECT
		r.restaurant_name,
		TO_CHAR(o.order_date, 'Day') AS day,
		COUNT(o.order_id) AS total_orders,
		RANK() OVER(PARTITION BY r.restaurant_name ORDER BY COUNT(o.order_id) DESC)
	FROM
		orders AS o
	JOIN
		restaurants AS r
	ON
		o.restaurant_id = r.restaurant_id
	GROUP BY 
		1, 2
	ORDER BY
		1, 3 DESC
)
WHERE 
	rank = 1;
```

### 16. Customer Lifetime Value (CLV): 
-- Calculate the total revenue generated by each customer over all their orders.

```sql
select c.customer_name,sum(total_amount) as revenue
from orders o join customers c on o.customer_id=c.customer_id
group by 1
order by 2 desc;
```

### 17. Monthly Sales Trends: 
-- Identify sales trends by comparing each month's total sales to the previous month.

```sql
SELECT YEAR,MONTH,ROUND(((TOTALSALES-PREVSALES)/(PREVSALES)*100),2) AS monthly_growthrate
FROM
(select extract(year from order_date) as year,
		extract(month from order_date) as month,
		sum(total_amount) as totalsales,
		lead(sum(total_amount)) over(order by 1,2) AS PREVSALES
from orders
group by 1,2
order by 1,2)A
```

### 18. Rider Efficiency: 
-- Evaluate rider efficiency by determining average delivery times and identifying those with the lowest and highest averages.

```sql
select rider_name,avg_deliverytime from
(select rider_name,avg_deliverytime,
		rank() over(order by avg_deliverytime desc) as rndesc,
		rank() over(order by avg_deliverytime asc) as rnasc
from
(select r.rider_name,
		round(
				avg(
					round(
						EXTRACT(EPOCH FROM (
						d.delivery_time - o.order_time +
						CASE WHEN d.delivery_time < o.order_time THEN INTERVAL '1 DAY'
						ELSE INTERVAL '0 DAY' END))/60,2)),2) AS avg_deliverytime
from orders o
join deliveries d on o.order_id=d.order_id
join riders r on d.rider_id=r.rider_id
where d.delivery_status='Delivered'
group by 1)A)B
where rndesc=1 or rnasc=1;

```

### 19. Order Item Popularity: 
-- Track the popularity of specific order items over time and identify seasonal demand spikes.

```sql
SELECT
	order_item,
	seasons,
	COUNT(order_id) AS total_orders
FROM(
	SELECT
		*,
		EXTRACT(MONTH FROM order_date) AS month,
		CASE 
			WHEN EXTRACT(MONTH FROM order_date) BETWEEN 3 AND 4 THEN 'Spring'
			WHEN EXTRACT(MONTH FROM order_date) > 4 AND EXTRACT(MONTH FROM order_date) < 7 THEN 'Summer'
			WHEN EXTRACT(MONTH FROM order_date) > 6 AND EXTRACT(MONTH FROM order_date) < 10 THEN 'Monsoon'
			WHEN EXTRACT(MONTH FROM order_date) > 9 AND EXTRACT(MONTH FROM order_date) < 12 THEN 'Autumn'
			ELSE 'Winter'
		END AS seasons
	FROM
		orders
	) AS t1
GROUP BY
	1, 2
ORDER BY
	1, 3 DESC
```

### 20. Rank each city based on the total revenue for last year 2023
```sql
SELECT
	EXTRACT(YEAR FROM o.order_date) AS year,
	r.city,
	SUM(o.total_amount) AS total_revenue,
	RANK() OVER(ORDER BY SUM(o.total_amount) DESC) AS city_rank
FROM
	restaurants AS r
JOIN
	orders AS o
ON
	r.restaurant_id = o.restaurant_id
GROUP BY
	1, 2
HAVING
	EXTRACT(YEAR FROM o.order_date) = 2023
ORDER BY
	4

```

## Conclusion

This project highlights my ability to handle complex SQL queries and provides solutions to real-world business problems in the context of a food delivery service like Swiggy. The approach taken here demonstrates a structured problem-solving methodology, data manipulation skills, and the ability to derive actionable insights from data.

## Notice 
All customer names and data used in this project are computer-generated using AI and random functions. They do not represent real data associated with Swiggy or any other entity. This project is solely for learning and educational purposes, and any resemblance to actual persons, businesses, or events is purely coincidental.
