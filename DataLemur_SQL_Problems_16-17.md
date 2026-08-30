# 📊 DataLemur SQL Practice --- Problems 16--17

Today's practice focused on aggregate reasoning, range calculations, and
weighted averages.

------------------------------------------------------------------------

## 16. Cards Issued Difference

### Problem

For each credit card, calculate the difference between its highest and
lowest monthly issuance amounts, then sort the cards from the largest
difference to the smallest.

### My Solution

``` sql
SELECT
    card_name,
    MAX(issued_amount) - MIN(issued_amount) AS difference
FROM monthly_cards_issued
GROUP BY card_name
ORDER BY difference DESC;
```

### 🧠 Reasoning

The required output is one row per credit card.

1.  Find the highest monthly issuance with `MAX(issued_amount)`.
2.  Find the lowest monthly issuance with `MIN(issued_amount)`.
3.  Subtract the minimum from the maximum.
4.  Group by `card_name`.
5.  Sort the difference from highest to lowest.

Core pattern:

``` sql
MAX(column) - MIN(column)
```

### What I Learned

This became straightforward once the wording was translated into
**highest value − lowest value, per card**.

`GROUP BY card_name` creates the per-card groups before `MAX()` and
`MIN()` are calculated.

### Result

**🟢 Independent** --- solved without assistance.

------------------------------------------------------------------------

## 17. Mean Items Per Order

### Problem

Calculate the mean number of items per Alibaba order, rounded to one
decimal place.

Each row contains an `item_count` and the number of orders with that
item count (`order_occurrences`), rather than representing one
individual order.

### My MySQL Solution

``` sql
SELECT ROUND(
    SUM(item_count * order_occurrences) / SUM(order_occurrences),
    1
) AS mean
FROM items_per_order;
```

### 🧠 Reasoning

A normal average of `item_count` would be incorrect because each item
count occurs a different number of times.

``` text
Total items  = SUM(item_count × order_occurrences)
Total orders = SUM(order_occurrences)

Mean = Total items / Total orders
```

This is a **weighted mean**.

### Important Distinction

This would be wrong:

``` sql
SUM(item_count) / COUNT(*)
```

because `COUNT(*)` counts table rows, not the actual number of orders
represented by those rows.

### DataLemur MySQL Runner Issue

The MySQL version produced a relation error even though
`items_per_order` was listed as an available relation. Other users
reported the same issue in DataLemur's discussion panel.

The MySQL query logic was correct, so this was a platform issue rather
than a SQL mistake.

### PostgreSQL Note

While troubleshooting, PostgreSQL produced a `ROUND()` type error
because the division result was `double precision`. One PostgreSQL form
is:

``` sql
SELECT ROUND(
    (SUM(item_count * order_occurrences) / SUM(order_occurrences))::NUMERIC,
    1
) AS mean
FROM items_per_order;
```

### What I Learned

When a table provides a value and its frequency, a weighted average can
follow:

``` sql
SUM(value * frequency) / SUM(frequency)
```

Do not automatically use `AVG()` when each row represents multiple
observations.

### Result

**🟢 Logic correct** --- weighted-mean structure identified correctly.

**🟣 Platform / new dialect detail** --- DataLemur's MySQL runner
failed, and PostgreSQL introduced a numeric-casting detail while
troubleshooting.

------------------------------------------------------------------------

# 🔑 Patterns Reinforced Today

## 1. Range Within Each Group

``` sql
SELECT
    category,
    MAX(value) - MIN(value) AS difference
FROM table_name
GROUP BY category;
```

Think:

> **range = maximum − minimum**

## 2. Weighted Mean

``` sql
SUM(value * frequency) / SUM(frequency)
```

Think:

> **weighted total ÷ total occurrences**

## 3. Translate Business Wording Before Writing SQL

``` text
"Difference between highest and lowest issuance"
→ MAX() - MIN()

"Mean items per order when order counts are aggregated"
→ SUM(value × frequency) / SUM(frequency)
```

The SQL becomes easier once the business wording is converted into the
underlying calculation.

------------------------------------------------------------------------

# 📈 Today's Practice Summary

  -----------------------------------------------------------------------
  Problem                 Main Concepts           Outcome
  ----------------------- ----------------------- -----------------------
  Cards Issued Difference `MAX`, `MIN`,           🟢 Independent
                          `GROUP BY`, `ORDER BY`  

  Mean Items Per Order    Weighted mean, `SUM`,   🟢 Logic correct / 🟣
                          `ROUND`                 platform issue
  -----------------------------------------------------------------------

## Main Takeaway

Today's strongest improvement was **recognizing the mathematical
operation before writing the SQL**. The first problem was solved
independently, and the second problem's weighted-mean logic was
correctly constructed before the platform error appeared.
