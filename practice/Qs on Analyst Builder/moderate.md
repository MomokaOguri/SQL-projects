# Moderate

## Tech Layoffs

Tech companies have been laying off employees after a large surge of hires in the past few years.

Write a query to determine the percentage of employees that were laid off from each company.

Output should include the company and the percentage (to 2 decimal places) of laid off employees.

Order by company name alphabetically.

```sql
-- OUTPUT: company name, %laid off (round 2dp)
-- CALC: %laid off = employees_fired / company_size * 100
-- ORDER BY: company ASC

SELECT company, 
  ROUND(employees_fired / company_size * 100, 2) as percentage_laid_off
FROM tech_layoffs 
ORDER BY company ASC;
```

## Separation

Data was input incorrectly into the database. The ID was combined with the First Name.

Write a query to separate the ID and First Name into two separate columns.

Each ID is 5 characters long.

- data
    
    | id |
    | --- |
    | 12345Johnny |
    | 93829Sally |
    | 20391Larry |
    | 29324Valerie |
    

```sql
-- SUBSTRING - to pull out numbers and then names
-- OUTPUT: ID, first name

SELECT
  SUBSTRING(id,1,5) as new_id, 
  SUBSTRING(id,6) as name
FROM bad_data;
```

## Senior Citizen Discount

If a customer is 55 or above they qualify for the senior citizen discount. Check which customers qualify.

Assume the current date 1/1/2023.

Return all of the Customer IDs who qualify for the senior citizen discount in ascending order.

```sql
-- OUTPUT: id ASC
-- FITLER: age >= 55

SELECT customer_id 
FROM customers
WHERE TIMESTAMPDIFF(YEAR, birth_date, '2023-01-01') >= 55
ORDER BY customer_id ASC;

/*
SELECT customer_id 
FROM customers
WHERE birth_date <= '1968-01-01'
ORDER BY customer_id ASC;
*/
```

- `TIMESTAMPDIFF(unit, date1, date2)`
    - unit: FRAC_SECOND (microseconds), SECOND, MINUTE, HOUR, DAY, WEEK, MONTH, QUARTER, or YEAR

