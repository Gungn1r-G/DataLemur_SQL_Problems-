# DataLemur SQL Practice --- Problems 12--13

A compact record of today's DataLemur SQL problems, final solutions, and
the main SQL patterns learned from them.

------------------------------------------------------------------------

## 12. Final Account Balance

### Problem

Given PayPal transactions containing deposits and withdrawals, calculate
the final balance for each account.

### Solution

``` sql
SELECT
    account_id,
    SUM(
        CASE
            WHEN transaction_type = 'Deposit' THEN amount
            WHEN transaction_type = 'Withdrawal' THEN -amount
        END
    ) AS final_balance
FROM transactions
GROUP BY account_id;
```

### What I Learned

-   `CASE` decides what value each row contributes.
-   Deposits can contribute `+amount` and withdrawals `-amount`.
-   `SUM()` adds the values returned by `CASE`.
-   This is simpler than creating separate deposit and withdrawal CTEs
    and subtracting them afterward.

``` text
CASE decides what each row contributes
        ↓
SUM combines those contributions
```

------------------------------------------------------------------------

## 13. App Click-Through Rate (CTR)

### Problem

Calculate the click-through rate for each app in 2022 and round the
result to two decimal places.

``` text
CTR = 100.0 × number of clicks / number of impressions
```

### PostgreSQL Solution

``` sql
SELECT
    app_id,
    ROUND(
        100.0 *
        SUM(CASE WHEN event_type = 'click' THEN 1 ELSE 0 END) /
        SUM(CASE WHEN event_type = 'impression' THEN 1 ELSE 0 END),
        2
    ) AS ctr
FROM events
WHERE EXTRACT(YEAR FROM "timestamp") = 2022
GROUP BY app_id;
```

### What I Learned

-   Conditional aggregation can calculate separate values from different
    row types.
-   Count clicks with:

``` sql
SUM(CASE WHEN event_type = 'click' THEN 1 ELSE 0 END)
```

-   Count impressions with:

``` sql
SUM(CASE WHEN event_type = 'impression' THEN 1 ELSE 0 END)
```

-   Aggregate results can be combined directly inside a formula.
-   `100.0` avoids integer-division problems.
-   PostgreSQL can filter the year with:

``` sql
EXTRACT(YEAR FROM "timestamp") = 2022
```

-   `ROUND(..., 2)` rounds the final result to two decimal places.

------------------------------------------------------------------------

## Key Pattern Learned Today --- Conditional Aggregation

Both problems reinforced the same pattern:

``` sql
SUM(
    CASE
        WHEN condition THEN value
        ELSE other_value
    END
)
```

### Mental Model

``` text
1. Identify the values needed for the final answer.
        ↓
2. Use CASE to decide what each row contributes.
        ↓
3. Use SUM (or another aggregate) to combine those values.
        ↓
4. Combine the aggregates in a formula if needed.
        ↓
5. GROUP BY the level at which the result is required.
```

### Useful Patterns

``` sql
-- Count rows matching a condition
SUM(CASE WHEN condition THEN 1 ELSE 0 END)

-- Add positive/negative amounts conditionally
SUM(CASE
        WHEN condition_a THEN amount
        WHEN condition_b THEN -amount
    END)

-- Combine conditional aggregates in a formula
100.0 *
SUM(CASE WHEN condition_a THEN 1 ELSE 0 END) /
SUM(CASE WHEN condition_b THEN 1 ELSE 0 END)
```

The main lesson today was learning how `CASE`, aggregate functions, and
formulas can work together to solve analytical problems.
