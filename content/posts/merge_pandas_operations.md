---
author: "Pavodive"
title: "Basic Merge Pandas Operations"
date: "2025-01-18"
description: "Pandas merge, pandas join"
tags: ["python", "pandas"]
categories: ["python"]
series: ["How-to"]
aliases: ["basic-pandas-merge-operations"]
ShowToc: true
TocOpen: true
---

Merging tables is one of the most common tasks when analyzing data. And yet, I always seem to forget how joins (merges) are done in pandas 🐼. So here’s a reminder for future me (and apparently for you, if you happened to land here 😉).

<!--more-->

# Merging with Pandas

## Our Base DataFrame

```python
import pandas as pd

items = {
    "color": ["red", "blue", "yellow", "black", "white"],
    "price": [50, 1200, 3, 40, 2],
    "date_created": ["1999-10-30 05:00:00",
                     "2002-08-21 08:23:00",
                     "2003-05-15 14:19:00",
                     "2006-11-29 11:21:00",
                     "2018-02-12 23:23:00"],
    "sku": ["1999-ROS-01",
            "2002-SKY-01",
            "2003-YOL-03",
            "2006-EBO-01",
            "2018-SNO-03"]
}

items_df = pd.DataFrame(items)
```

## Joining (Merging) Two Pandas DataFrames

Now, let’s merge (join) two dataframes using pandas. For this, we’ll create a dummy sales dataframe to compare with the one we already have. To illustrate the different types of joins, I’ve added a new SKU in the `sales` dataframe that doesn’t exist in `items_df`. I’ve also named the column `sku_sales` to demonstrate that the key columns in the two tables can have different names.

You’ll notice that I’m using `pd.merge` instead of `pd.join`. That’s because `pd.join` performs joins on indexes, whereas the more **flexible** `pd.merge` can join on columns.

```python
sales = pd.DataFrame({
    "sku_sales": ["1999-ROS-01", "2018-SNO-03", "2020-NAN-01", "2006-EBO-01"],
    "qty_sold": [100, 5, 7, 24]
})
```



### Left Join: All Items in Sales

A left join includes all rows from the `sales` dataframe. If a row in `sales` does not match a row in `items_df` (based on the join key), you’ll get `NaN` values for the unmatched columns. This means that we’ll see all items in `sales`, but we won’t have information about items in `items_df` that didn’t sell.

```python
merged_left = pd.merge(sales,
                       items_df,
                       left_on="sku_sales",
                       right_on="sku",
                       how="left"
                       )

print(merged_left)
#      sku_sales  qty_sold  color  price         date_created          sku
# 0  1999-ROS-01       100    red   50.0  1999-10-30 05:00:00  1999-ROS-01
# 1  2018-SNO-03         5  white    2.0  2018-02-12 23:23:00  2018-SNO-03
# 2  2020-NAN-01         7    NaN    NaN                  NaN          NaN
# 3  2006-EBO-01        24  black   40.0  2006-11-29 11:21:00  2006-EBO-01

```

> Tip: For both left and right joins, you may want to drop one of the duplicated key columns (`sku_sales` or `sku`). If you can’t remember how to drop a column (like me!), check [this post]({{< ref "basic_pandas_operations" >}} "Basic Pandas Operations").



### Right Join: All Items in `items_df`

A right join includes all rows from the `items_df` dataframe. In this case, you’ll lose information about `sales` rows that don’t match. For example, the `2020-NAN-01` item is not included in the result because it doesn’t exist in `items_df`.

```python
merged_right = pd.merge(sales,
                        items_df,
                        left_on="sku_sales",
                        right_on="sku",
                        how="right"
                        )

print(merged_right)
#      sku_sales  qty_sold   color  price         date_created          sku
# 0  1999-ROS-01     100.0     red     50  1999-10-30 05:00:00  1999-ROS-01
# 1          NaN       NaN    blue   1200  2002-08-21 08:23:00  2002-SKY-01
# 2          NaN       NaN  yellow      3  2003-05-15 14:19:00  2003-YOL-03
# 3  2006-EBO-01      24.0   black     40  2006-11-29 11:21:00  2006-EBO-01
# 4  2018-SNO-03       5.0   white      2  2018-02-12 23:23:00  2018-SNO-03
```

### Outer Join (Union): Items from Both DataFrames

