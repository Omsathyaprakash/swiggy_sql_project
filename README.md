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
