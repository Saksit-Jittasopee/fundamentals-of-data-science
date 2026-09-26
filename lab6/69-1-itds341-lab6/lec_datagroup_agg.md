# Data Grouping and Aggregation

## Learning Objectives

After this lecture, students should be able to:

* Explain the split-apply-combine framework.
* Use `groupby` with one or more grouping variables.
* Apply built-in and named aggregations.
* Distinguish aggregation, transformation, and filtering.
* Create pivot tables for compact summaries.
* Reshape grouped results for reporting and visualization.
* Recognize when joins and MultiIndex support a grouped analysis.

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
```

---

## 1. Running Example

The `tips` dataset contains restaurant bills and tips.

```python
tips = pd.read_csv("tips.csv")

print(tips)
# Output:
#      total_bill   tip smoker   day    time  size
# 0         16.99  1.01     No   Sun  Dinner     2
# 1         10.34  1.66     No   Sun  Dinner     3
# 2         21.01  3.50     No   Sun  Dinner     3
# 3         23.68  3.31     No   Sun  Dinner     2
# 4         24.59  3.61     No   Sun  Dinner     4
# ..          ...   ...    ...   ...     ...   ...
# 239       29.03  5.92     No   Sat  Dinner     3
# 240       27.18  2.00    Yes   Sat  Dinner     2
# 241       22.67  2.00    Yes   Sat  Dinner     2
# 242       17.82  1.75     No   Sat  Dinner     2
# 243       18.78  3.00     No  Thur  Dinner     2
#
# [244 rows x 6 columns]

tips.info()
# Output:
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 244 entries, 0 to 243
# Data columns (total 6 columns):
#  #   Column      Non-Null Count  Dtype
# ---  ------      --------------  -----
#  0   total_bill  244 non-null    float64
#  1   tip         244 non-null    float64
#  2   smoker      244 non-null    object
#  3   day         244 non-null    object
#  4   time        244 non-null    object
#  5   size        244 non-null    int64
# dtypes: float64(2), int64(1), object(3)
# memory usage: 11.6+ KB
```

Create a tip percentage for later analysis.

```python
tips["tip_pct"] = tips["tip"] / tips["total_bill"]

print(tips[["total_bill", "tip", "tip_pct"]])
# Output:
#      total_bill   tip   tip_pct
# 0         16.99  1.01  0.059447
# 1         10.34  1.66  0.160542
# 2         21.01  3.50  0.166587
# 3         23.68  3.31  0.139780
# 4         24.59  3.61  0.146808
# ..          ...   ...       ...
# 239       29.03  5.92  0.203927
# 240       27.18  2.00  0.073584
# 241       22.67  2.00  0.088222
# 242       17.82  1.75  0.098204
# 243       18.78  3.00  0.159744
#
# [244 rows x 3 columns]
```

Questions we can answer include:

* Which day has the highest average bill?
* Do smokers and non-smokers tip at different rates?
* How many parties are observed for each day and meal?
* Which observations have unusually high tip percentages within their group?

### Exercise 1: Identify the Parts

We want to calculate the average tip percentage for each day.

1. What does one row in the dataset represent?
2. Which column determines how rows are grouped?
3. Which column do we summarize, and which calculation do we use?
4. How many values should the result contain, and what does each represent?

The answers are:

1. Rows being split: All restaurant transactions in `tips`. Each row represents one bill, and rows are separated into groups by day.
2. Grouping variable: The `day` column: Thur, Fri, Sat, and Sun.
3. Calculation applied: Calculate the mean of `tip_pct` within each day’s group.
4. Expected result shape: Four values—one average tip percentage per day. With the code below, the result is a Series indexed by day.

---

## 2. Split-Apply-Combine

Grouping follows three conceptual steps:

1. **Split** rows into groups using one or more keys.
2. **Apply** a calculation to each group.
3. **Combine** the results into a Series or DataFrame.

```python
print(tips.groupby("day")["tip_pct"].mean())
# Output:
# day
# Fri     0.169913
# Sat     0.153152
# Sun     0.166897
# Thur    0.161276
# Name: tip_pct, dtype: float64
```

Pandas performs the split lazily. Creating a grouped object does not itself
calculate a result.

```python
by_day = tips.groupby("day")

print(type(by_day))
# Output:
# <class 'pandas.core.groupby.generic.DataFrameGroupBy'>
```

### Inspect Groups

```python
print(by_day.size())
# Output:
# day
# Fri     19
# Sat     87
# Sun     76
# Thur    62
# dtype: int64

print(by_day.get_group("Sun"))
# Output:
#      total_bill   tip smoker  day    time  size   tip_pct
# 0         16.99  1.01     No  Sun  Dinner     2  0.059447
# 1         10.34  1.66     No  Sun  Dinner     3  0.160542
# 2         21.01  3.50     No  Sun  Dinner     3  0.166587
# 3         23.68  3.31     No  Sun  Dinner     2  0.139780
# 4         24.59  3.61     No  Sun  Dinner     4  0.146808
# ..          ...   ...    ...  ...     ...   ...       ...
# 186       20.90  3.50    Yes  Sun  Dinner     3  0.167464
# 187       30.46  2.00    Yes  Sun  Dinner     5  0.065660
# 188       18.15  3.50    Yes  Sun  Dinner     3  0.192837
# 189       23.10  4.00    Yes  Sun  Dinner     3  0.173160
# 190       15.69  1.50    Yes  Sun  Dinner     2  0.095602
#
# [76 rows x 7 columns]
```

Use `.size()` for the number of rows. Use `.count()` for the number of
non-missing values in each selected column.

```python
print(tips.groupby("day").size())
# Output:
# day
# Fri     19
# Sat     87
# Sun     76
# Thur    62
# dtype: int64