[MySQL TIMESTAMPDIFF() function - w3resource](https://www.w3resource.com/mysql/date-and-time-functions/mysql-timestampdiff-function.php)

## LinkedIn Famous

Write a query to determine the popularity of a post on LinkedIn

Popularity is defined by number of actions (likes, comments, shares, etc.) divided by the number impressions the post received * 100.

If the post receives a score higher than 1 it was very popular.

Return all the post IDs and their popularity where the score is 1 or greater.

Order popularity from highest to lowest.

solution

```sql
SELECT post_id, (actions / impressions * 100) as popularity
FROM linkedin_posts
WHERE (actions /impressions * 100) >= 1.0
ORDER BY popularity DESC;
```

mine

```sql
-- OUTPUT: post_id, score DESC
-- score = num_actions / num_impression * 100
-- FILTER: score >= 1 (very popular)

SELECT post_id, actions / impressions * 100 as score
FROM linkedin_posts
HAVING score >= 1
ORDER BY score DESC;
```

## Company Wide Increase

If our company hits its yearly targets, every employee receives a salary increase depending on what level you are in the company.

Give each Employee who is a level 1 a 10% increase, level 2 a 15% increase, and level 3 a 200% increase.

Include this new column in your output as "new_salary" along with your other columns.

```sql
-- OUTPUT: *, new_salary
-- new_salary =
-- salary * 1.1, if pay_level = 1
-- salary * 1.15, if pay_level = 2
-- salary * 3, if pay_level = 3

SELECT *, 
CASE
  WHEN pay_level = 1 THEN salary * 1.1
  WHEN pay_level = 2 THEN salary * 1.15
  WHEN pay_level = 3 THEN salary * 3
END as new_salary
FROM employees;
```

## Media Addicts

Social Media Addiction can be a crippling disease affecting millions every year.

We need to identify people who may fall into that category.

Write a query to find the people who spent a higher than average amount of time on social media.

Provide just their first names alphabetically so we can reach out to them individually.

- data
    
    user_time
    
    | user_id | media_time_minutes |
    | --- | --- |
    | 1 | 0 |
    | 2 | 200 |
    | 3 | 250 |
    | 4 | 15 |
    | 5 | 500 |
    | 6 | 45 |
    | 7 | 450 |
    | 8 | 1000 |
    | 9 | 300 |
    | 10 | 60 |
    
    users
    
    | user_id | first_name |
    | --- | --- |
    | 1 | John |
    | 2 | Janice |
    | 3 | Michael |
    | 4 | Molly |
    | 5 | Adam |
    | 6 | Amanda |
    | 7 | Chris |
    | 8 | Christine |
    | 9 | Bella |
    | 10 | Brian |

```sql
-- OUTPUT: first name ASC
-- FILTER: higher than avg

-- 1. get avg_time
-- SELECT AVG(media_time_minutes)
-- FROM user_time;

-- 2. get user_id with media_time_minutes > avg_time
-- SELECT user_id
-- FROM user_time
-- WHERE media_time_minutes > 
--   (SELECT AVG(media_time_minutes) FROM user_time)
-- ;

-- 3. get first_name with user_id in filtered user_id
SELECT first_name
FROM users
WHERE user_id IN (
  SELECT user_id
  FROM user_time
  WHERE media_time_minutes > (
    SELECT AVG(media_time_minutes) 
    FROM user_time)
  )
ORDER BY first_name ASC;
```

- use multiple subqueries
- compare with subquery which compares with another subquery

old idea - doesn’t work

```sql
WITH CTE_avg_time AS (
  SELECT *, AVG(media_time_minutes) OVER() as avg_time
  FROM user_time
  JOIN users ON user_time.user_id = users.user_id
)
SELECT first_name
FROM CTE_avg_time
WHERE media_time_minutes > avg_time
ORDER BY first_name;
```

## Bike Price

Sarah's Bike Shop sells a lot of bikes and wants to know what the average sale price is of her bikes.

She sometimes gives away a bike for free for a charity event and if she does she leaves the price of the bike as blank, but marks it sold.

Write a query to show her the average sale price of bikes for only bikes that were sold, and not donated.

Round answer to 2 decimal places.

```sql
-- OUTPUT: avg sale price for sold bikes (round 2dp)
-- FILTER: sold bikes if bike_price IS NOT NULL & bike_sold = Y

-- get sold bikes
SELECT ROUND(AVG(bike_price),2)
FROM inventory
WHERE bike_price IS NOT NULL
AND bike_sold = 'Y';
```

## Direct Reports

Write a query to determine how many direct reports each Manager has.

*Note: Managers will have "Manager" in their title.*

Report the Manager ID, Manager Title, and the number of direct reports in your output.

Hint:

- We need to join the table back to itself where the managers_id = employee_id. This will help us tie employees to the managers.
- Now that we can filter down to just the employees who have 'Manager' in their title.
- Lastly, we can group on the employee id and position and perform an aggregate function to get the count of their employees.

- data
    
    
    | employee_id | position | managers_id |
    | --- | --- | --- |
    | 1001 | Analytics Manager | 1013 |
    | 1002 | Data Engineer | 1007 |
    | 1003 | Data Engineer | 1001 |
    | 1004 | Database Developer | 1017 |
    | 1005 | Data Analyst | 1001 |
    | 1006 | Data Engineer | 1017 |
    | 1007 | Data Engineering Manager | 1013 |
    | 1008 | Database Developer | 1001 |
    | 1009 | Data Engineer | 1007 |
    | 1010 | Data Scientist | 1017 |
    | 1011 | Data Analyst | 1001 |
    | 1012 | Data Engineer | 1007 |
    | 1013 | CTO | NULL |
    | 1014 | Data Scientist | 1017 |
    | 1015 | Data Analyst | 1001 |
    | 1016 | Data Scientist | 1017 |
    | 1017 | Data Science Manager | 1013 |
    | 1018 | Database Developer | 1007 |
    | 1019 | Data Analyst | 1001 |

solution

```sql
SELECT m.employee_id AS manager_id, 
	m.position AS manager_position, 
	COUNT(*) AS direct_reports
FROM direct_reports e
JOIN direct_reports m ON e.managers_id = m.employee_id
WHERE m.position LIKE '%Manager%'
GROUP BY m.employee_id, m.position;
```

- `count(*)`: number of rows in table
- `count(*)` with `GROUP BY`: returns number of rows in each group

mine #1 (before hints)

```sql
-- OUTPUT: manager id, manager title, num_direct_reports
-- number of direct reports = count of employees by managers_id - GROUP BY
-- FILTER: 'Manager' in title - position LIKE '%Manager%'

-- 1. list of employee_id of managers
SELECT employee_id 
FROM direct_reports
WHERE position LIKE '%Manager%';

-- 2. count of direct reports per managers_id
SELECT managers_id, COUNT(employee_id) as num_direct_reports
FROM direct_reports
GROUP BY managers_id;

-- 3. filter above step2 by comparing to list of managers' employee_id
SELECT managers_id, COUNT(employee_id) as num_direct_reports
FROM direct_reports
WHERE managers_id IN (
  SELECT employee_id 
  FROM direct_reports
  WHERE position LIKE '%Manager%'
  )
GROUP BY managers_id
;
```

mine #2 (after hints)

```sql
-- SELF JOIN to link managers_id = employee_id
-- SELECT 
--   e.employee_id, 
--   e.position,
--   e.managers_id,
--   m.employee_id as manager_id, 
--   m.position as manager_title
-- FROM direct_reports e
-- JOIN direct_reports m ON e.managers_id = m.employee_id;
  

SELECT 
  m.employee_id as manager_id, 
  m.position as manager_title,
  COUNT(e.employee_id) as num_direct_reports
FROM direct_reports e
JOIN direct_reports m ON e.managers_id = m.employee_id
WHERE m.position LIKE '%Manager%'
GROUP BY manager_id, manager_title;
```


## Food Divides Us

In the United States, fast food is the cornerstone of its very society. Without it, it would cease to exist.

But which region spends the most money on fast food?

Write a query to determine which region spends the most amount of money on fast food.

```sql
-- OUTPUT: region; limit 1
-- Calc: sum($)
-- GROUP BY: region
-- ORDER BY: $ DESC

/*
SELECT region, sum(fast_food_millions) 
FROM food_regions
GROUP BY region
ORDER BY sum(fast_food_millions) DESC
LIMIT 10;
*/

SELECT region
FROM food_regions
GROUP BY region
ORDER BY sum(fast_food_millions) DESC
LIMIT 1;
```

solution (using CTE, to keep ‘total_spending’)

```sql
WITH food_regions_grouped AS (
SELECT region, SUM(fast_food_millions) AS total_spending
FROM food_regions
GROUP BY region
ORDER BY total_spending DESC 
LIMIT 1
  )
SELECT region
  FROM food_regions_grouped;
```

## Kroger’s Members

solution

```sql
SELECT
    ROUND(COUNT(CASE WHEN has_member_card = 'Y' THEN 1 END) / COUNT(*) * 100,2) AS Percentage_with_Membership
FROM customers
WHERE kroger_id IS NOT NULL;
```

- Case statement
- subquery method is also fine

mine

```sql
-- OUTPUT: % customers with member card
-- calc: %member = num_member / num_customers
-- num_customers = count(*)
-- subquery to find num_member -- WHERE has_member_card = 'Y'

-- 1. num_customers
-- SELECT count(*)
-- FROM customers

-- 2. subquery: num_member
-- SELECT count(*)
-- FROM customers
-- WHERE has_member_card = 'Y';

SELECT 
	ROUND((SELECT count(*)
		FROM customers
		WHERE has_member_card = 'Y') / count(*) * 100, 2) as percentage_member
FROM customers;

-- solution
SELECT
    ROUND(COUNT(CASE WHEN has_member_card = 'Y' THEN 1 END) 
    / COUNT(kroger_id) * 100, 2) AS Percentage_with_Membership
FROM customers;
```

- can use SUM() or COUNT() for (CASE …)

## TMI (Too Much Information)

Often when you're working with customer information you'll want to sell that data to a third party. Sometimes it is illegal to give away sensitive information such as a full name.

Here you are given a table that contains a customer ID and their full name.

Return the customer ID with only the first name of each customer.

solution

```sql
SELECT Customer_ID, TRIM(SUBSTRING_INDEX(Full_Name, ' ', 1)) AS First_Name
FROM customers;
```

- `SUBSTRING_INDEX(str, delim, count)`

mine

```sql
-- OUTPUT: id, first name
-- substring by space on full_name to get first name

SELECT customer_id, SUBSTRING(full_name, 1, LOCATE(' ', full_name)) as first_name
FROM customers;
```
