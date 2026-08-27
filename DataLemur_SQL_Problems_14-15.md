# DataLemur SQL Practice --- Problems 14--15

Today's practice focused on MySQL date handling, choosing the correct
JOIN, preserving unmatched rows with `LEFT JOIN`, and breaking
multi-stage SQL problems into smaller steps.

------------------------------------------------------------------------

## 14. Second Day Confirmation

### Problem

Find the user IDs of TikTok users who confirmed their account exactly
one day after signing up.

### MySQL Solution

``` sql
SELECT user_id
FROM emails
JOIN texts
    ON emails.email_id = texts.email_id
WHERE action_date = DATE_ADD(signup_date, INTERVAL 1 DAY)
  AND signup_action = 'Confirmed';
```

### What I Learned

I knew the tables needed to be joined using `email_id` and that the
confirmation had to happen on the next day. The syntax I was missing was
how to add one day to a date in MySQL:

``` sql
DATE_ADD(signup_date, INTERVAL 1 DAY)
```

Using only `action_date != signup_date` would also allow confirmations
several days later, so the comparison needs to specify exactly one day.

**Gap identified:** Syntax recall / new date syntax.

------------------------------------------------------------------------

## 15. IBM Db2 Queries Histogram

### Problem

Create a histogram showing how many employees executed 0, 1, 2, 3, etc.
unique queries during Q3 2023, including employees who executed no
queries.

### MySQL Solution

``` sql
WITH employee_queries AS (
    SELECT
        emp.employee_id,
        COUNT(DISTINCT que.query_id) AS unique_queries
    FROM employees AS emp
    LEFT JOIN queries AS que
        ON emp.employee_id = que.employee_id
       AND que.query_starttime >= '2023-07-01'
       AND que.query_starttime < '2023-10-01'
    GROUP BY emp.employee_id
)

SELECT
    unique_queries,
    COUNT(*) AS employee_count
FROM employee_queries
GROUP BY unique_queries
ORDER BY unique_queries;
```

### My Initial Attempt

I knew the two tables needed to be joined, but I initially placed
`queries` on the left:

``` sql
SELECT *
FROM queries AS que
LEFT JOIN employees AS emp
    ON que.employee_id = emp.employee_id;
```

This would preserve all query rows, not all employees. The problem
specifically requires employees with **zero queries**, so `employees`
needs to be the preserved table.

### What I Learned

#### Choosing the JOIN

Ask:

> Which rows must survive even when there is no match?

Because every employee must remain in the result:

``` sql
FROM employees AS emp
LEFT JOIN queries AS que
    ON emp.employee_id = que.employee_id
```

Mental model:

-   `INNER JOIN` → keep only matches.
-   `LEFT JOIN` → keep every row from the left table.

#### Filtering Q3 2023

A reliable timestamp range is:

``` sql
query_starttime >= '2023-07-01'
AND query_starttime < '2023-10-01'
```

This includes July, August, and September while excluding October.

#### Filtering the right side of a LEFT JOIN

The date condition belongs in the `ON` clause for this problem:

``` sql
LEFT JOIN queries AS que
    ON emp.employee_id = que.employee_id
   AND que.query_starttime >= '2023-07-01'
   AND que.query_starttime < '2023-10-01'
```

Putting the query-date filter in `WHERE` could remove employees whose
joined query values are `NULL`, which would lose the zero-query
employees.

#### Two-stage aggregation

First calculate queries per employee:

``` text
employee_id | unique_queries
------------|---------------
1           | 0
2           | 0
3           | 1
4           | 1
5           | 2
```

Then group by that count:

``` text
unique_queries | employee_count
---------------|---------------
0              | 2
1              | 2
2              | 1
```

A CTE separates these two stages cleanly.

**Gaps identified:** JOIN reasoning + multi-stage aggregation.

------------------------------------------------------------------------

## MySQL Date Patterns Reviewed Today

``` sql
-- Extract parts
YEAR(date_column)
MONTH(date_column)
DAY(date_column)

-- Calendar date only
DATE(datetime_column)

-- Add/subtract time
DATE_ADD(date_column, INTERVAL 1 DAY)
DATE_SUB(date_column, INTERVAL 1 DAY)

-- Difference in days
DATEDIFF(end_date, start_date)

-- Timestamp range
date_column >= '2023-07-01'
AND date_column < '2023-10-01'

-- Year-month
DATE_FORMAT(date_column, '%Y-%m')
```

  Requirement         MySQL Pattern
  ------------------- -----------------------------------------
  In 2023             `YEAR(date_column) = 2023`
  In August           `MONTH(date_column) = 8`
  Next day            `DATE_ADD(date_column, INTERVAL 1 DAY)`
  Previous day        `DATE_SUB(date_column, INTERVAL 1 DAY)`
  Days between        `DATEDIFF(end_date, start_date)`
  Same calendar day   `DATE(date1) = DATE(date2)`
  Q3 2023             `>= '2023-07-01' AND < '2023-10-01'`

------------------------------------------------------------------------

## Key Patterns Learned Today

### Exact next-day comparison

``` sql
WHERE action_date = DATE_ADD(signup_date, INTERVAL 1 DAY)
```

### Preserve records with no match

``` sql
FROM main_table
LEFT JOIN optional_table
    ON main_table.id = optional_table.id
```

### Preserve unmatched rows while filtering matched activity

``` sql
LEFT JOIN queries
    ON employees.employee_id = queries.employee_id
   AND queries.query_starttime >= '2023-07-01'
   AND queries.query_starttime < '2023-10-01'
```

### Histogram / aggregation of an aggregation

``` text
1. Calculate a value per employee.
2. Group employees by that calculated value.
3. Count employees in each group.
```

------------------------------------------------------------------------

## Today's Main Takeaway

Problem 14 mainly exposed a **date-syntax gap**: the logic was clear,
but `DATE_ADD()` was new.

Problem 15 was harder because it combined `LEFT JOIN`, date filtering,
preserving zero-query employees, aggregation, a CTE, and a second
aggregation.

The most reusable JOIN question from today is:

> **Which records must survive even when there is no match?**
