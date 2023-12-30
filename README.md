# 8weeksqlchallenge-3

## Case Study #3: Foodie-Fi

View the case study [here](https://8weeksqlchallenge.com/case-study-3/)

Business Task
Danny and his friends launched a new startup Foodie-Fi and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world.

This case study focuses on using subscription style digital data to answer important business questions on customer journey, payments, and business performances.

Entity Relationship Diagram

![image](https://github.com/alankritm95/8weeksqlchallenge-3/assets/129503746/3c1a5e54-56a7-4a83-a9d7-6911ff08a643)

## Data analysis questions 

### How many customers has Foodie-Fi ever had?

select count(distinct customer_id) from subscriptions;

![image](https://github.com/alankritm95/8weeksqlchallenge-3/assets/129503746/a2b906dd-6382-4ad9-8580-acff7ed5bedb)

### What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value

SELECT count(distinct customer_id) as count,
       month(start_date)
FROM subscriptions s
JOIN plans p on s.plan_id = p.plan_id
where p.plan_id = 0
group by month(start_date)
order by month(start_date) asc;

![image](https://github.com/alankritm95/8weeksqlchallenge-3/assets/129503746/56ceba4b-f75f-44d5-b420-c31ac571a98c)


### What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name

SELECT 
  plans.plan_id,
  plans.plan_name,
  COUNT(sub.customer_id) AS num_of_events
FROM subscriptions AS sub
JOIN plans
  ON sub.plan_id = plans.plan_id
WHERE sub.start_date >= '2021-01-01'
GROUP BY plans.plan_id, plans.plan_name
ORDER BY plans.plan_id;

![image](https://github.com/alankritm95/8weeksqlchallenge-3/assets/129503746/6d44ca69-b350-43ed-be91-1d8c97303d8a)



### What is the customer count and percentage of customers who have churned rounded to 1 decimal place?

SELECT
  COUNT(DISTINCT sub.customer_id) AS churned_customers,
  ROUND(100.0 * COUNT(sub.customer_id)
    / (SELECT COUNT(DISTINCT customer_id) 
    	FROM subscriptions)
  ,1) AS churn_percentage
FROM subscriptions AS sub
JOIN plans
  ON sub.plan_id = plans.plan_id
WHERE plans.plan_id = 4;

![image](https://github.com/alankritm95/8weeksqlchallenge-3/assets/129503746/0c2556a1-31a6-4909-81ca-37cc037d2241)


### How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

with cte as(
select *, lead(plan_id, 1) over(partition by customer_id order by plan_id asc) as next_plan from subscriptions),

cte2 as (
select * from cte where plan_id = 0 and next_plan = 4
)

select count(customer_id) as 'customers_churned',
round(count(customer_id) *100/(select count(distinct customer_id) from subscriptions), 2) as '%customers_churned'
 from cte2;

![image](https://github.com/alankritm95/8weeksqlchallenge-3/assets/129503746/40f22c4f-8363-48a9-8b08-208439261001)


What is the number and percentage of customer plans after their initial free trial?



What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
How many customers have upgraded to an annual plan in 2020?
How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
How many customers downgraded from a pro monthly to a basic monthly plan in 2020?