Outer joins include all rows from both dataframes. If an item appears in only one of the dataframes, the unmatched columns will contain `NaN` values.

```python
merged_outer = pd.merge(sales,
                        items_df,
                        left_on="sku_sales",
                        right_on="sku",
                        how="outer"
                        )

print(merged_outer)

#      sku_sales  qty_sold   color   price         date_created          sku
# 0  1999-ROS-01     100.0     red    50.0  1999-10-30 05:00:00  1999-ROS-01
# 1          NaN       NaN    blue  1200.0  2002-08-21 08:23:00  2002-SKY-01
# 2          NaN       NaN  yellow     3.0  2003-05-15 14:19:00  2003-YOL-03
# 3  2006-EBO-01      24.0   black    40.0  2006-11-29 11:21:00  2006-EBO-01
# 4  2018-SNO-03       5.0   white     2.0  2018-02-12 23:23:00  2018-SNO-03
# 5  2020-NAN-01       7.0     NaN     NaN                  NaN          NaN
```

### Inner Join (Intersection): Items in Both `sales` and `items_df`

An inner join creates a new dataframe containing only the rows that exist in both dataframes (based on the join key).

```python
merged_inner = pd.merge(sales,
                        items_df,
                        left_on="sku_sales",
                        right_on="sku",
                        how="inner"
                        )

print(merged_inner)

#      sku_sales  qty_sold  color  price         date_created          sku
# 0  1999-ROS-01       100    red     50  1999-10-30 05:00:00  1999-ROS-01
# 1  2018-SNO-03         5  white      2  2018-02-12 23:23:00  2018-SNO-03
# 2  2006-EBO-01        24  black     40  2006-11-29 11:21:00  2006-EBO-01
```

### Cross Join: The Cartesian Product of Both DataFrames

A cross join might be useful in certain edge cases (though not for this example). It produces the Cartesian product of the two dataframes, meaning every row in `sales` is combined with every row in `items_df`. Notice that `left_on` and `right_on` are not needed for this operation.

```python
merged_cross = pd.merge(sales,
                        items_df,
                        how="cross"
                        )

print(merged_cross)

#       sku_sales  qty_sold   color  price         date_created          sku
# 0   1999-ROS-01       100     red     50  1999-10-30 05:00:00  1999-ROS-01
# 1   1999-ROS-01       100    blue   1200  2002-08-21 08:23:00  2002-SKY-01
# 2   1999-ROS-01       100  yellow      3  2003-05-15 14:19:00  2003-YOL-03
# 3   1999-ROS-01       100   black     40  2006-11-29 11:21:00  2006-EBO-01
# 4   1999-ROS-01       100   white      2  2018-02-12 23:23:00  2018-SNO-03
# 5   2018-SNO-03         5     red     50  1999-10-30 05:00:00  1999-ROS-01
# 6   2018-SNO-03         5    blue   1200  2002-08-21 08:23:00  2002-SKY-01
# 7   2018-SNO-03         5  yellow      3  2003-05-15 14:19:00  2003-YOL-03
# 8   2018-SNO-03         5   black     40  2006-11-29 11:21:00  2006-EBO-01
# 9   2018-SNO-03         5   white      2  2018-02-12 23:23:00  2018-SNO-03
# 10  2020-NAN-01         7     red     50  1999-10-30 05:00:00  1999-ROS-01
# 11  2020-NAN-01         7    blue   1200  2002-08-21 08:23:00  2002-SKY-01
# 12  2020-NAN-01         7  yellow      3  2003-05-15 14:19:00  2003-YOL-03
# 13  2020-NAN-01         7   black     40  2006-11-29 11:21:00  2006-EBO-01
# 14  2020-NAN-01         7   white      2  2018-02-12 23:23:00  2018-SNO-03
# 15  2006-EBO-01        24     red     50  1999-10-30 05:00:00  1999-ROS-01
# 16  2006-EBO-01        24    blue   1200  2002-08-21 08:23:00  2002-SKY-01
# 17  2006-EBO-01        24  yellow      3  2003-05-15 14:19:00  2003-YOL-03
# 18  2006-EBO-01        24   black     40  2006-11-29 11:21:00  2006-EBO-01
# 19  2006-EBO-01        24   white      2  2018-02-12 23:23:00  2018-SNO-03
```
