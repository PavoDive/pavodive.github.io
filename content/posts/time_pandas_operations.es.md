---
author: "Pavodive"
title: "Operaciones Básicas con Fechas en Pandas"
date: "2025-01-18"
description: "Conversión de fechas con pandas, filtrado con fechas con pandas"
tags: ["python", "pandas"]
categories: ["python"]
series: ["How-to"]
aliases: ["fechas-basicas-pandas"]
ShowToc: true
TocOpen: true
---

Las fechas y los objetos de fecha-hora son algunos de los tipos de datos más exigentes con los que podemos trabajar. Pueden venir en una variedad de formatos (¡viva la [ISO 8601](https:/es.wikipedia.org/wiki/ISO_8601)!), pueden tener diferentes zonas horarias y como resultado introducir una complejidad considerable.

<!--more-->

Además de las variaciones en formato, estas discrepancias pueden causar problemas grandes:

![](https://imgs.xkcd.com/comics/formatting_meeting.png)

Más aún, las zonas horarias añaden otra capa de confusión. Veamos como pandas 🐼 maneja esto.

## Nuestra DataFrame Base

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

## Convirtiendo Texto (Strings) a Fechas en Pandas

Vamos a ver como pandas interpreta los tipos de dato en nuestra dataframe:

```python
print(items_df.dtypes)

# Output:
# name            object
# price            int64
# date_created    object
```

Como podemos ver, la columna `date_created` es tratada como un `object` (probablemente porque contiene cadenas de texto o _strings_). Para poderla maejar como fecha-hora, necesitamos convertirla:

```python
items_df["date_created"] = pd.to_datetime(items_df["date_created"])
print(items_df.dtypes)

# Output:
# name            object
# price            int64
# date_created    datetime64[ns]
```

Ahora la columna `date_created` es correctamente procesada como `datetime64[ns]`.

## Filtrando una DataFrame con Una Fecha

La forma en que pandas maneja las fechas y horas es todavía un poco misteriosa para mí. Yo entiendo que las fechas y horas son inherentemente complejas, pero siento que el comportamiento de pandas algunas veces es confuso. Por ejemplo, considerá [esta pregunta en Stack Overflow](https://stackoverflow.com/questions/13703720/converting-between-datetime-timestamp-and-datetime64) y la respuesta detallada de Quant. A pesar de que el diagrama no está perfectamente actualizado (a fecha de hoy, 2025-01-19), es todavía bastante relevante y demuestra la complejidad involucrada.

Aquí está uno de esos comportamientos sorprendentes:

A pesar de que nuestra columna `date_created` muestra ser de tipo `datetime64[ns]`, cuando accedemos a sus elementos individuales se revela algo un poco distinto:

```python
print(type(items_df["date_created"].iloc[1]))
# Output: <class 'pandas._libs.tslibs.timestamps.Timestamp'>
```

Cuando tratamos de filtrar esta columna utilizando una simple cadena de texto con una fecha, pandas automáticamente coerce el texto a fecha, y la operación funciona como se espera:

```python
start_date_1 = "2000-01-01"
filtered_items_1 = items_df[items_df["date_created"] >= start_date_1]
print(filtered_items_1)
```

Sin embargo, si la cadena de texto incluye una hora y una zona horaria, pandas no hace la coerción automática, y la operación falla:

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

Pandas, tal vez obrando con cautela, evita la  coercíon en este caso, dejándonos la responsabilidad a nosotros. Corrijamos esto manualmente convirtiendo la cadena de texto en un _timestamp_ utilizando el mismo método que usamos antes:

```python
start_date_2 = pd.to_datetime(start_date_2)
print(type(start_date_2) == type(items_df["date_created"].iloc[1]))
# Output: True 
# Both are <class 'pandas._libs.tslibs.timestamps.Timestamp'>
```

En este punto los dos valores aparentemente son del mismo tipo, pero aún no pueden ser comparados:

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

El error ocurre porque una valor es **timezone-naive** (sin información sobre la zona horaria), y el otro es **timezone-aware** (con información de zona horaria). Para proceder, debemos convertir uno de ellos a que concuerde con el otro.

### Solución 1: Convertir `start_date_2` a Timezone-Naive

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

### Solución 2: Convertir `items_df["date_created"]` a Timezone-Aware

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

Un elemento clave aquí es el accesor `.dt`, que sirve como un gateway para plicar operaciones de fecha-hora en Series de pandas. De acuerdo con la [documentación](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.dt.html), el accesor `.dt` _"devuelve una Serie indexada como la Serie original. Eleva un TypeError si la Serie no contiene valores tipo fecha-hora"_. Esto es esencial porque `tz_localize` opera en objetos _datetime_, no directamente en Series (¡gracias [wordsforthewise](https://stackoverflow.com/questions/22800079/converting-time-zone-pandas-dataframe) por el tip!).

Las fechas y horas tienen sus trucos, pero pandas ofrece herramientas poderosas para manejarlas. Con algo de cuidado y estando alerta a los vericuetos de las fechas-hora, vamos a dominar estos retos ¡muy pronto! 🐼
