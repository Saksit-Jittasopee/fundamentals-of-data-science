
# Lab: Data Grouping and Aggregation

**General Instructions**

- Store each answer in the specified variable so it can be checked.
- Choose your own pandas approach. Your code does not need to match the solution.
  Hints suggest possible approaches; they do not prescribe the only valid method.
- Results should contain the requested groups, measurements, and observations.
  Equivalent row/column ordering, index layouts, compatible dtypes, and minor
  floating-point rounding differences are acceptable. Keep group labels
  identifiable and use any output column names explicitly requested by a task.
- Expected outputs illustrate the results; you do not need to reproduce their
  exact printed formatting. Printing is optional.
- Keep `df` unchanged unless a task asks you to add a column. Work on a copy
  when creating a modified DataFrame.
- Complete tasks independently except for the dependencies noted below.

**Sequence note:** Task 2.3 adds `Age_group` to `df`; retain it in later
DataFrame tasks. Task 3.2 reuses `gt_mean` from Task 2.4, and optional Task 5.6
reuses `fare_range` from Task 3.3. The optional second approach to Task 6 uses
the same missing-value setup as Task 6.

---

## Task 1: Load Titanic Dataset

Import pandas as `pd` and NumPy as `np`, then read `titanic-data-clean.csv` into
a DataFrame named `df`.

---


## Task 2: GroupBy


### Task 2.1: Compute the mean of the `Fare` of passengers from each `Pclass`

Store your answer in `result2_1`.

Expected output:

```
Pclass
1    84.154687
2    20.662183
3    13.675550
Name: Fare, dtype: float64
```


### Task 2.2: Compute the standard deviation of the `Fare` of passengers from each `Pclass` and `Sex`

Store your answer in `result2_2`.

Expected output:

```
Pclass  Sex   
1       female    74.259988
        male      77.548021
2       female    10.891796
        male      14.922235
3       female    11.690314
        male      11.681696
Name: Fare, dtype: float64
```


### Task 2.3: Compute the mean of the `Fare` of passengers from each `Pclass` and `Age_group`

Hint: please refer to the binning technique in the previous lecture to group passengers in the following bins:

```python
bins = [0, 25, 35, 60, 100]
group_names = ["Youth", "YoungAdult", "MiddleAged", "Senior"]
```

Each interval includes its upper endpoint: Youth `(0, 25]`, YoungAdult
`(25, 35]`, MiddleAged `(35, 60]`, and Senior `(60, 100]`.

Expected output:

```python
print(df['Age_group'])

# 0           Youth
# 1      MiddleAged
# 2      YoungAdult
# 3      YoungAdult
# 4      YoungAdult
#           ...    
# 886    YoungAdult
# 887         Youth
# 888    YoungAdult
# 889    YoungAdult
# 890    YoungAdult
# Name: Age_group, Length: 891, dtype: category
# Categories (4, object): ['Youth' < 'YoungAdult' < 'MiddleAged' < 'Senior']
```

Once you have added the `Age_group` column into the dataframe, `df`, write the code to compute the mean of the `Fare` of passengers from each `Pclass` and `Age_group`.

Store your answer in `result2_3`.

Expected output:

```
                         Fare
Pclass Age_group             
1      Youth       116.987200
       YoungAdult   78.512619
       MiddleAged   76.983334
       Senior       59.969050
2      Youth        25.102309
       YoungAdult   17.535386
       MiddleAged   19.760638
       Senior       10.500000
3      Youth        13.865335
       YoungAdult   13.727935
       MiddleAged   13.334195
       Senior        7.820000
```


### Task 2.4: Create a Custom Aggregation Function

For each combination of `Embarked` and `Sex`, count passengers whose `Fare`
is strictly greater than their group's mean fare. Define and use a custom
function named `gt_mean` for this calculation.

Hint: you may find `np.sum(values > values.mean())` useful for counting,
where `values` represents the group's values passed to your function.

Store your answer in `result2_4`.

Expected output:

```
                 Fare
Embarked Sex         
C        female    30
         male      25
Q        female     8
         male      12
S        female    53
         male     137
UNK      female     0
```


## Task 3: Multiple Aggregation Functions


### Task 3.1: Compute the mean and standard deviation of the `Fare` of passengers from each `Pclass` and `Sex`

Store your answer in `result3_1`.

Expected output:

```
                     Fare           
                     mean        std
Pclass Sex                          
1      female  106.125798  74.259988
       male     67.226127  77.548021
2      female   21.970121  10.891796
       male     19.741782  14.922235
3      female   16.118810  11.690314
       male     12.661633  11.681696
```


### Task 3.2: Use different aggregation functions on different columns

Group by `Pclass` and `Sex`, then apply these aggregations:

- `Fare`: `min`, `max`, `mean`, `std`
- `Age`: `gt_mean` (from Task 2.4)
- `Parch`: `count`

