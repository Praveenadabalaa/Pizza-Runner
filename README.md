********************Pizza Metrics**********************
1)How many pizzas were ordered?

select count(order_id) as Pizza_count from pizza_runner.customer_orders

2)How many unique customer orders were made?

select count(distinct(customer_id)) as Unique_customers from pizza_runner.customer_orders

for cleaning of data 
create TEMPORARY TABLE temp_customer_orders as (
select order_id,customer_id,pizza_id,
case when
exclusions=' ' then 'null'
when exclusions='null' then 'null'
else exclusions
end exclusions_cleaned,
case when
extras='NaN' then 'null'
when extras='null' then 'null'
when extras=' ' then 'null'
else extras
end extras_cleaned
from pizza_runner.customer_orders)

create temp table temp_runner_orders as (
  select order_id,runner_id,pickup_time,
distance,duration,
case when cancellation='NaN' then 'null'
  when cancellation=' ' then 'null'
  when cancellation='null' then 'null'
  else cancellation 
  end cancellation_cleaned from pizza_runner.runner_orders)
  

3) How many successful orders were delivered by each runner?

select runner_id,count(order_id) as Total_delivery_count from 
pizza_runner.runner_orders
where cancellation<>'Restaurant Cancellation' and cancellation<>'Customer Cancellation'
group by runner_id

4) How many of each type of pizza was delivered?
   
select C.pizza_id,R.runner_id,count(R.order_id) as Total_delivery_count from 
pizza_runner.runner_orders as R
INNER JOIN
pizza_runner.customer_orders AS C
on C.order_id=C.order_id
where cancellation<>'Restaurant Cancellation' and cancellation<>'Customer Cancellation'
group by runner_id,pizza_id

5) How many Vegetarian and Meatlovers were ordered by each customer?

select COUNT(N.pizza_id),C.customer_id,N.pizza_name from 
pizza_runner.customer_orders as C
inNer join 
pizza_runner.pizza_names as N
on 
C.pizza_id=N.pizza_id
GROUP BY C.customer_id,N.pizza_name
ORDER BY C.customer_id ASC

6)What was the maximum number of pizzas delivered in a single order?

SELECT count(C.order_id) AS Total_orders_deliverd, C.order_id  FROM pizza_runner.runner_orders as R
inner join
pizza_runner.customer_orders as C
on R.order_id=C.order_id
WHERE R.CANCELLATION<>'Restaurant Cancellation' AND R.CANCELLATION<> 'Customer Cancellation'
group by C.ORDER_ID
order by count(C.order_id) DESC
LIMIT 1

7)How many pizzas were delivered that had both exclusions and extras?

SELECT count(c.order_id) as both_exclusions_extras
FROM pizza_runner.customer_orders as c 
RIGHT JOIN pizza_runner.runner_orders as r ON c.order_id = r.order_id
WHERE c.exclusions<>'NULL' AND c.exclusions<>' ' AND c.extras<>' ' AND c.extras<>'NAN' AND c.extras<>'NULL' and R.CANCELLATION<>'Restaurant Cancellation' AND R.CANCELLATION<> 'Customer Cancellation'

8)What was the total volume of pizzas ordered for each hour of the day?

select count(order_id),DATEPART(hour,order_time) as Order_time
from pizza_runner.customer_orders
group by DATEPART(hour,order_time)
order by count(order_id) desc

9)What was the volume of orders for each day of the week?

select count(order_id),datepart(weekday,order_time) as Weekday_orders from pizza_runner.customer_orders
group by datepart(weekday,order_time)

select count(order_id),datename(weekday,order_time) as Weekday_orders from pizza_runner.customer_orders
group by datepart(weekday,order_time)

********************************Runner and Customer Experience*********************************

1)How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)?

select count(runner_id),week(registration_date,7) as registration_week from pizza_runner.runners group by registration_week

2)What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

select C.order_id,R.runner_id,C.order_time,R.pickup_time,
AVG(DATEDIFF(Minute,C.order_time,R.pickup_time)) as Timetakentopickup
from pizza_runner.customer_orders as C
inner join
pizza_runner.runner_orders as R
on C.pizza_id=R.pizza_id
group by R.runner_id,
order by R.runner_id

3)What was the average distance travelled for each customer?

SELECT customer_id,avg(distance) as avgtime
FROM pizza_runner.customer_orders as C
inner join 
pizza_runner.runner_orders as R
on c.order_id=r.order_id
group by customer_id

4)What was the difference between the longest and shortest delivery times for all orders?

select max(duration)-min(duration) from pizza_runner.runner_orders

5)What was the average speed for each runner for each delivery and do you notice any trend for these values?

select runner_id,duration,avg(duration) as Avg_speed
from runner_orders
group by runner_id

6)What is the successful delivery percentage for each runner?

;with cte as
(
select runner_id,case when cancellation="Restaurant Cancellation" or 
cancellation="Customer Cancellation" then 1
  else 0 as cancellation_orders,
  case when cancellation=null or cancellation="NaN" then 1 else 0
  as "No_cancellation_orders"
  from runner_orders)
  select sum(No_cancellation_orders)/(sum(No_cancellation_orders)+sum(No_cancellation_orders)) as Percentofordersdeliverd from cte
  group by runner_id
