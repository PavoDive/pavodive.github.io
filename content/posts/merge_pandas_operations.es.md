---
author: "Pavodive"
title: "Operaciones Básicas de Cruce de Tablas con Pandas"
date: "2025-01-18"
description: "Merge con pandas, join con pandas, cruce de tablas con pandas"
tags: ["python", "pandas"]
categories: ["python"]
series: ["How-to"]
aliases: ["operaciones-basicas-cruce-pandas"]
ShowToc: true
TocOpen: true
---

Cruzar tablas es una de las tareas más básicas cuando se analizan datos. Y sin embargo, yo siempre parezco olvidar como se hacen los cruces de tablas (merges o joins) en pandas 🐼. Así que aquí esta este recordatorio para mi yo futuro (y aparentemente para vos, que aterrizaste aquí 😉).

<!--more-->

# Cruzando Tablas con Pandas

## Nuestra DataFrame Base

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

## Uniendo (Cruzando, Merging, Joining) Dos DataFrames con Pandas

Ahora, vamos a cruzar dos dataframes utilizando pandas. Para esto, vamos a crear una dataframe de ventas para compararla con la que ya tenemos. Para ilustrar los diferentes tipos de uniones, Añadí un nuevo SKU en el dataframe `sales` (ventas) que no existe en `items_df`. También llamé la columna `sku_sales` para demostrar que las columnas clave (las que usaremos para el cruce) pueden tener diferentes nombres en las dos tablas.

Vas a notar que estoy usando `pd.merge` en lugar de `pd.join`. Eso es porque `pd.join` hace las uniones en los índices, mientras que el más **flexible** `pd.merge` puede hacer uniones en columnas.

```python
sales = pd.DataFrame({
    "sku_sales": ["1999-ROS-01", "2018-SNO-03", "2020-NAN-01", "2006-EBO-01"],
    "qty_sold": [100, 5, 7, 24]
})
```

### Unión Izquierda: Todos los Ítems en Ventas (Sales)

Una unión izquierda incluye todas las filas de la dataframe `ventas`. Si una fila en `ventas` no concuerda con una fila en `items_df` (basado en la clave de unión), vas a obtener `NaN` para los valores de las columnas que no concuerdan. Esto significa que vamos a ver todos los ítems en `sales`, pero que no vamos a ver información sobre los ítems en `items_df` que no vendieron.

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

> Tip: Para las uniones izquierda y derecha, es posible que querás descartar una de las columnas clave duplicadas (`sku_sales` o `sku`). Si no podés recordar cómo descartar una columna (¡como yo!), chequeá [este artículo]({{< ref "basic_pandas_operations.es" >}} "Operaciones Básicas con Pandas").

### Unión Derecha: Todos los Ítems en `items_df`

Una unión derecha incluye todas las filas de `items_df`. En este caso, vas a perder información sobre las filas de ventas (`sales`) que no concuerdan. Por ejemplo, el ítem `2020-NAN-01` no está incluido en el resultado, porque no existe en `items_df`.

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

### Unión Externa: Ítems de Ambas DataFrames

Las uniones externas incluyen todas las filas de ambas dataframes. Si un ítem sólo aparece en una de las dos dataframes, entonces las columnas no concordantes tendrán valores `NaN`.

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

### Unión Interna: Ítems en Ambas `sales` e `items_df`

Una unión interna crea una nueva dataframe que contiene solamente las filas que existen en ambas dataframes (basado en la clave de unión).

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

### Unión Cruzada: El Producto Cartesiano de Ambas DataFrames

Una unión cruzada puede ser útil en algunos casos específicos (aunque no en este ejemplo). Produce el Producto Cartesiano de las dos dataframes, lo que significa que cada fila en `sales` es combinada con cada fila en `items_df`. Notá que `left_on` y `right_on` no son necesarias en esta operación.

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
