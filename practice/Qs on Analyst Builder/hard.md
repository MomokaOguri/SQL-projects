# Hard

## Kelly’s 3rd Purchase

At Kelly's Ice Cream Shop, Kelly gives a 33% discount on each customer's 3rd purchase.

Write a query to select the 3rd transaction for each customer that received that discount. Output the customer id, transaction id, amount, and the amount after the discount as "discounted_amount".

Order output on **customer ID** in ascending order.

*Note: Transaction IDs occur sequentially. The lowest transaction ID is the earliest ID.*

- data
    
    | customer_id | transaction_id | amount |
    | --- | --- | --- |
    | 1001 | 339473 | 89 |
    | 1002 | 359433 | 5 |
    | 1003 | 43176 | 52 |
    | 1004 | 27169 | 19 |
    | 1001 | 530588 | 4 |
    | 1004 | 528902 | 78 |
    | 1005 | 584167 | 72 |
    | 1003 | 55479 | 45 |
    | 1005 | 500607 | 98 |
    | 1004 | 544617 | 65 |
    | 1001 | 374711 | 94 |
    | 1002 | 328456 | 42 |
    | 1005 | 412764 | 43 |
    | 1001 | 225602 | 19 |
    | 1004 | 642498 | 55 |
    | 1002 | 415562 | 50 |
    | 1005 | 272319 | 78 |
    | 1001 | 445346 | 92 |
    | 1002 | 458215 | 30 |
    | 1004 | 173711 | 91 |
    | 1003 | 102487 | 39 |
    | 1005 | 566617 | 58 |

solution (subquery)

```sql
SELECT customer_id, transaction_id, amount, amount * 0.67 AS discounted_amount
FROM (
    SELECT customer_id, transaction_id, amount, 
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY transaction_id) AS rn
    FROM purchases
) t
WHERE rn = 3
ORDER BY customer_id ASC;
```

- `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()` would give the same result for this case
    - but `ROW_NUMBER()` makes the most sense here — transID are all unique (no dupe)
    - `ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY transaction_id) AS rn`
    - `RANK() OVER(PARTITION BY customer_id ORDER BY transaction_id) as trans_rank`
- can use subquery, CTE, temp table, etc.

mine (CTE)

```sql
-- OUTPUT: customer id (ASC), transaction id, amount, discounted_amount
-- FILTER: 3rd transaction for each customer
-- rank & order by trans_id
-- Calc: discounted amount = amount * (1-0.33)
-- ORDER BY: customer_id ASC

-- SELECT *,
--   RANK() OVER(PARTITION BY customer_id ORDER BY transaction_id)
-- FROM purchases;

WITH CTE_rank AS (
  SELECT *,
  RANK() OVER(PARTITION BY customer_id ORDER BY transaction_id) as trans_rank
  FROM purchases
)
SELECT customer_id, transaction_id, amount, 
  amount * (1-0.33) as discounted_amount
FROM CTE_rank
WHERE trans_rank = 3
ORDER BY customer_id ASC;
```

## Temperature Fluctuations

Write a query to find all dates with higher temperatures compared to the previous dates (yesterday).

Order dates in ascending order.

- data
    
    | date | temperature |
    | --- | --- |
    | 2022-01-01 | 65 |
    | 2022-01-02 | 70 |
    | 2022-01-03 | 55 |
    | 2022-01-04 | 58 |
    | 2022-01-05 | 90 |
    | 2022-01-06 | 88 |
    | 2022-01-07 | 76 |
    | 2022-01-08 | 82 |
    | 2022-01-09 | 88 |
    | 2022-01-10 | 72 |

solution

```sql
SELECT t1.date
FROM temperatures t1
JOIN temperatures t2 
	ON DATEDIFF(t1.date, t2.date) = 1 
	AND t1.temperature > t2.temperature
ORDER BY t1.date;
```

- DATEFIFF(date1, date2) to specify the difference of date to 1 day
- include temperature comparison in JOIN conditions as well

mine

```sql
-- OUTPUT: date ASC
-- FILTER: today temp > yesterday temp
-- SELF JOIN with link date = date - 1 day

-- SELECT *, DATE_SUB(date, INTERVAL 1 DAY)
-- FROM temperatures;

SELECT today FROM (
  SELECT t.`date` today, 
    t.temperature today_temp, 
    y.`date` yesterday, 
    y.temperature yesterday_temp
  FROM temperatures t
  JOIN temperatures y 
  ON t.`date` = DATE_ADD(y.`date`, INTERVAL 1 DAY)
) table_join
WHERE today_temp > yesterday_temp
ORDER BY today ASC
;
```

## Cake vs Pie

Marcie's Bakery is having a contest at her store. Whichever dessert sells more each day will be on discount tomorrow. She needs to identify which dessert is selling more.

Write a query to report the difference between the number of Cakes and Pies sold each day.

Output should include the date sold, the difference between cakes and pies, and which one sold more (cake or pie). The difference should be a positive number.

Return the result table ordered by Date_Sold.

- data
    
    
    | date_sold | product | amount_sold |
    | --- | --- | --- |
    | 2022-06-01 | Cake | 6 |
    | 2022-06-01 | Pie | 18 |
    | 2022-06-02 | Pie | 3 |
    | 2022-06-02 | Cake | 2 |
    | 2022-06-03 | Pie | 14 |
    | 2022-06-03 | Cake | 15 |
    | 2022-06-04 | Pie | 15 |
    | 2022-06-04 | Cake | 6 |
    | 2022-06-05 | Cake | 16 |
    | 2022-06-05 | Pie | NULL |
    

solution

```sql
SELECT
    date_sold,
    ABS(SUM(CASE WHEN product = 'Cake' THEN amount_sold ELSE 0 END) -
        SUM(CASE WHEN product = 'Pie' THEN amount_sold ELSE 0 END)) AS difference,
    CASE
        WHEN SUM(CASE WHEN product = 'Cake' THEN amount_sold ELSE 0 END) >
            SUM(CASE WHEN product = 'Pie' THEN amount_sold ELSE 0 END)
        THEN 'Cake'
        ELSE 'Pie'
    END AS sold_more
FROM
    desserts
GROUP BY
    date_sold
ORDER BY
    date_sold;
```

mine

```sql
-- OUTPUT: date sold, 
  -- the difference between cakes and pies (positive num), 
  -- which one sold more (cake or pie)
-- ORDER BY date_sold
-- DATA: one row for date_sold & product, 
  -- 5 dates
  -- NULL in amount_sold for 6-5 pie

-- separate into 2 tables (cakes, pies)
-- self join on same date & higher amount_sold

-- UPDATE desserts
-- SET amount_sold = 0
-- WHERE amount_sold IS NULL;

SELECT t1.date_sold, 
  CASE
    WHEN t2.amount_sold IS NOT NULL THEN t1.amount_sold - t2.amount_sold
    WHEN t2.amount_sold IS NULL THEN t1.amount_sold
  END as difference,
  t1.product
FROM desserts t1
JOIN desserts t2 
  ON t1.date_sold = t2.date_sold
  AND ((t1.amount_sold > t2.amount_sold)
    OR (t1.amount_sold IS NOT NULL AND t2.amount_sold IS NULL))
ORDER BY t1.date_sold;
```
