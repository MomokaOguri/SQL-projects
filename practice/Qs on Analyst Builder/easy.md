# Easy Questions

## Car Failure

Cars need to be inspected every year in order to pass inspection and be street legal. If a car has any critical issues it will fail inspection or if it has more than 3 minor issues it will also fail.

Write a query to identify all of the cars that passed inspection.

Output should include the owner name and vehicle name. Order by the owner name alphabetically.

```sql
-- FILTER: 0 critical issues AND <= 3 minor issues
-- OUTPUT: Owner Name, Vehicle Name
-- ORDER BY Owner Name ASC

SELECT owner_name, vehicle FROM inspections
  WHERE critical_issues = 0
  AND minor_issues <= 3
  ORDER BY owner_name ASC;
```

## Apply Discount

A Computer store is offering a 25% discount for all new customers over the age of 65 or customers that spend more than $200 on their first purchase.

The owner wants to know how many customers received that discount since they started the promotion.

Write a query to see how many customers received that discount.

```sql
-- OUTPUT: Count on # of filtered customers
-- FILTER: age > 65 OR total_purchase > 200
-- all new customers in data

SELECT COUNT(customer_id)
FROM customers
WHERE age > 65
OR total_purchase > 200;
```

## Tesla Models

Tesla just provided their quarterly sales for their major vehicles.

Determine which Tesla Model has made the most profit.

Include all columns with the "profit" column at the end.

```sql
-- OUTPUT: all col, profit
-- CALC: profit = cars_sold * (car_price - production_cost)
-- FILTER: hihgest profit (order DESC & limit 1)

SELECT *, 
(cars_sold * (car_price - production_cost)) AS profit
FROM tesla_models
ORDER BY profit DESC
LIMIT 1;
```

## Heart Attack Risk

Dr. Obrien has seen an uptick in heart attacks for his patients over the past few months. He has been noticing some trends across his patients and wants to get ahead of things by reaching out to current patients who are at a high risk of a heart attack.

We need to identify which clients he needs to reach out to and provide that information to Dr. Obrien.

If a patient is over the age of 50, cholesterol level of 240 or over, and weight 200 or greater, then they are at high risk of having a heart attack.

Write a query to retrieve these patients. Include all columns in your output.

As Cholesterol level is the largest indicator, order the output by Cholesterol from Highest to Lowest so he can reach out to them first.

```sql
-- FILTER: age > 50, cholesterol >= 240, weight >= 200, AND
-- OUTPUT: *
-- ORDER BY: cholesterol DESC

SELECT * 
FROM patients
WHERE age > 50
  AND cholesterol >= 240
  AND weight >= 200
ORDER BY cholesterol DESC;
```

## Million Dollar Store

Write a query that returns all of the stores whose average yearly revenue is greater than one million dollars.

Output the store ID and average revenue. Round the average to 2 decimal places.

Order by store ID.

- data
    
    
    | store_id | year | revenue |
    | --- | --- | --- |
    | 1 | 2020 | 1000000 |
    | 2 | 2020 | 1500000 |
    | 3 | 2020 | 800000 |
    | 4 | 2020 | 180000 |
    | 1 | 2021 | 2000000 |
    | 2 | 2021 | 1800000 |
    | 3 | 2021 | 1000000 |
    | 4 | 2021 | 900000 |
    | 1 | 2022 | 700000 |
    | 2 | 2022 | 2000000 |
    | 3 | 2022 | 600000 |
    | 4 | 2022 | 1300000 |

```sql
-- FILTER: avg rev > 1 mill
-- OUTPUT: store ID, avg rev rounded to 2dp
-- ORDER BY: store ID

SELECT store_id, ROUND(AVG(revenue),2) as avg_yr_rev
FROM stores
GROUP BY store_id
HAVING avg_yr_rev > 1000000
ORDER BY store_id;

-- <<MISTAKE>>
-- WITH CTE_avg_yr_rev AS (
--   SELECT store_id, ROUND(AVG(revenue),2) as avg_yr_rev
--   FROM stores
--   GROUP BY store_id
--   ORDER BY store_id
-- )
-- SELECT store_id, avg_yr_rev
-- FROM CTE_avg_yr_rev
-- WHERE avg_yr_rev > 1000000;
```

- use `HAVING` for filtering aggregate functions after `GROUP BY`
- round number by x decimal places: `ROUND(col_name, x)`

