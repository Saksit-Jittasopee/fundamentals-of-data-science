# Lab: Introduction to Pandas

**General Instructions (Read Carefully):**

- For each task, store your answer in the variable name specified in the task (e.g., `df`, `result2_1`, etc.).
- Do **not** change the name of the variables. Your code will be autograded based on these names.
- Unless a task explicitly asks you to modify a DataFrame (e.g., add a column), do **not** mutate or overwrite it. If you need to change a DataFrame for a later task, save the original as a new variable (e.g., `df_task1 = df.copy()`) before making changes.
- Each task should be completed independently, unless otherwise stated. Do **not** rely on the output of previous tasks unless the instructions say so.
- You may print your variable to check your answer with the expected output provided, but it is not required for grading.
- Use only the required columns or rows as specified in each task.
- You may use any valid pandas method unless a specific method is requested.
- Your code should be reproducible and not rely on previous outputs unless stated.

**Sequence note:** Tasks 2–3 use the small `df` created in Task 1. Task 4 then
replaces `df` with the full CSV data, which Tasks 5–7 use. Tasks T6-4 and T7-1
through T7-4 also require the `price_per_head` column created in T6-3.

**Tips for Autograder-Friendly Code:**

- Always use the exact variable names specified.
- If you need to mutate a DataFrame (e.g., add a column), save a copy of the original DataFrame in a new variable before making changes, so earlier tasks can still be checked.
- If you are unsure, ask yourself: "If the autograder checks my variable, will it match the expected output exactly?"

## Task 1: Create a DataFrame

**T1**: Create a DataFrame named `df` for the following sample of restaurant tipping data. The empty cells represent Null or Not a Number (NaN) values. You may use `None` or `np.nan` for these. Your DataFrame should have the columns: `'total_bill', 'tip', 'smoker', 'day', 'time', 'size'`.

| total_bill | tip | smoker | day | time | size |
| --- | --- | --- | --- | --- | --- |
| 16.99 | 1.01 | No | Sun | Dinner | 2 |
| 10.34 |  | No |  | Dinner | 3 |
| 21.01 | 3.5 | Yes | Mon |  | 3 |
| 23.68 | 3.31 |  | Wed | Dinner | 2 |
| 24.59 | 3.61 | No | Sat | Lunch |  |

## Task 2: Select Columns

**T2-1**: Select `'tip'` column using `df[column]`.

Store your answer in `result2_1`.

```output
0    1.01
1     NaN
2    3.50
3    3.31
4    3.61
Name: tip, dtype: float64
```

**T2-2**: Select `'size','tip','day'` columns using `df[column]`.

Store your answer in `result2_2`.

```output
size   tip   day
0   2.0  1.01   Sun
1   3.0   NaN  None
2   3.0  3.50   Mon
3   2.0  3.31   Wed
4   NaN  3.61   Sat
```

**T2-3**: Select `'tip','size','day'` columns using `df.loc[:, cols]`.

Store your answer in `result2_3`.

```output
tip  size   day
0  1.01   2.0   Sun
1   NaN   3.0  None
2  3.50   3.0   Mon
3  3.31   2.0   Wed
4  3.61   NaN   Sat
```

**T2-4**: Select `'time','smoker','day'` columns using `df.iloc[:, cols]`.

Store your answer in `result2_4`.

```output
time smoker   day
0  Dinner     No   Sun
1  Dinner     No  None
2    None    Yes   Mon
3  Dinner    NaN   Wed
4   Lunch     No   Sat
```

## Task 3: Select Rows

**T3-1**: Select rows with `tip > 2` via `df[condition]`.
Store your answer in `result3_1`.

```output
total_bill   tip smoker  day    time  size
2       21.01  3.50    Yes  Mon    None   3.0
3       23.68  3.31    NaN  Wed  Dinner   2.0
4       24.59  3.61     No  Sat   Lunch   NaN
```

**T3-2**: Select rows with index 1, 3 via `df.iloc[list_of_indices]`

Store your answer in `result3_2`.

```output
total_bill   tip smoker   day    time  size
1       10.34   NaN     No  None  Dinner   3.0
3       23.68  3.31    NaN   Wed  Dinner   2.0
```

**T3-3**: Select rows with label 1, 3 via `df.loc[list_of_labels]`

Store your answer in `result3_3`.

```output
total_bill   tip smoker   day    time  size
1       10.34   NaN     No  None  Dinner   3.0
3       23.68  3.31    NaN   Wed  Dinner   2.0
```

## Task 4: Read CSV data

**T4**: Use `pd.read_csv` to read the local `tips.csv` file into a DataFrame named `df`.

