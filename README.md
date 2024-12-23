# E-Commerce-Sales-Data-Analysis-Using-SQL

![image] {}

## Objective 
The aim of this project is to analyze e-commerce sales data using SQL to extract meaningful insights about customer behavior, product performance, sales trends, and payment methods. By exploring the data, the project identifies regions with the highest customer activity. It also focuses on analyzing product sales by category and price range, highlighting best-selling items and underperforming products.
Additionally, the project uncovers order patterns, such as frequency, seasonal trends, and peak sales periods, while analyzing revenue growth on a monthly and quarterly basis. Payment insights, including customer preferences for payment methods.


-- Customer Analysis: 

-- Count the total number of customers 
select count(*) as total_customers from customers; 

-- Find the count of customers from each region. Display the region name along with the total number of customers. 
select region, count(*) count_of_customers from customers group by region order by count_of_customers desc; 

-- Calculate the average quantity of products ordered by each customer. 
select c.customer_id, c.name, round(avg(quantity),0) as avg_quantity
from orders as o inner join customers as c on c.customer_id = o.customer_id
group by c.customer_id, c.name;

-- Identify the top 5 customers who have contributed the most revenue (total amount spent). Display the customer name, total amount spent, and their rank based on the descending order of revenue. 
with customer_rank_cte1 as (
select c.customer_id, c.name, round(sum(o.total_amount),0) as total_amount_spent
from customers as c inner join orders as o on c.customer_id = o.customer_id
group by c.customer_id, c.name), 
customer_rank_cte2 as ( 
select *,
dense_rank() over(order by total_amount_spent desc) as ranking
from customer_rank_cte1
) 
select * from customer_rank_cte2 where ranking <= 5; 

-- For each customer, retrieve the very first order, the most recent order along with the total payment amount and order date.
with customer_orders_cte1 as(
select customer_id, order_date, total_amount,
row_number() over(partition by customer_id order by order_date) as first_order,
row_number() over(partition by customer_id order by order_date desc) as last_order
from orders
)
select customer_id, 
max(case when first_order = 1 then order_date end) as first_order_date,
round(max(case when first_order = 1 then total_amount end),0) as first_order_amount,
max(case when last_order = 1 then order_date end) as last_order_date,
round(max(case when last_order = 1 then total_amount end),0) as last_order_amount
from customer_orders_cte1 group by customer_id;

-- write a sql query to find the top 5 customers with the highest total spending in the last 6 months.
with top_5_customers as(
select c.customer_id, c.name, round(sum(o.total_amount),0) as total_spending
from orders as o inner join customers as c on o.customer_id = c.customer_id where order_date > current_date() - interval 6 month
group by c.customer_id, c.name
)
select * from top_5_customers order by total_spending desc limit 5;
-- *************************************************************************

-- Product Analysis:

-- List all product categories and the number of products in each category. Display the category name along with the corresponding product count.
select category, count(*) as no_of_products from products group by category;

-- Find the total revenue generated by each product:
select p.product_id, p.product_name, round(sum(o.total_amount),0) as total_revenue from products as p inner join orders as o on p.product_id = o.product_id
group by p.product_id, p.product_name;

-- Find the top 3 products in terms of revenue for each category:
with top_3_products as(
select p.category, p.product_name, round(sum(o.total_amount),0) as revenue
from products as p inner join orders as o on p.product_id = o.product_id
group by p.product_name, p.category order by p.category),
cte1 as(
select *,
dense_rank() over(partition by category order by revenue desc) ranking
from top_3_products)
select * from cte1 where ranking <= 3; 

-- Find the total revenue for each product category in the products table.
select p.category, round(sum(o.total_amount),0) as total_revenue from products as p inner join orders as o on p.product_id = o.product_id
group by p.category;

-- For each product, calculate the total quantity sold, the total payment received.
select p.product_id, p.product_name, sum(o.quantity) as total_quantity_sold, round(sum(o.total_amount),0) as total_payment_received
from products as p left join orders as o on p.product_id = o.product_id
group by p.product_id, p.product_name;

-- Write an SQL query to find the three products with the lowest total sales (revenue) in each category.
with least_selling_products as(
select p.category, p.product_name, round(sum(o.total_amount),0) as total_sales
from products as p inner join orders as o on p.product_id = o.product_id
group by p.category, p.product_name
),
least_selling_products_cte2 as(
select *,
dense_rank() over(partition by category order by total_sales) as rn
from least_selling_products
)
select * from least_selling_products_cte2 where rn <= 3;

-- *************************************************************************

-- Order Trends:

-- Count the total number of orders placed

select count(*) as total_number_of_orders from orders;

