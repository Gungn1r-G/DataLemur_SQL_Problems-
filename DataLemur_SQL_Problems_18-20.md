# 📊 DataLemur SQL Practice --- Pharmacy Analytics Parts 1--3

Today's SQL practice focused on translating business requirements into
calculations, grouping at the correct level, aggregating values across
groups, and formatting results for business-facing output.

------------------------------------------------------------------------

## 1. Pharmacy Analytics --- Top 3 Most Profitable Drugs

### Problem

Find the top 3 most profitable drugs and the amount of profit generated
by each drug.

**Profit = Total Sales - Cost of Goods Sold (COGS)**

### My Solution

``` sql
SELECT
    drug,
    (total_sales - cogs) AS total_profit
FROM pharmacy_sales
ORDER BY total_profit DESC
LIMIT 3;
```

### 🧠 Reasoning

The problem directly provides the profit formula:

``` text
Profit = Total Sales - COGS
```

So the row-level calculation is:

``` sql
total_sales - cogs
```

Then:

``` sql
ORDER BY total_profit DESC
```

places the most profitable drugs first, and:

``` sql
LIMIT 3
```

returns only the top three.

### What I Learned

Not every problem involving categories requires `GROUP BY`.

Each row already contains the sales and COGS for a drug, so no
aggregation is needed to calculate its profit.

A useful check is:

> Am I combining multiple rows into one result per category?

If not, `GROUP BY` may not be necessary.

### Result

**🟢 Independent** --- solved correctly.

------------------------------------------------------------------------

## 2. Pharmacy Analytics --- Manufacturers With Drug Losses

### Problem

Identify manufacturers associated with loss-making drugs. For each
manufacturer, return:

-   the manufacturer name,
-   the number of drugs that generated losses,
-   the total loss in absolute value.

Sort manufacturers from highest total loss to lowest.

### My Initial Attempt

``` sql
SELECT
    manufacturer,
    COUNT(*) AS drug_count,
    ABS(cogs - total_sales) AS total_loss
FROM pharmacy_sales
WHERE cogs > total_sales
GROUP BY manufacturer
ORDER BY total_loss DESC;
```

### What Was Wrong

The loss calculation:

``` sql
ABS(cogs - total_sales)
```

calculates the loss for **one row/drug**.

But the output is grouped by manufacturer, and one manufacturer can have
multiple loss-making drugs.

Therefore the losses also need to be aggregated.

### Correct Solution

``` sql
SELECT
    manufacturer,
    COUNT(*) AS drug_count,
    SUM(ABS(cogs - total_sales)) AS total_loss
FROM pharmacy_sales
WHERE cogs > total_sales
GROUP BY manufacturer
ORDER BY total_loss DESC;
```

Because the query already filters to:

``` sql
WHERE cogs > total_sales
```

the loss is guaranteed to be positive when calculated as:

``` sql
cogs - total_sales
```

so this can also be written as:

``` sql
SUM(cogs - total_sales)
```

### 🧠 Reasoning

First identify drugs that made a loss:

``` sql
WHERE cogs > total_sales
```

Then group them by manufacturer:

``` sql
GROUP BY manufacturer
```

Count the losing drugs:

``` sql
COUNT(*)
```

Finally, add the losses from all losing drugs belonging to each
manufacturer:

``` sql
SUM(cogs - total_sales)
```

### What I Learned

Once the output is **per manufacturer**, calculations involving multiple
manufacturer rows usually need to be aggregated to that same level.

The key distinction was:

``` text
Loss for one drug
→ cogs - total_sales

Total loss for a manufacturer
→ SUM(cogs - total_sales)
```

### Result

**🟠 Tool Recognition Gap** --- understood the row-level loss
calculation and grouping, but initially missed the second aggregation
required for total manufacturer loss.

------------------------------------------------------------------------

## 3. Pharmacy Analytics --- Total Sales by Manufacturer

### Problem

Calculate total drug sales for each manufacturer, round the result to
the nearest million, and format the result for a dashboard like:

``` text
$36 million
```

Sort by total sales descending. If manufacturers have the same total
sales, sort their names alphabetically.

### My Final Answer

``` sql
SELECT
    manufacturer,
    CONCAT('$', ROUND(SUM(total_sales) / 1000000), ' million') AS sale_mil
FROM pharmacy_sales
GROUP BY manufacturer
ORDER BY SUM(total_sales) DESC, manufacturer ASC;
```

### 🧠 Reasoning

