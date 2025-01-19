---
author: "Pavodive"
title: "Basic Time Pandas Operations"
date: "2025-01-18"
description: "pandas date conversion filtering, pandas date filtering"
tags: ["python", "pandas"]
categories: ["python"]
series: ["How-to"]
aliases: ["basic-pandas-dates"]
ShowToc: true
TocOpen: true
---

Dates and date-time objects are some of the most challenging data types to work with. They can come in a variety of formats (long live [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601)!), may have different time zones, and introduce considerable complexity as a result.

<!--more-->

Aside from variations in format, such discrepancies can cause massive issues:

![](https://imgs.xkcd.com/comics/formatting_meeting.png)

Moreover, time zones add yet another layer of potential confusion. Let’s see how pandas 🐼 handles this.

## Our Base DataFrame

```python
import pandas as pd

items = {
    "name": ["rose", "sky", "yolk", "ebony", "snow"],
    "price": [50, 1200, 3, 40, 2],
    "date_created": ["1999-10-30 05:00:00", 
                     "2002-08-21 08:23:00", 
                     "2003-05-15 14:19:00", 
                     "2006-11-29 11:21:00", 
                     "2018-02-12 23:23:00"
                     ],
}

items_df = pd.DataFrame(items)
```

## Converting Strings to Dates in Pandas

Let’s check how pandas interprets the data types in our dataframe:

```python
print(items_df.dtypes)

# Output:
# name            object
# price            int64
# date_created    object
```

As we can see, the `date_created` column is treated as an `object` (likely because it contains strings). To handle it as a date-time, we must convert it:

```python
items_df["date_created"] = pd.to_datetime(items_df["date_created"])
print(items_df.dtypes)

# Output:
# name            object
# price            int64
# date_created    datetime64[ns]
```

Now the `date_created` column is properly processed as `datetime64[ns]`.

## Filtering a DataFrame by a Date

Pandas’ handling of dates and times remains a bit mysterious to me. I understand that dates and times are inherently complex, but pandas' behavior can sometimes feel confusing. For instance, consider [this Stack Overflow question](https://stackoverflow.com/questions/13703720/converting-between-datetime-timestamp-and-datetime64) and Quant’s detailed answer. While the diagram isn’t perfectly up-to-date (as of 2025-01-19), it’s still quite relevant and demonstrates the complexity involved!

Here’s one such surprising behavior: 

While our `date_created` column shows a type of `datetime64[ns]`, attempting to access its individual elements reveals something slightly different:

```python
print(type(items_df["date_created"].iloc[1]))
# Output: <class 'pandas._libs.tslibs.timestamps.Timestamp'>
```

When we try to filter this column using a simple string date, pandas automatically coerces the string to a date, and the operation works as expected:

```python
start_date_1 = "2000-01-01"
filtered_items_1 = items_df[items_df["date_created"] >= start_date_1]
print(filtered_items_1)
```

However, if the string includes both a time and a time zone, pandas no longer performs automatic coercion, causing the operation to fail:

```python
start_date_2 = "2000-01-01 00:00:00-05:00"

try:
    filtered_items_2 = items_df[items_df["date_created"] >= start_date_2]
except Exception as e:
    print(e)
    filtered_items_2 = "Nothing to see here"
finally:
    print(filtered_items_2)

# Output:
# Invalid comparison between dtype=datetime64[ns] and str
# Nothing to see here
```

Pandas, perhaps cautiously, avoids coercion in this case, leaving the responsibility to us. Let’s fix this manually by converting the string to a timestamp using the same method we used earlier:

```python
start_date_2 = pd.to_datetime(start_date_2)
print(type(start_date_2) == type(items_df["date_created"].iloc[1]))
# Output: True 
# Both are <class 'pandas._libs.tslibs.timestamps.Timestamp'>
```

At this point, the two values appear to be of the same type, but they still can’t be compared:

```python
try:
    items_df['date_created'].iloc[1] > start_date_2
except Exception as e:
    print(e)
finally:
    pass
# Output:
# Cannot compare tz-naive and tz-aware timestamps
```

The error occurs because one value is **timezone-naive** (lacking information about the time zone), and the other is **timezone-aware**. To proceed, we must convert one of them to match the other.

### Solution 1: Convert `start_date_2` to Timezone-Naive

```python
start_date_naive = start_date_2.tz_localize(None)

try:
    filtered_items_2 = items_df[items_df["date_created"] >= start_date_naive]
except Exception as e:
    print(e)
    filtered_items_2 = "Nothing to see here"
finally:
    print(filtered_items_2)

# The filtering succeeds! 🥳
```

### Solution 2: Convert `items_df["date_created"]` to Timezone-Aware

```python
# Create a new column (optional)
items_df["new_date_created"] = items_df["date_created"].dt.tz_localize("America/Bogota")

# Now we can compare against our original timezone-aware variable
try:
    filtered_items_2 = items_df[items_df["new_date_created"] >= start_date_2]
except Exception as e:
    print(e)
    filtered_items_2 = "Nothing to see here"
finally:
    print(filtered_items_2)

# It works! 🥳
```

A key insight here is the `.dt` accessor, which acts as a gateway to apply date-time operations on pandas Series. According to the [documentation](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.dt.html), the `.dt` accessor "_returns a Series indexed like the original Series. Raises TypeError if the Series does not contain datetimelike values._" This is essential because `tz_localize` operates on datetime objects, not directly on Series (thanks [wordsforthewise](https://stackoverflow.com/questions/22800079/converting-time-zone-pandas-dataframe) for the hint!).

Dates and times can be tricky, but pandas provides powerful tools to manage them. With some care and awareness of datetime intricacies, you’ll master these challengesin no time! 🐼