print(tips.groupby("day")["tip"].count())
# Output:
# day
# Fri     19
# Sat     87
# Sun     76
# Thur    62
# Name: tip, dtype: int64
```

These differ when `tip` contains missing values.

---

## 3. Grouping with One or More Keys

### One Grouping Variable

To find the average bill for each day, read the expression from left to right:

1. `tips.groupby("day")` groups the rows by the values in the `day` column.
2. `["total_bill"]` selects the **column to summarize from the grouped data**.
   It does not create another grouping; the rows are still grouped by `day`.
3. `.mean()` computes the mean of the selected `total_bill` values within each
   day, producing one average bill per day.

Here, `day` is the **grouping column**, `total_bill` is the **column being
summarized**, and `mean` is the **aggregation**. `print(...)` displays the result.

```python
print(tips.groupby("day")["total_bill"].mean())
# Output:
# day
# Fri     17.151579
# Sat     20.441379
# Sun     21.410000
# Thur    17.682742
# Name: total_bill, dtype: float64
```

### Multiple Grouping Variables

```python
day_smoker_rates = tips.groupby(
    ["day", "smoker"]
)["tip_pct"].mean()

print(day_smoker_rates)
# Output:
# day   smoker
# Fri   No        0.151650
#       Yes       0.174783
# Sat   No        0.158048
#       Yes       0.147906
# Sun   No        0.160113
#       Yes       0.187250
# Thur  No        0.160298
#       Yes       0.163863
# Name: tip_pct, dtype: float64
```

The result uses a MultiIndex because each value is identified by both `day`
and `smoker`.

Use `as_index=False` when ordinary columns are more convenient.

```python
day_smoker_table = tips.groupby(
    ["day", "smoker"],
    as_index=False
)["tip_pct"].mean()

print(day_smoker_table)
# Output:
#     day smoker   tip_pct
# 0   Fri     No  0.151650
# 1   Fri    Yes  0.174783
# 2   Sat     No  0.158048
# 3   Sat    Yes  0.147906
# 4   Sun     No  0.160113
# 5   Sun    Yes  0.187250
# 6  Thur     No  0.160298
# 7  Thur    Yes  0.163863
```

### Select Columns Deliberately

```python
day_smoker_means = tips.groupby(
    ["day", "smoker"]
)[["total_bill", "tip", "tip_pct"]].mean()

print(day_smoker_means)
# Output:
#              total_bill       tip   tip_pct
# day  smoker
# Fri  No       18.420000  2.812500  0.151650
#      Yes      16.813333  2.714000  0.174783
# Sat  No       19.661778  3.102889  0.158048
#      Yes      21.276667  2.875476  0.147906
# Sun  No       20.506667  3.167895  0.160113
#      Yes      24.120000  3.516842  0.187250
# Thur No       17.113111  2.673778  0.160298
#      Yes      19.190588  3.030000  0.163863
```

Selecting relevant columns produces clearer results and avoids accidentally
aggregating unrelated numeric columns.

### Exercise 2: Basic Grouping

Write one expression for each question:

1. Number of parties on each day.
2. Median bill for each meal time.
3. Mean tip percentage by day and smoker status.
4. Maximum party size by day.

Possible answers:

```python
tips.groupby("day").size()
tips.groupby("time")["total_bill"].median()
tips.groupby(["day", "smoker"])["tip_pct"].mean()
tips.groupby("day")["size"].max()
```

---

## 4. Aggregation

An **aggregation** reduces each group to one or more summary values.

### Built-in Aggregations

```python
tip_statistics = tips.groupby("day")["tip_pct"].agg(
    ["count", "mean", "median", "std"]
)

print(tip_statistics)
# Output:
#       count      mean    median       std
# day
# Fri      19  0.169913  0.155625  0.047665
# Sat      87  0.153152  0.151832  0.051293
# Sun      76  0.166897  0.161103  0.084739
# Thur     62  0.161276  0.153846  0.038652
```

Common aggregations include:

| Function | Meaning |
|---|---|
| `size` | Number of rows |
| `count` | Number of non-missing values |
| `sum` | Total |
| `mean` | Arithmetic mean |
| `median` | Median |
| `min`, `max` | Extreme values |
| `std`, `var` | Dispersion |
| `nunique` | Number of distinct values |

### Named Aggregation

Named aggregation gives each output column a clear name.

```python
day_summary = (
    tips.groupby("day", as_index=False)
    .agg(
        parties=("total_bill", "size"),
        average_bill=("total_bill", "mean"),
        median_tip_pct=("tip_pct", "median"),
        largest_party=("size", "max")
    )
)