Store your answer in `result3_2`.

Expected output:

```
                  Fare                                      Age Parch
                   min       max        mean        std gt_mean count
Pclass Sex                                                           
1      female  25.9292  512.3292  106.125798  74.259988      44    94
       male     0.0000  512.3292   67.226127  77.548021      53   122
2      female  10.5000   65.0000   21.970121  10.891796      38    76
       male     0.0000   73.5000   19.741782  14.922235      47   108
3      female   6.7500   69.5500   16.118810  11.690314      81   144
       male     0.0000   69.5500   12.661633  11.681696     202   347
```


### Task 3.3: A Summary Table with Named Columns

Create one row per `Pclass` with these columns:

- `Pclass`: passenger class;
- `passengers`: number of passengers;
- `average_fare`: mean fare;
- `fare_range`: difference between the highest and lowest fares.

Define and use a custom function named `fare_range` for the last calculation.
Store the table in `result3_3`.

Hint: named aggregation can help you give summary columns meaningful names.

Expected output:

```
   Pclass  passengers  average_fare  fare_range
0       1         216     84.154687    512.3292
1       2         184     20.662183     73.5000
2       3         491     13.675550     69.5500
```

---

## Task 4: Top Fares Within Each Class

Find the three passengers with the largest `Fare` in each `Pclass` and store
their records in `result4_1`. Keep the passenger details and make each
passenger's class identifiable, either in the index or a column.

Return exactly three passengers per class. Where fares tie, any selection
among the tied passengers is acceptable; your passenger IDs and row order
may differ from the example.

Hint: You may find `df.sort_values` with `ascending=False` useful.

One valid result is shown below using selected columns for readability:

```python
print(result4_1[["PassengerId", "Sex", "Fare", "Age_group"]])
# Output:
#             PassengerId     Sex      Fare   Age_group
# Pclass
# 1      258          259  female  512.3292  YoungAdult
#        679          680    male  512.3292  MiddleAged
#        737          738    male  512.3292  YoungAdult
# 2      72            73    male   73.5000       Youth
#        120          121    male   73.5000       Youth
#        385          386    male   73.5000       Youth
# 3      159          160    male   69.5500  YoungAdult
#        180          181  female   69.5500  YoungAdult
#        201          202    male   69.5500  YoungAdult
```


## Task 5: Pivot Table


### Task 5.1: Create a pivot table to aggregate the `Fare`, `Parch`, and `SibSp` of passengers in each `Pclass` group using mean

Store your answer in `result5_1`.

Expected output:

```
             Fare     Parch     SibSp
Pclass                               
1       84.154687  0.356481  0.416667
2       20.662183  0.380435  0.402174
3       13.675550  0.393075  0.615071
```


### Task 5.2: Extend the code from Task 5.1 to aggregate using `count` and `mean`

Store your answer in `result5_2`.

Hint: you may find the `aggfunc` argument of `df.pivot_table` useful.

Expected output:

```
       count                   mean                    
        Fare Parch SibSp       Fare     Parch     SibSp
Pclass                                                 
1        216   216   216  84.154687  0.356481  0.416667
2        184   184   184  20.662183  0.380435  0.402174
3        491   491   491  13.675550  0.393075  0.615071
```


### Task 5.3: Extend the code from Task 5.1 to further compute the mean of each sex for `Fare`, `Parch`, `SibSp`

Store your answer in `result5_3`.

Hint: you may find the `columns` argument of `df.pivot_table` useful.

Expected output:

```
              Fare                Parch               SibSp          
Sex         female       male    female      male    female      male
Pclass                                                               
1       106.125798  67.226127  0.457447  0.278689  0.553191  0.311475
2        21.970121  19.741782  0.605263  0.222222  0.486842  0.342593
3        16.118810  12.661633  0.798611  0.224784  0.895833  0.498559
```


### Task 5.4: Extend the code from Task 5.3 to also compute the mean from all passengers

Store your answer in `result5_4`.

Hint: you may find the `margins` argument of `df.pivot_table` useful.

```
              Fare                           Parch                      \
Sex         female       male        All    female      male       All   
Pclass                                                                   
1       106.125798  67.226127  84.154687  0.457447  0.278689  0.356481   
2        21.970121  19.741782  20.662183  0.605263  0.222222  0.380435   
3        16.118810  12.661633  13.675550  0.798611  0.224784  0.393075   
All      44.479818  25.523893  32.204208  0.649682  0.235702  0.381594   

           SibSp                      
Sex       female      male       All  
Pclass                                
1       0.553191  0.311475  0.416667  
2       0.486842  0.342593  0.402174  
3       0.895833  0.498559  0.615071  
All     0.694268  0.429809  0.523008
```


### Task 5.5: Different Aggregations and Named Columns