## Low Quality YouTube Video

Write a query to report the IDs of low quality YouTube videos.

A video is considered low quality if the like percentage of the video (number of likes divided by the total number of votes) is less than 55%.

Return the result table ordered by ID in ascending order.

```sql
-- OUTPUT: id
-- FILTER: % < 55
-- ORDER BY: id ASC

SELECT video_id
FROM youtube_videos
WHERE thumbs_up/(thumbs_up + thumbs_down) < 0.55
ORDER BY video_id ASC;
```

## Chocolate

I love chocolate and only want delicious baked goods that have chocolate in them!

Write a Query to return bakery items that contain the word "Chocolate".

```sql
-- FILTER: product_name contains "Chocolate"

SELECT * 
FROM bakery_items
WHERE product_name LIKE '%Chocolate%';
```

- filter text column that contains specific string using `LIKE` and `%`

## On The Way Out

Herschel's Manufacturing Plant has hit some hard times with the economy and unfortunately they need to let some people go.

They figure the younger employees need their jobs more as they are growing families so they decide to let go of their 3 oldest employees. They have more experience and will be able to land on their feet easier (and they had to pay them more).

Write a query to identify the ids of the three oldest employees.

Order output from oldest to youngest.

```sql
-- OUTPUT: id
-- FILTER: 3 oldest
-- ORDER BY: birth_date ASC

SELECT employee_id
FROM employees
ORDER BY birth_date ASC
LIMIT 3;
```

## Sandwich Generation

Yan is a sandwich enthusiast and is determined to try every combination of sandwich possible. He wants to start with every combination of bread and meats and then move on from there, but he wants to do it in a systematic way.

Below we have 2 tables, bread and meats

Output every possible combination of bread and meats to help Yan in his endeavors.

Order by the bread and then meat alphabetically. This is what Yan prefers.

- data
    
    bread_table
    
    | bread_id | bread_name |
    | --- | --- |
    | 1 | Whole Wheat |
    | 2 | White |
    | 3 | Sourdough |
    | 4 | Brioche |
    
    meat_table
    
    | meat_id | meat_name |
    | --- | --- |
    | 1 | Turkey |
    | 2 | Ham |
    | 3 | Roast Beef |
    | 4 | Pastrami |
    | 5 | Salami |
    | 6 | Chicken |
    | 7 | Bacon |

solution

```sql
SELECT bread_name, meat_name
FROM meat_table
CROSS JOIN bread_table
ORDER BY bread_name, meat_name;
```

mine

```sql
-- OUTPUT: bread_name, meat_name
-- JOIN bread_table & meat_table
-- ORDER BY bread ASC, then meat ASC
-- expect 7 meat * 4 bread = 28 comb

SELECT bread_name, meat_name
FROM bread_table
JOIN meat_table
ORDER BY bread_name ASC, meat_name ASC;
```

- use CROSS JOIN to have all records from both tables

## Electric Bike Replacement

After about 10,000 miles, Electric bike batteries begin to degrade and need to be replaced.

Write a query to determine the amount of bikes that currently need to be replaced.

```sql
-- OUTPUT: count of filtered bikes
-- FILTER: miles > 10000

SELECT COUNT(bike_id)
FROM bikes
WHERE miles > 10000;
```

## Perfect Data Analyst

Return all the candidate IDs that have problem solving skills, SQL experience, knows Python or R, and has domain knowledge.

Order output on IDs from smallest to largest.

```sql
-- OUTPUT: cand ID
-- FILTER: has 
-- problem_solving AND 
-- sql_experience AND
-- (python OR r_programming) AND
-- domain_knowledge
-- ORDER BY: id ASC

SELECT candidate_id
FROM candidates
WHERE problem_solving = 'X' 
  AND sql_experience  = 'X'
  AND (python = 'X' OR r_programming = 'X') 
  AND domain_knowledge = 'X'
ORDER BY candidate_id ASC;
```

## Costco Rotisserie Loss

Costco is known for their rotisserie chickens they sell, not just because they are delicious, but because they are a loss leader in this area.

This means they actually lose money in selling the chickens, but they are okay with this because they make up for that in other areas.

Using the sales table, calculate how much money they have lost on their rotisserie chickens this year. Round to the nearest whole number.

```sql
-- OUTPUT: total loss on chicken, rounded to whole number

SELECT ROUND(SUM(lost_revenue_millions))
FROM sales;
```