print(day_summary)
# Output:
#     day  parties  average_bill  median_tip_pct  largest_party
# 0   Fri       19     17.151579        0.155625              4
# 1   Sat       87     20.441379        0.151832              5
# 2   Sun       76     21.410000        0.161103              6
# 3  Thur       62     17.682742        0.153846              6
```

This form is suitable for reports because the resulting columns are flat and
self-explanatory.

### Multiple Functions for Multiple Columns

```python
multi_column_statistics = tips.groupby("day")[["tip", "total_bill"]].agg(
    ["mean", "max"]  # Apply both functions to both selected columns.
)

print(multi_column_statistics)
# Output:
#            tip        total_bill
#           mean    max       mean    max
# day
# Fri   2.734737   4.73  17.151579  40.17
# Sat   2.993103  10.00  20.441379  50.81
# Sun   3.255132   6.50  21.410000  48.17
# Thur  2.771452   6.70  17.682742  43.11
```

The outer column level identifies the value column; the inner level identifies
the aggregation function.

### Multiple Columns and Custom Summary Names

```python
named_statistics = tips.groupby("day")[["tip_pct", "total_bill"]].agg(
    [("average", "mean"), ("stdev", "std")]  # (output name, function)
)

print(named_statistics)
# Output:
#        tip_pct           total_bill
#        average     stdev    average     stdev
# day
# Fri   0.169913  0.047665  17.151579  8.302660
# Sat   0.153152  0.051293  20.441379  9.480419
# Sun   0.166897  0.084739  21.410000  8.832122
# Thur  0.161276  0.038652  17.682742  7.886170
```

The same functions are applied to both selected columns. The result has two
column levels: the original column and the chosen summary name.

### Different Functions for Different Columns

```python
different_statistics = tips.groupby("day").agg({
    "tip": "max",         # Largest tip per day.
    "total_bill": "mean"  # Average bill per day.
})

print(different_statistics)
# Output:
#         tip  total_bill
# day
# Fri    4.73   17.151579
# Sat   10.00   20.441379
# Sun    6.50   21.410000
# Thur   6.70   17.682742
```

Each column receives its own function. To use several functions for just one
column, pass a list for that column:

```python
column_statistics = tips.groupby("day").agg({
    "tip_pct": ["mean", "std"],  # Two summaries for tip percentage.
    "size": "sum"               # One summary for the number of diners.
})

print(column_statistics)
# Output:
#        tip_pct           size
#           mean       std  sum
# day
# Fri   0.169913  0.047665   40
# Sat   0.153152  0.051293  219
# Sun   0.166897  0.084739  216
# Thur  0.161276  0.038652  152
```

Dictionary keys select columns; their values specify one function or a list
of functions. This is the dictionary form used in Lab Task 3.2. Use named
aggregation, shown above, when you prefer flat output columns.

### Custom Aggregation

For the aggregations taught here, a custom function receives a Series containing
one column's values from one group and should return one **scalar summary**,
such as a number. With several functions, each returns its own summary value.
Use `apply` when the function needs to return several rows, as in `top_bills`.

```python
def peak_to_peak(values):
    # Receive one column from one group; return one summary value.
    return values.max() - values.min()

bill_ranges = tips.groupby("day")["total_bill"].agg(
    bill_range=peak_to_peak
)

print(bill_ranges)
# Output:
#       bill_range
# day
# Fri        34.42
# Sat        47.74
# Sun        40.92
# Thur       35.60
```

`values` is the selected column's Series for one day. Here the function returns
its range: maximum minus minimum. Pass `peak_to_peak` without quotes or
parentheses so pandas can call it with each group's values.

**Combine a custom function with built-in functions:**

```python
custom_statistics = tips.groupby("day")["total_bill"].agg(
    ["mean", "max", peak_to_peak]  # Built-in names and a custom function.
)

print(custom_statistics)
# Output:
#            mean    max  peak_to_peak
# day
# Fri   17.151579  40.17         34.42
# Sat   20.441379  50.81         47.74
# Sun   21.410000  48.17         40.92
# Thur  17.682742  43.11         35.60
```

**Use a custom function in named aggregation:**

```python
custom_summary = tips.groupby("day").agg(
    average_bill=("total_bill", "mean"),
    bill_range=("total_bill", peak_to_peak),  # Custom function, clear output name.
    highest_tip=("tip", "max")
)

print(custom_summary)
# Output:
#       average_bill  bill_range  highest_tip
# day
# Fri      17.151579       34.42         4.73
# Sat      20.441379       47.74        10.00
# Sun      21.410000       40.92         6.50
# Thur     17.682742       35.60         6.70
```

Custom functions also work as values in the aggregation dictionary shown
above. For these summaries, each function should return one value per group.
Prefer built-in functions when they already perform the calculation you need.

### Exercise 3: Build a Summary Table

Create one row per combination of `time` and `smoker` with:

* number of parties;
* average total bill;
* median tip percentage;
* maximum party size.

Possible answer:

```python
tips.groupby(
    ["time", "smoker"],
    as_index=False
)
.agg(
    parties=("total_bill", "size"),
    average_bill=("total_bill", "mean"),
    median_tip_pct=("tip_pct", "median"),
    maximum_size=("size", "max")
)
```

---

## 5. Aggregation, Transformation, and Filtering

These operations answer different kinds of questions.

| Operation | Output shape | Typical question |
|---|---|---|
| Aggregation | One or more rows per group | What is each group’s mean? |
| Transformation | Same number of rows as input | How does each row compare with its group? |
| Filtering | Selected original rows or groups | Which groups satisfy a condition? |

### Transformation

`transform` returns a value aligned with every original row.

```python
tips["day_mean_tip_pct"] = (
    tips.groupby("day")["tip_pct"]
    .transform("mean")
)