The output is per manufacturer, so first aggregate all drug sales
belonging to each manufacturer:

``` sql
SUM(total_sales)
```

Convert the result from dollars to millions:

``` sql
SUM(total_sales) / 1000000
```

Round to the nearest whole million:

``` sql
ROUND(SUM(total_sales) / 1000000)
```

Then format the result:

``` sql
CONCAT(
    '$',
    ROUND(SUM(total_sales) / 1000000),
    ' million'
)
```

The sorting requirement translates to:

``` sql
ORDER BY SUM(total_sales) DESC,
         manufacturer ASC
```

### Mistake I Encountered

I initially tried:

``` sql
ORDER BY total_sales DESC
```

after grouping by manufacturer.

The query needed to sort by the **manufacturer-level aggregated sales**,
not an individual row's `total_sales`.

Therefore:

``` sql
ORDER BY SUM(total_sales) DESC
```

was required.

There was also an output-format mismatch while testing because the
challenge expected the exact requested dashboard formatting, including
lowercase `million`.

### What I Learned

Formatting can be part of the SQL requirement, especially for
dashboard-oriented questions.

Useful pattern:

``` sql
CONCAT('$', ROUND(value), ' million')
```

More importantly, this reinforced the same aggregation concept as
Problem 2:

> If the result is grouped by manufacturer, ask whether the numeric
> value also needs to be calculated at the manufacturer level.

### Result

**🟠 Tool Recognition / aggregation reinforcement** --- built the main
calculation correctly but needed to align sorting and aggregation with
the grouped output.

------------------------------------------------------------------------

# 🔑 Patterns Reinforced Today

## 1. Row-Level Calculation vs Aggregate Calculation

Row-level:

``` sql
total_sales - cogs
```

Aggregate across multiple rows:

``` sql
SUM(total_sales - cogs)
```

Ask:

> Does one output row represent one source row, or multiple source rows
> combined?

------------------------------------------------------------------------

## 2. Think About the Level of the Output

If the output says:

``` text
per drug
```

a row-level calculation may be enough.

If the output says:

``` text
per manufacturer
```

and each manufacturer has multiple drugs, aggregation is probably
required.

Example:

``` sql
GROUP BY manufacturer
```

pairs naturally with calculations such as:

``` sql
SUM(total_sales)
COUNT(*)
SUM(cogs - total_sales)
```

------------------------------------------------------------------------

## 3. Business Language → SQL

``` text
"Profit"
→ total_sales - cogs

"Loss"
→ cogs - total_sales

"Number of loss-making drugs"
→ COUNT(*)

"Total manufacturer loss"
→ SUM(cogs - total_sales)

"Total sales per manufacturer"
→ SUM(total_sales)

"Nearest million"
→ ROUND(SUM(total_sales) / 1000000)

"Top 3"
→ ORDER BY ... DESC + LIMIT 3
```

------------------------------------------------------------------------

## 4. GROUP BY Check

Before adding `GROUP BY`, ask:

> Am I combining multiple rows into one result?

Problem 1 did not require it.

Problems 2 and 3 did because multiple drugs needed to be combined at the
manufacturer level.

------------------------------------------------------------------------

# 📈 Today's Practice Summary

  -----------------------------------------------------------------------
  Problem                 Main Concepts           Outcome
  ----------------------- ----------------------- -----------------------
  Top 3 Profitable Drugs  Arithmetic, aliases,    🟢 Independent
                          sorting, `LIMIT`        

  Manufacturer Drug       `WHERE`, `COUNT`,       🟠 Aggregation
  Losses                  `SUM`, `GROUP BY`,      recognition gap
                          `ABS`                   

  Total Sales by          `SUM`, `ROUND`,         🟠 Aggregation
  Manufacturer            `CONCAT`, grouping,     reinforcement
                          multi-column sorting    
  -----------------------------------------------------------------------

## Main Weakness Identified Today

The recurring issue was **aggregation after grouping**.

When using:

``` sql
GROUP BY manufacturer
```

I need to consciously ask:

> What should each numeric value represent for the entire manufacturer?

That helps distinguish:

``` sql
total_sales
```

from:

``` sql
SUM(total_sales)
```

and:

``` sql
cogs - total_sales
```

from:

``` sql
SUM(cogs - total_sales)
```

## Main Positive

The first problem was solved independently, and the business
calculations in all three problems were understood. The main area to
reinforce is recognizing when a row-level calculation must become an
aggregate calculation after grouping.
