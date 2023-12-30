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


### What is the number and percentage of customer plans after their initial free trial?

SELECT plan_name,
       count(customer_id) customer_count,
       round(100 *count(DISTINCT customer_id) /
               (SELECT count(DISTINCT customer_id) 
                FROM subscriptions), 2) AS 'customer percentage'
FROM subscriptions s
JOIN plans p on s.plan_id = p.plan_id
WHERE plan_name != 'trial'
GROUP BY plan_name;

![image](https://github.com/alankritm95/8weeksqlchallenge-3/assets/129503746/20d17759-f7cd-4b6e-8b06-c1ce6cbcc95a)


### What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

with cte AS (
  SELECT
    customer_id,
    plan_id,
  	start_date,
    LEAD(start_date) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS next_date
  FROM subscriptions
  WHERE start_date <= '2020-12-31'
)

SELECT
	plan_id, 
	COUNT(DISTINCT customer_id) AS customers,
  ROUND(100.0 * 
    COUNT(DISTINCT customer_id)
    / (SELECT COUNT(DISTINCT customer_id) 
      FROM subscriptions)
  ,1) AS percentage
FROM cte
WHERE next_date IS NULL
GROUP BY plan_id;

![image](https://github.com/alankritm95/8weeksqlchallenge-3/assets/129503746/94a075b5-6fd3-4b9f-bead-5895b0aa6ac2)


How many customers have upgraded to an annual plan in 2020?

with cte AS (
  SELECT
    customer_id,
    plan_id,
    LEAD(plan_id) OVER (
      PARTITION BY customer_id
      ORDER BY start_date
    ) AS next_plan
  FROM subscriptions
  WHERE start_date <= '2020-12-31'
)

SELECT
	next_plan, 
	COUNT(DISTINCT customer_id) AS customers,
  ROUND(100.0 * 
    COUNT(DISTINCT customer_id)
    / (SELECT COUNT(DISTINCT customer_id) 
      FROM subscriptions)
  ,1) AS percentage
FROM cte
WHERE next_plan = 3
group by next_plan;


![image](https://github.com/alankritm95/8weeksqlchallenge-3/assets/129503746/00df82ad-1dc8-4c7c-b322-0e520f6387bf)

How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?

WITH trial_plan AS (
  SELECT 
    customer_id, 
    start_date AS trial_date
  FROM subscriptions
  WHERE plan_id = 0
), annual_plan AS (
  SELECT 
    customer_id, 
    start_date AS annual_date
  FROM subscriptions
  WHERE plan_id = 3
)
SELECT 
  ROUND(
    AVG(
      annual.annual_date - trial.trial_date)
  ,0) AS avg_days_to_upgrade
FROM trial_plan AS trial
JOIN annual_plan AS annual
  ON trial.customer_id = annual.customer_id;

![image](https://github.com/alankritm95/8weeksqlchallenge-3/assets/129503746/c51c7eff-c156-4ed5-8bce-6c92e2dfee48)


Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

WITH next_plan_cte AS
  (SELECT *,
          lead(start_date, 1) over(PARTITION BY customer_id
                                   ORDER BY start_date) AS next_plan_start_date,
          lead(plan_id, 1) over(PARTITION BY customer_id
                                ORDER BY start_date) AS next_plan
   FROM subscriptions),
     window_details_cte AS
  (SELECT *,
          datediff(next_plan_start_date, start_date) AS days,
          round(datediff(next_plan_start_date, start_date)/30) AS window_30_days
   FROM next_plan_cte
   WHERE next_plan=3)
SELECT window_30_days,
       count(*) AS customer_count
FROM window_details_cte
GROUP BY window_30_days
ORDER BY window_30_days;


![image](https://github.com/alankritm95/8weeksqlchallenge-3/assets/129503746/af2e844d-4a93-47f8-9ec6-a4c9b5d33ee8)