tips["tip_pct_vs_day"] = (
    tips["tip_pct"] - tips["day_mean_tip_pct"]
)

print(tips[["day", "tip_pct", "day_mean_tip_pct", "tip_pct_vs_day"]])
# Output:
#       day   tip_pct  day_mean_tip_pct  tip_pct_vs_day
# 0     Sun  0.059447          0.166897       -0.107451
# 1     Sun  0.160542          0.166897       -0.006356
# 2     Sun  0.166587          0.166897       -0.000310
# 3     Sun  0.139780          0.166897       -0.027117
# 4     Sun  0.146808          0.166897       -0.020090
# ..    ...       ...               ...             ...
# 239   Sat  0.203927          0.153152        0.050775
# 240   Sat  0.073584          0.153152       -0.079568
# 241   Sat  0.088222          0.153152       -0.064929
# 242   Sat  0.098204          0.153152       -0.054947
# 243  Thur  0.159744          0.161276       -0.001531
#
# [244 rows x 4 columns]

print(tips.shape[0])
# Output:
# 244
```

Group-wise standardization:

```python
tips["bill_z_by_day"] = (
    tips.groupby("day")["total_bill"]
    .transform(lambda x: (x - x.mean()) / x.std())
)

print(tips[["day", "total_bill", "bill_z_by_day"]])
# Output:
#       day  total_bill  bill_z_by_day
# 0     Sun       16.99      -0.500446
# 1     Sun       10.34      -1.253379
# 2     Sun       21.01      -0.045289
# 3     Sun       23.68       0.257016
# 4     Sun       24.59       0.360049
# ..    ...         ...            ...
# 239   Sat       29.03       0.905933
# 240   Sat       27.18       0.710794
# 241   Sat       22.67       0.235076
# 242   Sat       17.82      -0.276505
# 243  Thur       18.78       0.139137
#
# [244 rows x 3 columns]
```

This is useful for comparing an observation with a relevant local baseline
instead of the entire dataset.

### Fill Missing Values by Group

The goal is to fill each missing tip with the median tip for the same day,
while keeping existing tip values unchanged.

```python
example = tips.copy()

example.loc[[0, 10, 20], "tip"] = np.nan

example["tip_filled"] = (
    example["tip"]
    .fillna(
        example.groupby("day")["tip"]
        .transform("median")
    )
)

print(example.loc[[0, 10, 20], ["day", "tip", "tip_filled"]])
# Output:
#     day  tip  tip_filled
# 0   Sun  NaN       3.205
# 10  Sun  NaN       3.205
# 20  Sat  NaN       2.735
```

How it works:

1. Copy `tips` and create three missing tips for practice.
2. `groupby("day")["tip"]` selects tips within each day.
3. `transform("median")` gives every row its day's median, keeping the original
   row index so `fillna` can match the replacement to the correct row.
4. `fillna` replaces only missing tips and saves the result in `tip_filled`.

The median is less affected by unusually large tips than the mean. Using a
separate median per day assumes day is a useful basis for filling missing tips.

**Alternative: `median()` + `map()`.** Calculate one median per day, then look
up the appropriate median for each row:

```python
median_by_day = example.groupby("day")["tip"].median()

print(median_by_day)
# Output:
# day
# Fri     3.000
# Sat     2.735
# Sun     3.205
# Thur    2.305
# Name: tip, dtype: float64

replacement_tips = example["day"].map(median_by_day)

print(replacement_tips)
# Output:
# 0      3.205
# 1      3.205
# 2      3.205
# 3      3.205
# 4      3.205
#        ...
# 239    2.735
# 240    2.735
# 241    2.735
# 242    2.735
# 243    2.305
# Name: day, Length: 244, dtype: float64


example["tip_filled"] = example["tip"].fillna(replacement_tips)

print(example.loc[[0, 10, 20], ["day", "tip", "tip_filled"]])
# Output:
#     day  tip  tip_filled
# 0   Sun  NaN       3.205
# 10  Sun  NaN       3.205
# 20  Sat  NaN       2.735
```

Both approaches give the same result. If a day has no observed tips, its
median is missing and those tips remain unfilled.

### Filter Groups

Keep groups with at least 20 observations:

```python
large_day_groups = tips.groupby(
    "day"
).filter(lambda group: len(group) >= 20)

print(large_day_groups[["day", "total_bill", "tip"]])
# Output:
#       day  total_bill   tip
# 0     Sun       16.99  1.01
# 1     Sun       10.34  1.66
# 2     Sun       21.01  3.50
# 3     Sun       23.68  3.31
# 4     Sun       24.59  3.61
# ..    ...         ...   ...
# 239   Sat       29.03  5.92
# 240   Sat       27.18  2.00
# 241   Sat       22.67  2.00
# 242   Sat       17.82  1.75
# 243  Thur       18.78  3.00
#
# [225 rows x 3 columns]

print(large_day_groups.shape[0])
# Output:
# 225

