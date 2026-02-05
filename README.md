-- project: Pizza Sales Analysis

create database `Pizza Sales`;
use `Pizza Sales`;

-- Create tabels order_details
Create table order_details (
	order_details_id int primary key ,
	order_id int ,
	pizza_id varchar(30) ,
	quantity int
    ) ;


-- Create tabels order_date
create table order_date (
	order_id int primary key ,
	date date ,
	time time 
    ) ;


-- Create tabels pizza_type
create table pizza_type (
	pizza_type_id varchar(30) ,
	name varchar(60) ,
	category varchar(20) ,
	ingredients varchar(150)
    );


-- Create tabels pizza_price
create table pizza_price(
	pizza_id varchar(50) ,
	pizza_type_id varchar(50) ,
	size varchar(10) ,
	price float 
    ) ;
select * from pizza_price ;
select * from order_details ;
select * from order_date ;
select * from pizza_type ;


-- 1.Retrieve the total number of orders placed.
select count(order_id) as `No. of order` from order_date ; 

-- 2.Calculate the total revenue generated from pizza sales.
select round(sum(quantity * price),2) as total_sales 
from order_details join pizza_price
on pizza_price.pizza_id = order_details.pizza_id ;

-- 3.Identify the highest-priced pizza.
select name, price
from pizza_type join pizza_price
on pizza_price.pizza_type_id = pizza_type.pizza_type_id 
order by price desc limit 1 ;

-- 4.Identify the most common pizza quantity ordered.
select quantity,count(quantity) number_of_order from order_details group by 1 with rollup order by 2  desc ;

-- 5.Identify the most common pizza size ordered.
select size, count(size) number_of_order 
from pizza_price join order_details
on order_details.pizza_id = pizza_price.pizza_id 
group by 1  with rollup order by 2 desc ; 

-- 6.List the top 5 most ordered pizza types along with their quantities.
select name, sum(quantity) quantities
from pizza_type join pizza_price
on pizza_type.pizza_type_id = pizza_price.pizza_type_id join order_details 
on order_details.pizza_id = pizza_price.pizza_id group by 1 order by 2 desc  limit 5  ;

-- 7.Find the total quantity of each pizza category ordered.
select category, sum(quantity) quantities from pizza_type join pizza_price
on pizza_type.pizza_type_id = pizza_price.pizza_type_id join order_details 
on order_details.pizza_id = pizza_price.pizza_id group by 1 with rollup order by 2 desc  ;

-- 8.Determine the distribution of orders by hour of the day.
select hour(time) hours,sum(quantity) number_of_pizza_order from order_date join order_details
on order_details.order_id = order_date.order_id group by 1 with rollup order by 2 desc ;

-- 9.Find the category-wise distribution of pizzas.
select * from pizza_type ;
select category,count(name) type_of_pizza
from pizza_type group by 1 with rollup order by 2 desc ;

-- 10.Group the orders by date and calculate the average number of pizzas ordered per day.
with cte_01 as (
select date, sum(quantity) quantity from order_details join order_date
on order_date.order_id = order_details.order_id 
group by 1 order by 2 desc )

select round(avg(quantity),2) as avg_quantity_per_day from cte_01 ;

-- 11.Determine the top 3 most ordered pizza types based on revenue.
select round(sum((price * quantity)),2) revenue , name 
from order_details join pizza_price
on pizza_price.pizza_id = order_details.pizza_id
join pizza_type
on pizza_type.pizza_type_id = pizza_price.pizza_type_id 
group by 2 with rollup order by 1 desc limit 4 ;


-- 12.Calculate the percentage contribution of each pizza category to total revenue.
with cte_02 as (
select category, round(sum(quantity * price ),2) revenue
from pizza_type join pizza_price
on pizza_type.pizza_type_id = pizza_price.pizza_type_id join order_details 
on order_details.pizza_id = pizza_price.pizza_id group by 1 order by 2 desc 
)
select category, round(sum(quantity * price ) *100 / (select sum(revenue) from cte_02 ),2 )
percentage_each_pizza_category_to_total_revenue
from pizza_type join pizza_price
on pizza_type.pizza_type_id = pizza_price.pizza_type_id
join order_details 
on order_details.pizza_id = pizza_price.pizza_id
group by 1 with rollup order by 2 desc  ;

-- 13.Analyze the cumulative revenue generated over time.
select date, revenue,
round(sum(revenue) over(order by date),2) as cumulative_revenue from
(
select date, round(sum(quantity * price),2) revenue
from order_date join order_details
on order_date.order_id = order_details.order_id
join pizza_price
on  order_details.pizza_id = pizza_price.pizza_id
group by 1
) as sub_query ;

-- 14.Determine the top 3 most ordered pizza types based on revenue for each pizza category.
with cte_05 as
 (
select category, name, revenue,
dense_rank() over(partition by category order by revenue desc) as dens_rank
from
		(
select category, name,
round(sum(quantity * price),2) revenue
from pizza_type join pizza_price
on pizza_type.pizza_type_id = pizza_price.pizza_type_id
join order_details
on order_details.pizza_id = pizza_price.pizza_id
group by 1,2 order by 1 asc 
		)
  sub_query
)
select * from cte_05 where dens_rank <= 3 ;





-- moveing average 4 and 5 day 
select date, revenue,
avg(revenue) over(order by date rows between 3 preceding and current row) as '4_day_Moving_avg' from
(
select date, round(sum(quantity * price),2) revenue
from order_date join order_details
on order_date.order_id = order_details.order_id
join pizza_price
on  order_details.pizza_id = pizza_price.pizza_id
group by 1
) as sub_query ;



