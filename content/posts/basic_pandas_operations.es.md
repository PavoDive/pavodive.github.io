---
author: "Pavodive"
title: "Operaciones Básicas con Pandas"
date: "2025-01-18"
description: "Remover columnas con pandas, reordenar columnas con pandas y cambiar el nombre de columnas con pandas"
tags: ["python", "pandas"]
categories: ["python"]
series: ["How-to"]
aliases: ["operaciones-basicas-pandas"]
ShowToc: true
TocOpen: true
---

Hay algunas operaciones con pandas que frecuentemente olvido. Este sitio existe exactamente para eso: para recordarme como llevar a cabo esas operaciones. En este artículo cubriré algunas operaciones muy básicas que frecuentemente olvido, incluyendo la remoción de columnas, el reordenamiento de columnas y el cambio de nombre de columnas.

<!--more-->

## Pandas

No gastaré tiempo explicando pandas 🐼, la poderosa librería de Python para procesamiento de datos. Pandas puede manejar varias operaciones complejas con datos y es una herramienta que debe tener cualquier persona que trabaje con datos.

Al momento de escribir esto, soy mucho más proficiente con R que con pandas, lo que podría explicar porqué olvido repetidamente estas sencillas operaciones.

## Nuestra DataFrame Base

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

Nuestra dataframe base es una tabla simple que contiene datos variados sobre algunos ítems. Incluso tiene una columna innecesaria, que vamos a eliminar prontamente.

## Removiendo o Eliminando Columnas

Vamos a librarnos de la columna inútil. El parámetro `inplace=True` se explica a sí mismo: remueve la columna en la misma dataframe, en lugar de devolvernos un objeto nuevo. De esta manera, la dataframe original es directamente actualizada.

```python
items_df.drop(columns=["useless_column"], inplace=True)
```

## Reordenamiento de Columnas

Algunas veces necesito exportar dataframes a tablas o archivos CSV en los que el orden de las columnas es importante para los usuarios finales. Reordenar las columnas es tan simple como pasar una lista de nombres de columnas en el orden deseado:

```python
name_order = ["id", "date_created", "name", "color", "price"]
items_df = items_df[name_order]
```

## Cambiando los Nombres de Columnas

Otra operación que frecuentemente necesito hacer es renombrar columnas. Para cambiar el nombre de una columna, hay que pasar un diccionario con el formato `{"nombre_viejo": "nombre_nuevo"}` al método `rename`.

```python
items_df.rename(columns={"id": "sku", "color": "Color"}, inplace=True)
```