print(large_day_groups.groupby("day").size())
# Output:
# day
# Sat     87
# Sun     76
# Thur    62
# dtype: int64
```

To obtain a summary rather than original rows, aggregate first and filter the
summary.

```python
day_counts = (
    tips.groupby("day", as_index=False)
    .agg(parties=("total_bill", "size"))
)

print(day_counts.loc[day_counts["parties"] >= 20])
# Output:
#     day  parties
# 1   Sat       87
# 2   Sun       76
# 3  Thur       62
```

### Exercise 4: Choose the Operation

Choose aggregation, transformation, or filtering:

1. Add each day’s average bill to every row.
2. Calculate one average bill per day.
3. Keep only days with at least 30 parties.
4. Calculate each bill’s difference from its day’s median.

Answers: transformation, aggregation, filtering, transformation.

---

## 6. `apply`: Use When Necessary

`apply` passes each group to a function and combines the returned objects. It
is flexible, but often less predictable and slower than a specific aggregation
or transformation.

Example: return the three largest bills from each day.

```python
def top_bills(group, n=3):
    # Return the three largest bills within one day's group.
    return group.nlargest(n, "total_bill")

top_three_per_day = (
    tips.groupby("day", group_keys=True)  # Add day labels to the result's index.
    .apply(top_bills, include_groups=False)  # Pass each group without the day column.
)

print(top_three_per_day[["total_bill", "tip", "smoker", "time", "size"]])
# Output:
#           total_bill    tip smoker    time  size
# day
# Fri  95        40.17   4.73    Yes  Dinner     4
#      90        28.97   3.00    Yes  Dinner     2
#      96        27.28   4.00    Yes  Dinner     2
# Sat  170       50.81  10.00    Yes  Dinner     3
#      212       48.33   9.00     No  Dinner     4
#      59        48.27   6.73     No  Dinner     4
# Sun  156       48.17   5.00     No  Dinner     6
#      182       45.35   3.50    Yes  Dinner     3
#      184       40.55   3.00    Yes  Dinner     2
# Thur 197       43.11   5.00    Yes   Lunch     4
#      142       41.19   5.00     No   Lunch     5
#      85        34.83   5.17     No   Lunch     4
```

`apply` runs `top_bills` for each day and combines the results into 12 rows.
Although `day` is excluded from the columns passed to the function,
`group_keys=True` keeps it in the result's index alongside the original row
index. This shows which day each selected bill belongs to.

Use this decision order:

1. Built-in aggregation such as `.mean()` or `.sum()`
2. `.agg()` for group summaries
3. `.transform()` for row-aligned results
4. `.filter()` for selecting complete groups
5. `.apply()` for group results that do not fit the earlier forms

---

## 7. Pivot Tables

A pivot table summarizes values across row and column categories.

```python
tip_pivot = pd.pivot_table(
    tips,
    values="tip_pct",
    index="day",
    columns="smoker",
    aggfunc="mean"
)

print(tip_pivot)
# Output:
# smoker        No       Yes
# day
# Fri     0.151650  0.174783
# Sat     0.158048  0.147906
# Sun     0.160113  0.187250
# Thur    0.160298  0.163863
```

This is closely related to grouping:

```python
grouped_pivot = (
    tips.groupby(
        ["day", "smoker"]
    )["tip_pct"]
    .mean()
    .unstack()
)

print(grouped_pivot)
# Output:
# smoker        No       Yes
# day
# Fri     0.151650  0.174783
# Sat     0.158048  0.147906
# Sun     0.160113  0.187250
# Thur    0.160298  0.163863
```

### Rename Pivot Columns

```python
renamed_pivot = tip_pivot.rename(columns={
    "No": "non_smoker",
    "Yes": "smoker"
})

print(renamed_pivot)
# Output:
# smoker  non_smoker    smoker
# day
# Fri       0.151650  0.174783
# Sat       0.158048  0.147906
# Sun       0.160113  0.187250
# Thur      0.160298  0.163863
```

`rename(columns=...)` changes the result's column labels; it does not change
the source data or recalculate the values.

### Multiple Aggregation Functions

```python
multi_agg_pivot = pd.pivot_table(
    tips,
    values="tip_pct",
    index="day",
    columns="smoker",
    aggfunc=["count", "mean"]  # Apply both functions to tip_pct.
)

print(multi_agg_pivot)
# Output:
#        count          mean
# smoker    No Yes        No       Yes
# day
# Fri        4  15  0.151650  0.174783
# Sat       45  42  0.158048  0.147906
# Sun       57  19  0.160113  0.187250
# Thur      45  17  0.160298  0.163863
```

The columns now have two levels: aggregation function, then smoker status.
`count` counts non-missing tip percentages within each group.

### Rename an Aggregation Level

```python
renamed_agg_pivot = multi_agg_pivot.rename(
    columns={"count": "valid_tips", "mean": "average"},
    level=0  # Rename the function labels in the outer column level.
)

print(renamed_agg_pivot)
# Output:
#        valid_tips       average
# smoker         No Yes        No       Yes
# day
# Fri             4  15  0.151650  0.174783
# Sat            45  42  0.158048  0.147906
# Sun            57  19  0.160113  0.187250
# Thur           45  17  0.160298  0.163863
```

Use `level=1` to rename the inner smoker-status labels instead.

### Multiple Values with Multiple Functions

```python
multi_value_pivot = pd.pivot_table(
    tips,
    values=["total_bill", "tip"],
    index="day",
    aggfunc=["mean", "max"]  # Apply both functions to both value columns.
)

