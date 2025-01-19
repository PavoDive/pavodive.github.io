---
author: "Pavodive"
title: "Basic Pandas Operations"
date: "2025-01-18"
description: "Pandas drop columns, reorder columns, change name of columns"
tags: ["python", "pandas"]
categories: ["python"]
series: ["How-to"]
aliases: ["basic-pandas-operations"]
ShowToc: true
TocOpen: true
---

There are some operations with pandas that I often forget. This site exists exactly for that: to remind me how to perform these tasks. In this post, I’ll cover some very basic pandas operations that I often forget, including dropping columns, reordering columns, and changing the names of columns.

<!--more-->

## Pandas

I won’t waste time explaining pandas 🐼, the powerful Python library for data processing. Pandas can handle many complex operations and is a must-have tool for anyone working with data.

At the time of writing this, I am much more proficient with R than pandas, which might explain why I repeatedly forget these simple operations.

## Our Base DataFrame

```python
import pandas as pd

items = {
    "color": ["red", "blue", "yellow", "black", "white"],
    "name": ["rose", "sky", "yolk", "ebony", "snow"],
    "price": [50, 1200, 3, 40, 2],
    "date_created": ["1999-10-30 05:00:00", 
                     "2002-08-21 08:23:00", 
                     "2003-05-15 14:19:00", 
                     "2006-11-29 11:21:00", 
                     "2018-02-12 23:23:00"
                     ],
    "useless_column": ["data", "data", "data", "data", "data"],
    "id": ["1999-ROS-01",
           "2002-SKY-01",
           "2003-YOL-03",
           "2006-EBO-01",
           "2018-SNO-03"
           ]
}

items_df = pd.DataFrame(items)
```

Our base dataframe is a simple table containing miscellaneous data about items. There’s even an unnecessary column, which we’ll remove shortly.

## Dropping Columns

Let’s get rid of the useless column. The `inplace=True` parameter is self-explanatory: it removes the column in place rather than returning a new object. This way, the original dataframe is directly updated.

```python
items_df.drop(columns=["useless_column"], inplace=True)
```

## Reordering Columns

Sometimes, I need to export dataframes to tables or CSV files where the ordering of columns matters for end users. Reordering columns is as simple as passing a list of column names in the desired order:

```python
name_order = ["id", "date_created", "name", "color", "price"]
items_df = items_df[name_order]
```

## Changing the Names of Columns

Another operation I often need is renaming columns. To rename columns, pass a dictionary in the format `{"old_name": "new_name"}` to the `rename` method. 

```python
items_df.rename(columns={"id": "sku", "color": "Color"}, inplace=True)
```


