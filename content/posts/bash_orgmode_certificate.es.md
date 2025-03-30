---
author: "Pavodive"
title: "Un Script en Bash Para Editar y Exportar Desde una Plantilla en Org-mode"
date: "2025-03-30"
description: "Yo tengo la plantilla de un documento en org-mode. Yo debo editar una información en ese archivo, y exportarla a texto y luego a PDF. Yo lo logro con un script de bash."
tags: ["shell", "bash"]
categories: ["shell"]
series: ["How-to"]
aliases: ["bash-org-mode-document-es"]
ShowToc: true
TocOpen: true
---

Yo tengo la plantilla de un documento escrita como un archivo de Org. Yo tengo que editar algunos campos en ese archivo para poder generar diferentes versiones de ese documento para diferentes personas. Yo lo exporto como un archivo de texto para dar la sensación de un documento escrito con un software antiguo e impreso con una impresora de matriz de puntos. Finalmente convierto el documento producido a PDF.

<!--more-->

## El Problema

Cada tanto yo tengo que producir un documento que tiene que tener la apariencia de software antiguo y de haber sido impreso en una impresora de matriz de puntos. Ese documento tiene que tener algunos datos fijos y algunos datos dinámicos, que están relacionados con el caso particular que en ese momento tenga a la mano. Sin ninguna automatización, el proceso es como este:

1. Abrir el archivo de Org y editar los campos. Tratar **de no olvidar** editar ninguno de los campos. _No que me haya pasado a mi, claro 🙄_.
2. Exportar a texto con `C-c C-e t a`.
3. Convertir el archivo de texto a pdf.

Yo me preguntaba si podría hacer esto sin necesidad de abrir el archivo de Org, y especialmente sin **olvidarme de alguno de los campos**. Un comando, escribir las sustituciones ¡y listo! Eso sería ideal.

## La Solución

> Esta solución es uno de esos casos en los que el formato de texto plano es el Rey.

La solución que pensé es un script de bash que yo pueda ejecutar desde el terminal, y está pensada en cuatro partes:

- Pedir al usuario los valores para los campos dinámicos utilizando `read` de bash.
- Sustituir los campos con los valores ingresados por el usuario utilizando `sed`.
- Exportar a un archivo temporal de texto utilizando `emacs` desde la línea de comando, y
- Convertir el archivo de texto a PDF utilizando `cups`.

### Preguntar al Usuario por los Campos Dinámicos

```shell
#!/usr/bin/bash

read -p "Por favor ingresa el año: " year
read -p "Por favor ingresa el nombre de la persona: " nombre_sujeto
read -p "Por favor ingresa el texto para otro campo: " other_field
# Algunos otros comandos read similares a los de arriba
```

### Sustituir Campos en el Archivo de Org
El plan en este paso es sustituir las ocurrencias de "campo" en el archivo Org con los valores proporcionados por el usuario para "campo" (`$campo`), y hacerlo con `sed`. El paso después de este será ejecutar el comando org-export en el texto producido por la sustitución. Aquí debemos tener en cuenta que Emacs no lee desde standard input, sino desde un archivo. Así que necesitamos hacer las sustituciones con `sed`, _exportarlas a un archivo temporal_, y luego ejecutar el comando org export en ese archivo.

Miremos las dos primeras partes:

```shell
# Crear el archivo temporal
tempfile=$(mktemp /tmp/docXXXXXXXXXXXXXXX.org)

# Hacer las sustituciones con sed
sed -e "s/year/$year/g" -e "s/nombre_sujeto/$nombre_sujeto/g" -e "s/other_field/$other_field/g" /full-path-to/template_document.org > "$tempfile"
```

Algunos puntos interesantes aquí:

- `mktemp`: Es muy útil para casos como este, en el que necesitamos una salida intermedia. ¡Recordá borrarlo después de trabajar con él!
- El uso de las comillas dobles ["] en `sed`. Si usás comillas sencillas, vas a reemplazar cualquier ocurrencia de `year` con el texto literal `$year` **y no con el texto que esperás**, digamos `2025`.
- Direccionamos los resultados de la sustitución al archivo temporal.

### Ejecutar el Comando Org Export en el Archivo Temporal
Emacs nos ofrece una característica muy interesante, que es permitirnos ejecutar una función sobre el archivo de entrada. Usaremos esa característica para ejecutar el comando de Org export sobre nuestro archivo temporal.

```shell
emacs "$tempfile" --batch \
      -f org-ascii-export-to-ascii \
      --kill
```

La opción `-f` es la que te permite definir la función que querés ejecutar (`org-ascii-export-to-ascii` en nuestro caso).

La opción `--batch` impide que Emacs tenga un display interactivo, y la opción `--kill` termina sin pedir confirmación (podés ver `emacs --help` o `man emacs`).

Ahora tenemos que hacer un poco de limpieza antes de nuestro paso siguiente (y final):

```shell
# Crear el nombre del archivo de salida
output_name="document_${nombre_sujeto// /_}_$year.txt"

# Renombrar el archivo producido por emacs con el nuevo nombre
mv "${tempfile%.org}.txt" "./$output_name"

# Borrar el archivo temporal producido por sed
rm "$tempfile"
```

- La parte `${nombre_sujeto// /_}` toma el valor en `nombre_sujeto` y reemplaza cualquier espacio con barras al piso [_]. Por tanto, el valor `John Smith` producirá `John_Smith`.
- La parte `${tempfile%.org}` se deshace de la parte `.org`, simplemente dejando el nombre del archivo (sin la extensión), adicionando luego `.txt`. Entonces digamos que `tempfile` era `docAFGERWESF42424.org`, el nombre del archivo que estamos renombrando es `docAFGERWESF42424.txt`, que es, de hecho, el archivo producido por el comando de Org export. No queremos mantener ese nombre feo, y por eso es que lo estamos renombrando a algo más amigable.
- Finalmente borramos el archivo temporal para no dejar rastros 🥷🏽.

> "Toma únicamente fotos y deja únicamente huellas."

### Convertir el Archivo de Texto a PDF

Como mencioné antes, uno de los requerimientos de este proyecto era producir un documento que tuviera la apariencia de haber sido impreso en una impresora de matriz de puntos. Exportarlo directamente desde Org a PDF no hubiera cumplido ese requerimiento sin trabaje en plantillas de LaTeX.

Así que la idea es que retengamos tanto como sea posible del formato y el aspecto del texto plano, pero poder exportarlo a un archivo PDF. Aunque hay varias opciones para hacerlo, `cups` es, en mi opinión, la que produce el resultado que yo quería. También es un one-liner muy simple:

```shell
cupsfilter "$output_name" > "${output_name%.txt}.pdf"
```

Y ya estuvo, produje un documento en PDF con la apariencia de haber sido impreso en matriz de puntos, a partir de una plantilla en Org, sin siquiera haber abierto Emacs.