print(multi_value_pivot)
# Output:
#           mean               max
#            tip total_bill    tip total_bill
# day
# Fri   2.734737  17.151579   4.73      40.17
# Sat   2.993103  20.441379  10.00      50.81
# Sun   3.255132  21.410000   6.50      48.17
# Thur  2.771452  17.682742   6.70      43.11
```

With no `columns` argument, there is one row per day and a summary column for
each function/value pair.

### Different Functions for Different Values

```python
different_agg_pivot = pd.pivot_table(
    tips,
    values=["total_bill", "tip"],
    index="day",
    columns="smoker",
    aggfunc={"total_bill": "mean", "tip": "max"}
)

print(different_agg_pivot)
# Output:
#         tip        total_bill
# smoker   No    Yes         No        Yes
# day
# Fri     3.5   4.73  18.420000  16.813333
# Sat     9.0  10.00  19.661778  21.276667
# Sun     6.0   6.50  20.506667  24.120000
# Thur    6.7   5.00  17.113111  19.190588
```

The dictionary applies the mean only to `total_bill` and the maximum only to
`tip`, separately for each day/smoker combination.

### Mix One Function and Several Functions

```python
mixed_agg_pivot = pd.pivot_table(
    tips,
    values=["total_bill", "tip"],
    index="day",
    aggfunc={"total_bill": ["mean", "max"], "tip": "median"}
)

print(mixed_agg_pivot)
# Output:
#         tip total_bill
#      median        max       mean
# day
# Fri   3.000      40.17  17.151579
# Sat   2.750      50.81  20.441379
# Sun   3.150      48.17  21.410000
# Thur  2.305      43.11  17.682742
```

Here the column levels are value column, then function. You can also add
`columns="smoker"` to split each summary by smoker status.

### Custom Aggregation Function

```python
def peak_to_peak(values):
    # Calculate the difference between the largest and smallest bill.
    return values.max() - values.min()

bill_range_pivot = pd.pivot_table(
    tips,
    values="total_bill",
    index="day",
    aggfunc=peak_to_peak  # Pass the function without quotes or parentheses.
)

print(bill_range_pivot)
# Output:
#       total_bill
# day
# Fri        34.42
# Sat        47.74
# Sun        40.92
# Thur       35.60
```

Pandas passes each day's `total_bill` values to `peak_to_peak`, which returns
one range per day. The same scalar-summary requirement applies to custom
functions passed to `pivot_table` through `aggfunc`.

### Multiple Values and Margins

```python
meal_pivot = pd.pivot_table(
    tips,
    values=["total_bill", "tip_pct"],
    index="day",
    columns="time",
    aggfunc="mean",
    margins=True,
    margins_name="Overall"  # Rename the default All row and columns.
)

print(meal_pivot)
# Output:
#           tip_pct                     total_bill
# time       Dinner     Lunch   Overall     Dinner      Lunch    Overall
# day
# Fri      0.158916  0.188765  0.169913  19.663333  12.845714  17.151579
# Sat      0.153152       NaN  0.153152  20.441379        NaN  20.441379
# Sun      0.166897       NaN  0.166897  21.410000        NaN  21.410000
# Thur     0.159744  0.161301  0.161276  18.780000  17.664754  17.682742
# Overall  0.159518  0.164128  0.160803  20.797159  17.168676  19.785943
```

`Overall` recalculates each mean from the underlying transactions; it is not
the unweighted average of the displayed group means.

Use `fill_value` only when replacing missing combinations with a specific
value is meaningful. A missing combination does not always mean zero.

### Counts

For frequency tables, `pd.crosstab` is concise.

```python
smoker_counts = pd.crosstab(
    tips["day"],
    tips["smoker"],
    margins=True
)

print(smoker_counts)
# Output:
# smoker   No  Yes  All
# day
# Fri       4   15   19
# Sat      45   42   87
# Sun      57   19   76
# Thur     45   17   62
# All     151   93  244
```

### Fill Missing Cells in a Count Table

```python
party_counts = pd.pivot_table(
    tips,
    index="day",
    columns="time",
    aggfunc="size",  # Count rows rather than non-missing values.
    fill_value=0     # No observed parties for this day/meal combination.
)

print(party_counts)
# Output:
# time  Dinner  Lunch
# day
# Fri       12      7
# Sat       87      0
# Sun       76      0
# Thur       1     61
```

Zero is meaningful here as a count of records in this dataset. It would be
misleading to replace an unknown mean tip percentage with zero.

### Exercise 5: Report Table

Create a pivot table showing the median tip percentage:

* rows: `time`;
* columns: `day`;
* separate values for smoker status.

```python
median_pivot = pd.pivot_table(
    tips,
    values="tip_pct",
    index=["time", "smoker"],
    columns="day",
    aggfunc="median"
)

print(median_pivot)
# Output:
# day                 Fri       Sat       Sun      Thur
# time   smoker
# Dinner No      0.142857  0.150152  0.161665  0.159744
#        Yes     0.146628  0.153624  0.138122       NaN
# Lunch  No      0.187735       NaN       NaN  0.150964
#        Yes     0.189569       NaN       NaN  0.153846
```

---

## 8. Plot a Grouped Summary

This example connects grouping to plotting: create a summary table, then use its columns to draw a bar chart.

```python
plot_data = (
    tips.groupby(
        ["day", "smoker"],
        as_index=False
    )
    .agg(mean_tip_pct=("tip_pct", "mean"))
)

