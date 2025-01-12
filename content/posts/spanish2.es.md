---
author: "Pavodive"
title: "Paquete spanish2 para R"
date: "2025-01-11"
description: "Paquete para convertir números a su equivalente en texto en Español con R"
tags: ["R"]
categories: ["R"]
series: ["How-to"]
aliases: ["paquete-spanish2"]
ShowToc: true
TocOpen: true
---

En documentos de tipo legal en español con frecuencia es necesario agregar el "valor en letras" de un número: $2400 (dos mil cuatrocientos pesos). Escribí el paquete `spanish2` para hacer exactamente esta tarea. Si quieres conocer más sobre este paquete, continúa la lectura. Si quieres instalar el paquete o ver su código, lo puedes hacer desde el repo en [github](https://github.com/pavodive/spanish2). La escritura de este paquete inspiró [este artículo]({{< ref "writing-r-package-emacs.es" >}} "Escribir un paquete de R con ESS") sobre cómo escribir un paquete de R con emacs y ESS (Emacs Speaks Statistics), tal vez le quieras dar una mirada.

<!--more-->

## ¿Por qué escribir el valor en letras?

No sé si es solamente una costumbre en mi país🇨🇴, pero en textos legales aquí se acostumbra escribir el valor en letras inmediatamente después de una cantidad. Por ejemplo:

> ...con un precio de $1.000.000 (un millón de pesos)...

o

> ...el lote tiene un área de 43 ha (cuarenta y tres hectáreas)...

Debido a unas colaboraciones que hago, me he enfrentado a esa situación muchas veces durante los últimos meses.


## ¿Qué alternativas tenía?

Hay un paquete en [CRAN](https://cran.r-project.org/web/packages/spanish/index.html) y en [github](https://github.com/rOpenSpain/spanish) llamado `spanish` que cumple casi con la funcionalidad que necesitaba. El paquete tiene una función, `spanish::to_words()` que convierte los números en texto. Pero esta función tenía algunas limitaciones:

### Primer inconveniente: Representaciones Incorrectas de Números

En ciertos casos, el paquete generaba representaciones de texto incorrectas para números. Por ejemplo:

```r{linenos=true}
spanish::to_words(40000000)
# "cuarenta millones mil "
```

Puede identificar algunos de estos casos y sugerir un ajuste al código, pero luego encontré nuevos casos de la peor forma posible: en un documento terminado y entregado a un cliente 😵.

### Segundo Inconveniente: Rango Limitado

El paquete también tiene una limitación importante: no puede procesar números mayores a 999,999,999.

## Creando una Solución: `spanish2`

Así que había que desarrollar una solución para el problema que tenía en frente, y **no hay nada mejor que estandarizar las soluciones que te sirven a tí**, es posible que alguien más las pueda utilizar: allí nace el paquete `spanish2`.

**Nota Importante**: Aunque el paquete `spanish`  no cumplió mis necesidades específicas para esta tarea, tiene unas funciones únicas y muy valiosas. Por ejemplo, `to_number()`, convierte valores en letras a sus equivalentes en números, y el paquete incluye funcionalidad relacionada con geolocalización en España. Si estas funcionalidades suentan interesante, ¡te invito a probar el paquete `spanish`!

## Estrategia de solución

Para convertir de números a letras, seguí el mismo principio que seguimos cuando leemos números en español:

- Agrupar los números de 3 en 3, de derecha a izquierda.
- Leer estos números de "centenas" (esta idea resultó clave, ya explico😉).
- Unir los diferentes bloques de "centenas" y finalmente
- Depurar el texto.

### Agrupar los números de 3 en 3 y leerlos en "centenas"

Lo primero es convertir el número en una cadena de texto, y luego dividirlo en grupos de 3 dígitos, para lo que utilizamos un regex (expresión regular):

```r
groups = regmatches(y, gregexpr(".{1,3}(?=(.{3})*$)", y, perl = TRUE))[[1]]
```

Una vez hemos dividido el número en grupos de 3 dígitos, pasamos cada uno de esos grupos a la función `convert_3_digits`. La función `convert_3_digits` "lee" centenas, decenas y unidades:

```r{linenos=true, hl_lines=[7,8]}
convert_3_digits = function(string_value){
  units = c("cero", "uno", "dos", "tres", "cuatro",
            "cinco", "seis", "siete", "ocho", "nueve")

  string_value = sprintf("%03d", as.integer(string_value))
  digits = strsplit(string_value, "")[[1]]
  a = sapply(digits, function(x) units[as.integer(x) + 1])
  raw_text = mapply(paste0, a, c("cientos", "diez y", ""))
  clean_text(paste(raw_text, collapse = " "))
}
```

Básicamente lo que hacemos con el `sapply` es convertir cada dígito a su equivalente texto: _234_ quedará convertido en `c("dos", "tres", "cuatro")`. Lo que hacemos con `mapply` es mapear este vector al vector de centenas, decenas y unidades, resultando en `dos cientos tres diez y cuatro`. La función `clean_text` convierte algunos casos atípicos, como "tres diez", que verdaderamente es "treinta".

### Mapear las grandes unidades

Hasta este punto tenemos un número convertido en texto en grupos de tres. Por ejemplo, el número 1 234 567 estaría convertido en `c("cero cientos cero diez y uno", "dos cientos tres diez y cuatro", "cinco cientos seis diez y siete")`. Pero eso no tiene mayor sentido, ¿cierto?

Es necesario mapear este vector a un vector que nos indique las grandes unidades: el segundo grupo de derecha a izquierda serán "miles", y el trecer grupo serán "millones". Después de usar `mapply`, el texto quedara convertido en "cero cientos cero diez y uno **millones** dos cientos tres diez y cuatro **mil** cinco cientos seis diez y siete". Esto se ve mal, pero en esencia está correcto.

### Limpiar el texto

En esta última parte utilizamos mucho la función `gsub` para reemplazar expresiones regulares en la cadena de texto. Esencialmente vamos a:

- Retirar lo que no sirve: "cero cientos" no debe aparecer, "cero diez y" tampoco.
- Ajustar algunas pronunciaciones especiales: "diez y cinco" es quince, "diez y tres" es trece, etc.

## Qué logramos

La función `spanish2::to_words()` convierte números hasta 1e22 o cadenas de texto con números de hasta 60 caracteres. ¡Esos son números muy grandes!

La conversión a texto sigue el estándar del idioma español que es usar la [escala numérica larga](https://es.wikipedia.org/wiki/Escalas_num%C3%A9ricas_larga_y_corta), y por lo tanto el número 1e9 (1,000,000,000) será convertido a "mil millones" (no a un billón, como puede ser la expectativa de personas angloparlantes que están acostumbradas a la escala corta).

La conversión sigue la forma más estándar **y sencilla** de expresar números, a pesar de que puedan existir otras posibles representaciones:

| Número | Estilo 1        | Otros Estilos             | resultado de spanish2 |
|:------:|:---------------:|:-------------------------:|-----------------------|
| 77     | Setenta y siete |                           | setenta y siete       |
| 16     | Dieciséis       | Diez y seis               | diez y seis           |
| 27     | Veintisiete     | Veinte y siete            | veinte y siete        |
| 1100   | Mil cien        | Mil ciento / mil y ciento | mil cien              |

Espero que este paquete te sea de utilidad, si es así ¿qué tal darle una estrellita en github⭐?

[Dale ⭐ en github](https://github.com/pavodive/spanish2)
