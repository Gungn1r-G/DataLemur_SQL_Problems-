# DataLemur SQL Practice --- Problems 8--11

A compact record of the DataLemur SQL problems I solved today, with
final solutions and the main concepts I learned.

------------------------------------------------------------------------

## 8. Duplicate Job Listings

### Problem

Find the number of companies that posted duplicate job listings. A
duplicate means the same company posted listings with identical titles
and descriptions.

### Solution

``` sql
WITH duplicates AS (
    SELECT company_id, title, description, COUNT(*)
    FROM job_listings
    GROUP BY company_id, title, description
    HAVING COUNT(*) > 1
)
SELECT COUNT(*) AS duplicate_companies
FROM duplicates;
```

### What I Learned

-   `GROUP BY company_id, title, description` groups identical listings
    within the same company.
-   `HAVING COUNT(*) > 1` keeps duplicate groups.
-   Some problems require an intermediate result before the final
    calculation.
-   A CTE can hold that intermediate result for a second calculation.
-   This reinforced the multi-stage query pattern from the tweet
    histogram problem.

------------------------------------------------------------------------

## 9. Top 3 Cities by Completed Trades

### Problem

Find the three cities with the highest number of completed trade orders.

### Solution

``` sql
SELECT city, COUNT(*) AS total_orders
FROM trades AS td
JOIN users AS us
    ON td.user_id = us.user_id
WHERE status = 'Completed'
GROUP BY city
ORDER BY total_orders DESC
LIMIT 3;
```

### What I Learned

-   I can build a query incrementally instead of solving the entire
    query mentally at once.
-   Trade status comes from `trades`, while city comes from `users`, so
    the tables must be joined using `user_id`.
-   `WHERE` filters completed trades before aggregation.
-   `GROUP BY city` with `COUNT(*)` counts completed trades per city.
-   `ORDER BY ... DESC` and `LIMIT 3` return the top three.
-   I solved this problem independently after identifying the join as
    the first step.

------------------------------------------------------------------------

## 10. Average Review Ratings by Month

### Problem

Calculate the average star rating for each product for each month. Show
the numerical month, product ID, and average rating rounded to two
decimal places. Sort by month and then product ID.

### PostgreSQL Solution

``` sql
SELECT
    EXTRACT(MONTH FROM submit_date) AS mth,
    product_id,
    ROUND(AVG(stars), 2) AS avg_stars
FROM reviews
GROUP BY mth, product_id
ORDER BY mth, product_id;
```

### What I Learned

-   PostgreSQL extracts a numerical month with:

``` sql
EXTRACT(MONTH FROM submit_date)
```

-   `ROUND(value, 2)` rounds a value to two decimal places.
-   Functions can be nested:

``` sql
ROUND(AVG(stars), 2)
```

-   Grouping by month and product creates a separate average for each
    month/product combination.
-   Sorting requirements matter: this problem required month first, then
    product ID.

------------------------------------------------------------------------

## 11. Employees Earning More Than Their Managers

### Problem

Find employees whose salary is higher than their direct manager's
salary. Return the employee ID and employee name.

### Solution

``` sql
SELECT emp.employee_id,
       emp.name AS employee_name
FROM employee AS emp
JOIN employee AS mgr
    ON emp.manager_id = mgr.employee_id
WHERE emp.salary > mgr.salary;
```

### What I Learned

-   This introduced a new pattern: **self joins**.
-   A self join is useful when one row references another row in the
    same table.
-   There is no `SELF JOIN` keyword; the same table is joined normally
    using different aliases.

``` sql
FROM employee AS emp
JOIN employee AS mgr
    ON emp.manager_id = mgr.employee_id
```

-   `emp` represents the employee and `mgr` represents the manager.
-   `emp.manager_id = mgr.employee_id` connects each employee to their
    manager.
-   After the self join, values from the related rows can be compared
    directly:

``` sql
WHERE emp.salary > mgr.salary
```

-   Self joins were a new SQL pattern for me.

------------------------------------------------------------------------

## Key Patterns Learned Today

``` sql
-- Multi-stage query with a CTE
WITH result AS (
    SELECT ...
    FROM ...
    GROUP BY ...
    HAVING ...
)
SELECT COUNT(*)
FROM result;

-- Top N grouped results
GROUP BY column_name
ORDER BY COUNT(*) DESC
LIMIT n;

-- PostgreSQL: extract month
EXTRACT(MONTH FROM date_column)

-- Round an aggregate
ROUND(AVG(column_name), 2)

-- Self join
FROM table_name AS a
JOIN table_name AS b
    ON a.related_id = b.id;
```

------------------------------------------------------------------------

## Problem-Solving Lesson

A recurring challenge is translating the wording of a problem into SQL
operations. Building queries incrementally helps:

``` text
Understand the requested output
        ↓
Identify where the required data comes from
        ↓
Start with the first operation I am confident about
        ↓
Inspect the intermediate result
        ↓
Add filtering, grouping, aggregation, sorting, or limiting as needed
```

The goal is not only to remember SQL syntax, but to improve at
converting a business question into a sequence of data operations.