print(plot_data)
# Output:
#     day smoker  mean_tip_pct
# 0   Fri     No      0.151650
# 1   Fri    Yes      0.174783
# 2   Sat     No      0.158048
# 3   Sat    Yes      0.147906
# 4   Sun     No      0.160113
# 5   Sun    Yes      0.187250
# 6  Thur     No      0.160298
# 7  Thur    Yes      0.163863

sns.barplot(
    data=plot_data,
    x="day",
    y="mean_tip_pct",
    hue="smoker"
)
plt.ylabel("Mean tip percentage")
plt.show()
# Output: A grouped bar chart with day on the x-axis, mean_tip_pct on
# the y-axis, and separate bars for smoker = No and Yes. The eight bar
# heights are the mean_tip_pct values printed above.
```

Keep the unit of analysis clear: the bars summarize groups, not individual
transactions.

---

## 9. Supporting Concepts

These concepts are useful when grouped analysis involves several tables or
multi-level results. They are supporting material rather than the center of
this lecture.

### MultiIndex

Grouping by multiple keys commonly creates a MultiIndex.

```python
grouped = tips.groupby(
    ["day", "smoker"]
)["tip_pct"].mean()

print(grouped.loc["Sun"])
# Output:
# smoker
# No     0.160113
# Yes    0.187250
# Name: tip_pct, dtype: float64

print(grouped.reset_index())
# Output:
#     day smoker   tip_pct
# 0   Fri     No  0.151650
# 1   Fri    Yes  0.174783
# 2   Sat     No  0.158048
# 3   Sat    Yes  0.147906
# 4   Sun     No  0.160113
# 5   Sun    Yes  0.187250
# 6  Thur     No  0.160298
# 7  Thur    Yes  0.163863

print(grouped.unstack())
# Output:
# smoker        No       Yes
# day
# Fri     0.151650  0.174783
# Sat     0.158048  0.147906
# Sun     0.160113  0.187250
# Thur    0.160298  0.163863
```

For most reporting tasks:

* use `as_index=False` or `.reset_index()` for a flat table;
* use `.unstack()` when one index level should become columns.

### Categorical Groups: When `observed` Matters

The `day`, `smoker`, and `time` columns loaded from `tips.csv` are ordinary
strings, so the examples above do not need `observed`.

A categorical column can define possible values that do not appear in any row.
Here, Wednesday is an allowed category, but there are no Wednesday records.

```python
visits = pd.DataFrame({
    "day": pd.Categorical(
        ["Mon", "Mon", "Tue"],
        categories=["Mon", "Tue", "Wed"]
    )
})

print(visits)
# Output:
#    day
# 0  Mon
# 1  Mon
# 2  Tue

print(visits.groupby("day", observed=False).size())
# Output:
# day
# Mon    2
# Tue    1
# Wed    0
# dtype: int64

print(visits.groupby("day", observed=True).size())
# Output:
# day
# Mon    2
# Tue    1
# dtype: int64
```

Use `observed=True` when you want only categories represented in the data.
Use `observed=False` when you want to include unused categories too. With
multiple categorical grouping keys, this also affects which combinations
appear. The zero above comes from counting rows; an empty group's mean would
be `NaN`.

This choice also applies to categorical columns created with `pd.cut()`, such
as `Age_group` in the lab. It becomes visible when a category or combination
has no observations. Both settings are explicit here so the comparison does
not depend on the pandas version's default.

### Joining a Lookup Table

Suppose another table assigns each day to a business category.

```python
day_lookup = pd.DataFrame({
    "day": ["Thur", "Fri", "Sat", "Sun"],
    "period": ["weekday", "weekday", "weekend", "weekend"]
})

tips_with_period = tips.merge(
    day_lookup,
    on="day",
    how="left",
    validate="many_to_one"
)

print(tips_with_period[["day", "total_bill", "tip", "period"]])
# Output:
#       day  total_bill   tip   period
# 0     Sun       16.99  1.01  weekend
# 1     Sun       10.34  1.66  weekend
# 2     Sun       21.01  3.50  weekend
# 3     Sun       23.68  3.31  weekend
# 4     Sun       24.59  3.61  weekend
# ..    ...         ...   ...      ...
# 239   Sat       29.03  5.92  weekend
# 240   Sat       27.18  2.00  weekend
# 241   Sat       22.67  2.00  weekend
# 242   Sat       17.82  1.75  weekend
# 243  Thur       18.78  3.00  weekday
#
# [244 rows x 4 columns]

print(tips_with_period.shape[0])
# Output:
# 244
```

Then group using the added information:

```python
period_average_bill = tips_with_period.groupby(
    "period"
)["total_bill"].mean()

print(period_average_bill)
# Output:
# period
# weekday    17.558148
# weekend    20.893006
# Name: total_bill, dtype: float64
```

`validate="many_to_one"` checks the expected relationship and helps catch
duplicate lookup keys.

### Concatenating Compatible Tables

Use `concat` to stack tables with the same columns.

```python
first_half = tips.iloc[:100]