Create a pivot table showing median `Fare` and maximum `Age` for each
combination of class and sex. Place classes in rows and sex categories in
columns. Label the two measurements `median_fare` and `max_age`.

Store your answer in `result5_5`.

Hint: `aggfunc` can accept different functions for different columns.
`rename` can help with the output labels.

Expected output:

```
       max_age       median_fare
Sex     female  male      female     male
Pclass
1         63.0  80.0    82.66455  41.2625
2         57.0  70.0    22.00000  13.0000
3         63.0  74.0    12.47500   7.9250
```

### Task 5.6 (Optional): A Custom Function in a Pivot Table

Create a pivot table showing the range of `Fare` for each class and sex,
using your `fare_range` function from Task 3.3. Place classes in rows and sex
categories in columns. Store your answer in `result5_6`.

Hint: a custom function can be passed to `aggfunc`.

Expected output:

```
Sex     female      male
Pclass
1        486.4  512.3292
2         54.5   73.5000
3         62.8   69.5500
```

---

## Task 6: Fill Missing Fares

Fill missing `Fare` values using the mean of the remaining observed fares in
each passenger's `Pclass`. Store the completed DataFrame in `df2`.
Keep `df` unchanged and preserve all non-missing fares and other columns in
`df2`. Choose your own method for filling the missing values.

Hint: consider how `transform` or `map` could match a group statistic to
each passenger.

Start with this code to create reproducible missing values:

```python
df2 = df.copy()
drop_idx = df.sample(frac=0.1, random_state=42).index
df2.loc[drop_idx, "Fare"] = np.nan
```

Before filling, the selected fares are missing:

```python
print(df2.loc[drop_idx, ["PassengerId", "Pclass", "Fare"]])
# Output:
#      PassengerId  Pclass  Fare
# 709          710       3   NaN
# 439          440       2   NaN
# 840          841       3   NaN
# 720          721       2   NaN
# 39            40       3   NaN
# ..           ...     ...   ...
# 174          175       1   NaN
# 493          494       1   NaN
# 215          216       1   NaN
# 309          310       1   NaN
# 822          823       1   NaN
#
# [89 rows x 3 columns]
```

After filling, the same rows should look like this:

```python
print(df2.loc[drop_idx, ["PassengerId", "Pclass", "Fare"]])
# Output:
#      PassengerId  Pclass       Fare
# 709          710       3  13.809933
# 439          440       2  20.327466
# 840          841       3  13.809933
# 720          721       2  20.327466
# 39            40       3  13.809933
# ..           ...     ...        ...
# 174          175       1  85.934486
# 493          494       1  85.934486
# 215          216       1  85.934486
# 309          310       1  85.934486
# 822          823       1  85.934486
#
# [89 rows x 3 columns]
```

### Optional: Solve Task 6 Another Way

Solve the same missing-fare problem using a different approach. Start from
`df` with the same fares missing at `drop_idx`, and store the second result in
`df2_map`. Check that both approaches produce equivalent results.

```python
print(df2_map.equals(df2))
# Output:
# True
```

This checks exact equality. A `False` result does not necessarily mean your
answer is wrong: equivalent results may differ in ordering, dtypes, or minor
numerical rounding.

---

## Task 7: Row-Level Comparison and Group Filtering

Use `df`, not the modified fares in `df2`. These two tasks are independent of
each other. Keep `df` unchanged.

### Task 7.1: Compare Each Fare with Its Class Median

Create `result7_1` with all rows and columns of `df`, plus a column named
`fare_vs_class_median`: each passenger's fare minus their class's median fare.

Hint: `transform` can provide a group statistic for each original row.

The displayed result includes only three columns for readability.

```python
print(result7_1[["Pclass", "Fare", "fare_vs_class_median"]])
# Output:
#      Pclass     Fare  fare_vs_class_median
# 0         3   7.2500               -0.8000
# 1         1  71.2833               10.9958
# 2         3   7.9250               -0.1250
# 3         1  53.1000               -7.1875
# 4         3   8.0500                0.0000
# ..      ...      ...                   ...
# 886       2  13.0000               -1.2500
# 887       1  30.0000              -30.2875
# 888       3  23.4500               15.4000
# 889       1  30.0000              -30.2875
# 890       3   7.7500               -0.3000
#
# [891 rows x 3 columns]
```

### Task 7.2: Keep Groups with at Least 40 Passengers

Keep all passengers whose combination of `Embarked` and `Sex` contains at
least 40 passengers. Store their records in `result7_2`, retaining the original
columns and passenger identities.

Hint: `GroupBy.filter` can retain all rows from groups that satisfy a condition.

The retained group sizes and total number of passengers are shown below:

```
Embarked  Sex
C         female     73
          male       95
Q         male       41
S         female    203
          male      441
dtype: int64
```

```
853
```
