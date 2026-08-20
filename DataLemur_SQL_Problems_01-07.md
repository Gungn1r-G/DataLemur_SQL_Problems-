# DataLemur SQL Practice --- Problems & Solutions

A compact record of the DataLemur SQL problems I solved while practicing
SQL, along with the main concept or pattern I learned from each problem.

------------------------------------------------------------------------

## 1. Histogram of Tweets

### Problem

Find the number of tweets posted by each user in 2022, then create a
histogram showing how many users fall into each tweet-count bucket.

### Solution

``` sql
WITH tweetcount AS (
    SELECT
        user_id,
        COUNT(*) AS tweet_count
    FROM tweets
    WHERE YEAR(tweet_date) = 2022
    GROUP BY user_id
)

SELECT
    tweet_count AS tweet_bucket,
    COUNT(*) AS users_num
FROM tweetcount
GROUP BY tweet_count;
```

### What I Learned

-   Some problems require **two levels of aggregation**.
-   First calculate tweets per user, then count how many users have each
    tweet count.
-   A **CTE** is useful when I need an intermediate result before
    calculating the final answer.

------------------------------------------------------------------------

## 2. Data Science Skills

### Problem

Find candidates who possess all three required skills: Python, Tableau,
and PostgreSQL.

### Solution

``` sql
SELECT candidate_id
FROM candidates
WHERE skill IN ('Python', 'Tableau', 'PostgreSQL')
GROUP BY candidate_id
HAVING COUNT(*) = 3
ORDER BY candidate_id ASC;
```

### What I Learned

-   `WHERE` filters individual rows.
-   `HAVING` filters groups after aggregation.
-   An aggregate such as `COUNT(*)` can be used in `HAVING` without
    being displayed in the final `SELECT`.
-   When values belonging to one candidate are stored across multiple
    rows, I may need to `GROUP BY` the candidate.

------------------------------------------------------------------------

## 3. Page With No Likes

### Problem

Find Facebook pages that have zero likes.

### Solution

``` sql
SELECT pag.page_id
FROM pages AS pag
LEFT JOIN page_likes AS lik
    ON pag.page_id = lik.page_id
WHERE lik.page_id IS NULL
ORDER BY pag.page_id ASC;
```

### What I Learned

-   A useful pattern for finding records with **no matching record** is:

``` sql
LEFT JOIN ...
WHERE right_table.id IS NULL
```

-   `LEFT JOIN` and `LEFT OUTER JOIN` mean the same thing.

------------------------------------------------------------------------

## 4. Tesla Unfinished Parts

### Problem

Find parts that have begun assembly but are unfinished. An unfinished
part has no `finish_date`.

### Solution

``` sql
SELECT
    part,
    assembly_step
FROM parts_assembly
WHERE finish_date IS NULL;
```

### What I Learned

-   Use `IS NULL` to check for missing values.
-   Do not use `= NULL`.
-   Not every problem needs a JOIN, CTE, or aggregation; sometimes a
    simple filter is enough.

------------------------------------------------------------------------

## 5. Laptop vs. Mobile Viewership

### Problem

Calculate total laptop views and total mobile views, where mobile
consists of tablet and phone views.

### Solution

``` sql
SELECT
    SUM(
        CASE
            WHEN device_type = 'laptop' THEN 1
            ELSE 0
        END
    ) AS laptop_views,

    SUM(
        CASE
            WHEN device_type IN ('tablet', 'phone') THEN 1
            ELSE 0
        END
    ) AS mobile_views
FROM viewership;
```

### What I Learned

-   A `CASE` expression can be placed inside `SUM()`.
-   This pattern is called **conditional aggregation**.

``` sql
SUM(CASE WHEN condition THEN 1 ELSE 0 END)
```

-   `CASE` converts matching rows to `1` and non-matching rows to `0`;
    `SUM()` then adds them.

------------------------------------------------------------------------

## 6. Days Between First and Last Post

### Problem

For each user who posted at least twice in 2021, calculate the number of
days between their first and last post of the year.

### Solution

``` sql
SELECT
    user_id,
    DATEDIFF(MAX(post_date), MIN(post_date)) AS days_between
FROM posts
WHERE YEAR(post_date) = 2021
GROUP BY user_id
HAVING COUNT(*) > 1;
```

### What I Learned

-   Multiple aggregate functions can operate on the same grouped data.
-   `MIN(post_date)` finds the earliest post.
-   `MAX(post_date)` finds the latest post.
-   MySQL's `DATEDIFF(later_date, earlier_date)` calculates the number
    of days between two dates.
-   `YEAR(post_date) = 2021` is a better way to filter a timestamp by
    year than treating it as text with `LIKE`.
-   `HAVING COUNT(*) > 1` keeps only users with at least two posts.

------------------------------------------------------------------------

------------------------------------------------------------------------

## 7. Top 2 Power Users on Microsoft Teams

### Problem

Find the top 2 users who sent the highest number of messages in August
2022. Return each `sender_id` and their total `message_count`, ordered
from highest to lowest.

### PostgreSQL Solution

``` sql
SELECT sender_id,
       COUNT(*) AS message_count
FROM messages
WHERE EXTRACT(YEAR FROM sent_date) = 2022
  AND EXTRACT(MONTH FROM sent_date) = 8
GROUP BY sender_id
ORDER BY message_count DESC
LIMIT 2;
```

### MySQL Equivalent

``` sql
SELECT sender_id,
       COUNT(*) AS message_count
FROM messages
WHERE YEAR(sent_date) = 2022
  AND MONTH(sent_date) = 8
GROUP BY sender_id
ORDER BY message_count DESC
LIMIT 2;
```

### What I Learned

-   Read every requirement carefully before writing the query. I
    initially missed that the question only wanted messages from
    **August 2022**.
-   `GROUP BY sender_id` with `COUNT(*)` counts the messages sent by
    each user.
-   `ORDER BY message_count DESC` ranks users from most messages to
    least.
-   `LIMIT 2` returns only the top two users.
-   DataLemur used PostgreSQL for this problem, so MySQL date functions
    produced an error.
-   MySQL uses `YEAR(sent_date)` and `MONTH(sent_date)`.
-   PostgreSQL can use `EXTRACT(YEAR FROM sent_date)` and
    `EXTRACT(MONTH FROM sent_date)`.

### PostgreSQL Error I Encountered

I originally tried MySQL syntax:

``` sql
WHERE YEAR(sent_date) = 2022
  AND MONTH(sent_date) = 8
```

DataLemur returned:

``` text
function year(timestamp without time zone) does not exist
```

The problem was not the query logic. The editor was using PostgreSQL,
where the equivalent syntax is:

``` sql
WHERE EXTRACT(YEAR FROM sent_date) = 2022
  AND EXTRACT(MONTH FROM sent_date) = 8
```

## Key Patterns Learned So Far

``` sql
-- Missing value
WHERE column_name IS NULL

-- Find records with no match
LEFT JOIN table_b
    ON ...
WHERE table_b.id IS NULL

-- Conditional aggregation
SUM(CASE WHEN condition THEN 1 ELSE 0 END)

-- Filter grouped results
GROUP BY column_name
HAVING COUNT(*) > 1

-- Difference between dates
DATEDIFF(later_date, earlier_date)

-- Filter by year
WHERE YEAR(date_column) = 2021

-- PostgreSQL: extract year/month from a timestamp
EXTRACT(YEAR FROM date_column)
EXTRACT(MONTH FROM date_column)
```

------------------------------------------------------------------------

*More problems will be added as I continue practicing.*