second_half = tips.iloc[100:]

combined = pd.concat(
    [first_half, second_half],
    ignore_index=True
)

print(combined[["total_bill", "tip", "day"]])
# Output:
#      total_bill   tip   day
# 0         16.99  1.01   Sun
# 1         10.34  1.66   Sun
# 2         21.01  3.50   Sun
# 3         23.68  3.31   Sun
# 4         24.59  3.61   Sun
# ..          ...   ...   ...
# 239       29.03  5.92   Sat
# 240       27.18  2.00   Sat
# 241       22.67  2.00   Sat
# 242       17.82  1.75   Sat
# 243       18.78  3.00  Thur
#
# [244 rows x 3 columns]

print(combined.shape)
# Output:
# (244, 10)

print(combined.equals(tips))
# Output:
# True
```

Joining and concatenation deserve deeper treatment when data integration is
the main objective. Here they support the grouped analysis.

---

## 10. Common Mistakes

### Mistake 1: Confusing `size` and `count`

`size` counts rows; `count` excludes missing values.

### Mistake 2: Aggregating Every Numeric Column

Select columns deliberately and name outputs.

### Mistake 3: Using `apply` for Everything

Prefer specific operations because their result shape and intent are clearer.

### Mistake 4: Ignoring the Unit of Analysis

State whether a result describes transactions, days, customers, or another
unit.

### Mistake 5: Treating Missing Pivot Cells as Zero

Confirm whether a combination is absent, unobserved, or truly zero.

### Mistake 6: Comparing Means Without Group Sizes

Report counts alongside group statistics, especially for small groups.

---

## 11. Integrated Exercise

Answer the question:

> How does tipping behavior vary by day and smoker status?

Produce a table containing:

* number of parties;
* mean and median tip percentage;
* standard deviation of tip percentage;
* average total bill.

Then:

1. Identify groups with fewer than 10 observations.
2. Create a pivot table of mean tip percentage.
3. Add each row’s difference from its group median.
4. Write two conclusions and one limitation.

One possible summary:

```python
summary = (
    tips.groupby(
        ["day", "smoker"],
        as_index=False
    )
    .agg(
        parties=("tip_pct", "size"),
        mean_tip_pct=("tip_pct", "mean"),
        median_tip_pct=("tip_pct", "median"),
        sd_tip_pct=("tip_pct", "std"),
        average_bill=("total_bill", "mean")
    )
)

print(summary)
# Output:
#     day smoker  parties  mean_tip_pct  median_tip_pct  sd_tip_pct  average_bill
# 0   Fri     No        4      0.151650        0.149241    0.028123     18.420000
# 1   Fri    Yes       15      0.174783        0.173913    0.051293     16.813333
# 2   Sat     No       45      0.158048        0.150152    0.039767     19.661778
# 3   Sat    Yes       42      0.147906        0.153624    0.061375     21.276667
# 4   Sun     No       57      0.160113        0.161665    0.042347     20.506667
# 5   Sun    Yes       19      0.187250        0.138122    0.154134     24.120000
# 6  Thur     No       45      0.160298        0.153492    0.038774     17.113111
# 7  Thur    Yes       17      0.163863        0.153846    0.039389     19.190588
```

Row-level comparison:

```python
tips["group_median_tip_pct"] = (
    tips.groupby(
        ["day", "smoker"]
    )["tip_pct"]
    .transform("median")
)

tips["tip_pct_vs_group_median"] = (
    tips["tip_pct"] - tips["group_median_tip_pct"]
)

print(tips[["day", "smoker", "tip_pct", "group_median_tip_pct", "tip_pct_vs_group_median"]])
# Output:
#       day smoker   tip_pct  group_median_tip_pct  tip_pct_vs_group_median
# 0     Sun     No  0.059447              0.161665                -0.102218
# 1     Sun     No  0.160542              0.161665                -0.001123
# 2     Sun     No  0.166587              0.161665                 0.004922
# 3     Sun     No  0.139780              0.161665                -0.021885
# 4     Sun     No  0.146808              0.161665                -0.014857
# ..    ...    ...       ...                   ...                      ...
# 239   Sat     No  0.203927              0.150152                 0.053775
# 240   Sat    Yes  0.073584              0.153624                -0.080041
# 241   Sat    Yes  0.088222              0.153624                -0.065402
# 242   Sat     No  0.098204              0.150152                -0.051948
# 243  Thur     No  0.159744              0.153492                 0.006252
#
# [244 rows x 5 columns]

print(tips.shape[0])
# Output:
# 244
```

## Summary

* Grouping follows split-apply-combine.
* Aggregation creates group summaries.
* Transformation returns row-aligned group calculations.
* Filtering selects groups that meet a condition.
* Named aggregation produces clear report tables.
* Pivot tables reshape grouped summaries across dimensions.
* MultiIndex, joins, and concatenation support grouped workflows but are not
  substitutes for understanding `groupby`.

## References

* [pandas: Group by](https://pandas.pydata.org/docs/user_guide/groupby.html)
* [pandas: Reshaping and pivot tables](https://pandas.pydata.org/docs/user_guide/reshaping.html)
* [pandas: Merge, join, concatenate and compare](https://pandas.pydata.org/docs/user_guide/merging.html)