```output
total_bill   tip smoker   day    time  size
0         16.99  1.01     No   Sun  Dinner     2
1         10.34  1.66     No   Sun  Dinner     3
2         21.01  3.50     No   Sun  Dinner     3
3         23.68  3.31     No   Sun  Dinner     2
4         24.59  3.61     No   Sun  Dinner     4
..          ...   ...    ...   ...     ...   ...
239       29.03  5.92     No   Sat  Dinner     3
240       27.18  2.00    Yes   Sat  Dinner     2
241       22.67  2.00    Yes   Sat  Dinner     2
242       17.82  1.75     No   Sat  Dinner     2
243       18.78  3.00     No  Thur  Dinner     2

[244 rows x 6 columns]
```

## Task 5: Select Rows based on Complex Conditions

**T5-1**: Select tipping data on Friday.
Store your answer in `result5_1`.

```output
total_bill   tip smoker  day    time  size
90        28.97  3.00    Yes  Fri  Dinner     2
91        22.49  3.50     No  Fri  Dinner     2
92         5.75  1.00    Yes  Fri  Dinner     2
93        16.32  4.30    Yes  Fri  Dinner     2
94        22.75  3.25     No  Fri  Dinner     2
95        40.17  4.73    Yes  Fri  Dinner     4
96        27.28  4.00    Yes  Fri  Dinner     2
97        12.03  1.50    Yes  Fri  Dinner     2
98        21.01  3.00    Yes  Fri  Dinner     2
99        12.46  1.50     No  Fri  Dinner     2
100       11.35  2.50    Yes  Fri  Dinner     2
101       15.38  3.00    Yes  Fri  Dinner     2
220       12.16  2.20    Yes  Fri   Lunch     2
221       13.42  3.48    Yes  Fri   Lunch     2
222        8.58  1.92    Yes  Fri   Lunch     1
223       15.98  3.00     No  Fri   Lunch     3
224       13.42  1.58    Yes  Fri   Lunch     2
225       16.27  2.50    Yes  Fri   Lunch     2
226       10.09  2.00    Yes  Fri   Lunch     2
```

**T5-2**: Select tipping data on the weekend (i.e., Saturday and Sunday).

Store your answer in `result5_2`.

```output
total_bill   tip smoker  day    time  size
0         16.99  1.01     No  Sun  Dinner     2
1         10.34  1.66     No  Sun  Dinner     3
2         21.01  3.50     No  Sun  Dinner     3
3         23.68  3.31     No  Sun  Dinner     2
4         24.59  3.61     No  Sun  Dinner     4
..          ...   ...    ...  ...     ...   ...
238       35.83  4.67     No  Sat  Dinner     3
239       29.03  5.92     No  Sat  Dinner     3
240       27.18  2.00    Yes  Sat  Dinner     2
241       22.67  2.00    Yes  Sat  Dinner     2
242       17.82  1.75     No  Sat  Dinner     2

[163 rows x 6 columns]
```

**T5-3**: From T5-2, show only the `'total_bill'` column.

Store your answer in `result5_3`.

```output
0      16.99
1      10.34
2      21.01
3      23.68
4      24.59
       ...  
238    35.83
239    29.03
240    27.18
241    22.67
242    17.82
Name: total_bill, Length: 163, dtype: float64
```

**T5-4**: Select tipping data from the tables with a smoker (i.e., `df['smoker']=='Yes'`) and the total bill is less than 10 (i.e., `df['total_bill'] < 10`).

Store your answer in `result5_4`.

```output
total_bill   tip smoker  day    time  size
67         3.07  1.00    Yes  Sat  Dinner     1
92         5.75  1.00    Yes  Fri  Dinner     2
172        7.25  5.15    Yes  Sun  Dinner     2
178        9.60  4.00    Yes  Sun  Dinner     2
218        7.74  1.44    Yes  Sat  Dinner     2
222        8.58  1.92    Yes  Fri   Lunch     1
```

**T5-5**: From T5-4: show only the `'tip','size'` columns.

Store your answer in `result5_5`.

```output
tip  size
67   1.00     1
92   1.00     2
172  5.15     2
178  4.00     2
218  1.44     2
222  1.92     1
```

## Task 6: Compute descriptive statistics

**T6-1**: Calculate the mean of the `'total_bill'` during the weekend.
Store your answer in `result6_1`.

```output
20.893006134969326
```

**T6-2**: Calculate the mean of the `'tip'` from the tables with a smoker.

Store your answer in `result6_2`.

```output
3.008709677419355
```

**T6-3**: Create a new column, named `'price_per_head'`, representing the price per head. This can be calculated by dividing the `'total_bill'` with the `'size'`.

Store your answer in `df` (overwrite the DataFrame with the new column).