-- Find the number of orders placed in each year. Display the year along with the total count of orders
select year(order_date) as year, count(*) count_of_orders from orders group by year(order_date) order by year(order_date);


-- Calculate the running total (total_amount) for each order in the orders table, ordered by order_date.
select order_id, order_date, total_amount,
round(sum(total_amount) over(order by order_date, order_id),0) as running_total from orders;

-- **************************************************************************
-- Payment Insights:

-- Identify the payment method through which the maximum number of payments were made. Display the payment method and the corresponding total count of payments.

select method, count(*) as no_of_payments from payments group by method;

-- **************************************************************************

-- Comprehensive Sales Analysis

-- Find the total sales (total_amount) for each region.
select c.region, round(sum(total_amount),0) as total
from customers as c inner join orders as o on c.customer_id = o.customer_id
group by c.region order by total desc;

-- What is the total number of orders and total payment amount for each customer?
select c.customer_id, c.name, count(o.order_id) as total_number_of_order, 
round(sum(total_amount),0) as total_payment
from customers as c inner join orders as o on c.customer_id = o.customer_id
group by c.customer_id, c.name;

-- Retrieve the most recent order for each customer along with the total payment amount and order date.
with cte as(
select *,
row_number() over(partition by customer_id order by order_date desc) as rn
from orders)
select c.customer_id, c.name, order_date as most_recent_order_date, total_amount from
cte inner join customers as c on cte.customer_id = c.customer_id where rn = 1;

-- Write an SQL query to calculate the total sales for each region, displaying the total sales for each year (from the earliest year in the dataset to the latest year).
with cte as(
select c.region, year(order_date) as year, round(sum(total_amount),0) as total_sales
from customers as c inner join orders as o on c.customer_id = o.customer_id group by c.region, year(order_date)
)
select region,
sum(case when year = 2022 then total_sales end) as sales_2022,
sum(case when year = 2023 then total_sales end) as sales_2023,
sum(case when year = 2024 then total_sales end) as sales_2024
from cte group by region;

-- Write an SQL query to calculate the total sales for each category, displaying the total sales for each year (from the earliest year in the dataset to the latest year).
with cte as(
select p.category, year(o.order_date) as year, round(sum(total_amount),0) as total_sales
from products as p inner join orders as o on p.product_id = o.product_id group by p.category, year(o.order_date)
)
select category,
sum(case when year= 2022 then total_sales end) as sales_2022,
sum(case when year= 2023 then total_sales end) as sales_2023,
sum(case when year= 2024 then total_sales end) as sales_2024
from cte group by category;

-- Write a sql query to calculate year over year growth in sales from a orders table.
with yoy_sales as(
select year(order_date) as order_year, sum(total_amount) as total_sales
from orders group by year(order_date) order by order_year), 
yoy_sales2 as(
select order_year, total_sales,
lag(total_sales) over() as previous_year_sales 
from yoy_sales)
select order_year, round(total_sales,0) as current_year_sales, round(previous_year_sales,0) as previous_year_sales,
concat(round((total_sales - previous_year_sales)*100 / previous_year_sales,0),'%') as yoy_growth
 from yoy_sales2;

-- Determine the month with the highest total sales for each year. Display the year, month, and the total sales amount
with highest_total_sales as (
select year(order_date) as year, monthname(order_date) as month, sum(total_amount) as total_sales from orders group by year(order_date), monthname(order_date) order by year(order_date)),
highest_total_sales_cte2 as (
select *,
dense_rank() over(partition by year order by total_sales desc) as ranking
from highest_total_sales
)
select year, month, round(total_sales,0)  as total_sales from highest_total_sales_cte2 where ranking = 1;


## Key Insights :

Identified the top 5 customers contributing significantly to total revenue, with Rebecca May leading at $50,000, followed by Jessica Stout at $44,352.

South America leads with the highest number of customers at 517, followed closely by Asia with 516, North America with 484, and Europe with 483.

In product range, the maximum number of products are under the clothing category, totaling 36, followed by electronics with 35, and finally furniture with 29.

The maximum revenue is generated by the clothing category, amounting to 13,623,658, followed by furniture with 13,299,819.

The maximum number of orders were placed in the year 2023, totaling 14,883, followed by 2024 with 14,133, and finally 2022 with 984.

The most popular payment method among customers is credit card, with 10,132 customers making payments through it, followed by PayPal with 10,046, and finally bank transfer with 9,822.

The highest sales are from Asia region, amounting to 10,346,319, followed by South America with 10,231,948.

In 2022, total sales amounted to 1,302,491, while 2023 saw a record-high sales figure of 19,562,924. In 2024, sales reached 18,815,141.