```output
total_bill   tip smoker   day    time  size  price_per_head
0         16.99  1.01     No   Sun  Dinner     2        8.495000
1         10.34  1.66     No   Sun  Dinner     3        3.446667
2         21.01  3.50     No   Sun  Dinner     3        7.003333
3         23.68  3.31     No   Sun  Dinner     2       11.840000
4         24.59  3.61     No   Sun  Dinner     4        6.147500
..          ...   ...    ...   ...     ...   ...             ...
239       29.03  5.92     No   Sat  Dinner     3        9.676667
240       27.18  2.00    Yes   Sat  Dinner     2       13.590000
241       22.67  2.00    Yes   Sat  Dinner     2       11.335000
242       17.82  1.75     No   Sat  Dinner     2        8.910000
243       18.78  3.00     No  Thur  Dinner     2        9.390000

[244 rows x 7 columns]
```

**T6-4**: Calculate the maximum `'price_per_head'` on Sunday.

Store your answer in `result6_4`.

```output
20.275
```

## Task 7: Sorting

**T7-1:** Sort by `'tip'` in an ascending order.
Store your answer in `result7_1`.

```output
total_bill    tip smoker   day    time  size  price_per_head
67         3.07   1.00    Yes   Sat  Dinner     1        3.070000
92         5.75   1.00    Yes   Fri  Dinner     2        2.875000
111        7.25   1.00     No   Sat  Dinner     1        7.250000
236       12.60   1.00    Yes   Sat  Dinner     2        6.300000
0         16.99   1.01     No   Sun  Dinner     2        8.495000
..          ...    ...    ...   ...     ...   ...             ...
141       34.30   6.70     No  Thur   Lunch     6        5.716667
59        48.27   6.73     No   Sat  Dinner     4       12.067500
23        39.42   7.58     No   Sat  Dinner     4        9.855000
212       48.33   9.00     No   Sat  Dinner     4       12.082500
170       50.81  10.00    Yes   Sat  Dinner     3       16.936667

[244 rows x 7 columns]
```

**T7-2:** Sort by `'tip'` in descending order.

Store your answer in `result7_2`.

```output
total_bill    tip smoker   day    time  size  price_per_head
170       50.81  10.00    Yes   Sat  Dinner     3       16.936667
212       48.33   9.00     No   Sat  Dinner     4       12.082500
23        39.42   7.58     No   Sat  Dinner     4        9.855000
59        48.27   6.73     No   Sat  Dinner     4       12.067500
141       34.30   6.70     No  Thur   Lunch     6        5.716667
..          ...    ...    ...   ...     ...   ...             ...
0         16.99   1.01     No   Sun  Dinner     2        8.495000
67         3.07   1.00    Yes   Sat  Dinner     1        3.070000
92         5.75   1.00    Yes   Fri  Dinner     2        2.875000
111        7.25   1.00     No   Sat  Dinner     1        7.250000
236       12.60   1.00    Yes   Sat  Dinner     2        6.300000

[244 rows x 7 columns]
```

**T7-3:** Sort by `'total_bill'` and `'tip'` in descending order.

Store your answer in `result7_3`.

```output
total_bill    tip smoker   day    time  size  price_per_head
170       50.81  10.00    Yes   Sat  Dinner     3       16.936667
212       48.33   9.00     No   Sat  Dinner     4       12.082500
59        48.27   6.73     No   Sat  Dinner     4       12.067500
156       48.17   5.00     No   Sun  Dinner     6        8.028333
182       45.35   3.50    Yes   Sun  Dinner     3       15.116667
..          ...    ...    ...   ...     ...   ...             ...
149        7.51   2.00     No  Thur   Lunch     2        3.755000
172        7.25   5.15    Yes   Sun  Dinner     2        3.625000
111        7.25   1.00     No   Sat  Dinner     1        7.250000
92         5.75   1.00    Yes   Fri  Dinner     2        2.875000
67         3.07   1.00    Yes   Sat  Dinner     1        3.070000

[244 rows x 7 columns]
```

**T7-4:** Show the `'total_bill'`, `'tip'` and `'price_per_head'` columns during the weekend sorted by `'price_per_head'` in an ascending order.

Store your answer in `result7_4`.

```output
total_bill    tip  price_per_head
67         3.07   1.00        3.070000
16        10.33   1.67        3.443333
1         10.34   1.66        3.446667
172        7.25   5.15        3.625000
218        7.74   1.44        3.870000
..          ...    ...             ...
237       32.83   1.17       16.415000
175       32.90   3.11       16.450000
170       50.81  10.00       16.936667
179       34.63   3.55       17.315000
184       40.55   3.00       20.275000

[163 rows x 3 columns]
```
