---
author: "Pavodive"
title: "Automatizando el Renderizado de Rmarkdown y Uniendo PDFs con Bash"
date: "2025-02-06"
description: "Usé bash para automatizar el renderizado de cientos de archivos Rmarkdown y para unir los PDFs resultantes"
tags: ["shell", "bash"]
categories: ["shell"]
series: ["How-to"]
aliases: ["bash-scripts-rmarkdown"]
ShowToc: true
TocOpen: true
---

Yo trabajo con muchos archivos Rmarkdown estructurados dentro de una jerarquía de directorios. Yo necesitaba renderizar esos archivos a PDF y luego utilizar Ghostscript para unirlos. Este artículo explica dos pequeños scripts de shell que usé para la tarea.

<!--more-->

## El Problema

Para el proyecto de un cliente yo necesito producir varios documentos PDF que consisten de:

- Una carta de presentación (cover)
- Un reporte

Por razones que no son relevantes para este artículo, la carta de presentación y el reporte utilizan plantillas de renderizado diferentes, así que no se pueden unir antes de renderizar. Los dos documentos son escritos en Rmarkdown (archivos `.Rmd`) que renderizan directamente a PDF.

Cada proyecto involucra _cientos_ de pares cover-reporte, lo que convierte en impráctico el proceso de renderizarlos manualmente. La estructura de directorios sigue este patrón:

```
📂--client-root
    📂--project-1
    |  📂--report
    |  |  |--cover.Rmd
    |  |  |--report_project-1.Rmd
    |  📂--data
    📂--project-2
    |  📂--report
    |  |  |--cover.Rmd
    |  |  |--report_project-2.Rmd
    |  📂--data
```

Por supuesto, en la realidad mis directorios no se llaman "project-n"; ellos tienen nombres reales que tienen sentido.

> Yo nunca, **nunca**, uso espacios o caracteres no-ASCII en los nombres de ningún directorio o nombre de archivo.

## La Solución

Yo usé un comando de una sola línea para renderizar todos los `Rmd` a PDF:

```sh
find client-root -type f -name "*.Rmd" | xargs -I{} Rscript -e 'rmarkdown::render("{}")'
```

### Como Funciona

- `find client-root -type f -name "*.Rmd"` es un comando estándar de `find` que:
  - Busca dentro de `client-root`
  - Por archivos (`-type f`) (la f es de _file_: archivo)
  - Cuyos nombres terminen en `.Rmd` (`-name "*.Rmd"`)
  - El resultado que produce es una lista de _paths_, por ejemplo, `./client-root/project-1/report/cover.Rmd`.

- El `|` (pipe) envía esta lista al siguiente comando.

- `xargs -I{} Rscript -e 'rmarkdown::render("{}")'` procesa cada archivo:
  - `xargs` construye y ejecuta comandos para cada archivo encontrado.
  - `-I{}` le dice a `xargs` que reemplace `{}` con cada nombre de archivo.
  - `Rscript -e` ejecuta una expresión de R (`-e` significa ejecución en línea).
  - `rmarkdown::render("{}")` llama la función de R que procesa cada archivo dinámicamente, reemplazando `{}` con el nombre de cada archivo.

Después de correr esto, la estructura de directorios ahora contiene los PDFs correspondientes:

```
📂--client-root
    📂--project-1
    |  📂--report
    |  |  |--cover.Rmd
    |  |  |--cover.pdf
    |  |  |--report_project-1.Rmd
    |  |  L--report_project-1.pdf
    |  📂--data
    📂--project-2
    |  📂--report
    |  |  |--cover.Rmd
    |  |  |--cover.pdf
    |  |  |--report_project-2.Rmd
    |  |  L--report_project-2.pdf
    |  📂--data
```

## Un Nuevo Problema
Ahora necesito combinar la carta de presentación (cover) en PDF con el reporte en PDF para cada proyecto.

Para un único proyecto, podría haberlo hecho manualmente utilizando Ghostscript (`gs`):

```sh
gs -dBATCH -dNOPAUSE -sDEVICE=pdfwrite \
   -sOutputFile=merged_report_project-1.pdf \
   cover.pdf report_project-1.pdf
```

Pero como dije, tengo muchos proyectos, así que necesitaba automatizar el proceso utilizando Bash.

Adicionalmente, tenía que seguir esta convención para nombrar los archivos resultantes:

El nombre de archivo resultante debe comenzar con `"merged_"`, seguido por el nombre de archivo del reporte, por ejemplo:

```
merged_report_project-1.pdf
```

## Uniendo los PDFs

Para unir los PDFs, my plan era:

1. Encontrar todos los directorios `report` en todos los proyectos.
2. Extraer las rutas para los archivos PDF de la carta de presentación y el reporte.
3. Construir dinámicamente el nombre de archivo resultado.
4. Utilizar Ghostscript para unir los archivos.

Aquí está el script:

```sh
find client-root -type d -name "report" | \
    while read -r dir; do
        cover_pdf="$dir/cover.pdf"
        report_pdf=("$dir/report_"*.pdf)
        output_pdf="$dir/merged_$(basename "${report_pdf[0]}")"
        gs -dBATCH -dNOPAUSE -sDEVICE=pdfwrite \
           -sOutputFile="$output_pdf" \
           "$cover_pdf" \
           "${report_pdf[0]}"
    done
```

### Entendiendo el Script

- **Encontrando los directorios**
  ```sh
  find client-root -type d -name "report"
  ```
  - Busca bajo `client-root`
  - Encuentra solamente **directorios** (`-type d`) llamados `"report"`.
  - Los resultados son pasados (piped) al siguiente comando.

- **Procesando cada directorio**
  ```sh
  while read -r dir; do ... done
  ```
  - Itera sobre cada directorio encontrado.
  - `read -r dir` asigna cada ruta de directorio a `dir`.
  - La bandera `-r` asegura que la ruta es leída literalmente, evitando secuencias de escape no previstas.

- **Definiendo las Rutas de Archivo**
  ```sh
  cover_pdf="$dir/cover.pdf"
  ```
  - Construye la ruta para el PDF de la carta de presentación.
  - Las comillas aseguran el correcto manejo de los espacios en los nombres de directorio, si existen (aún a pesar de que yo los evito).

  ```sh
  report_pdf=("$dir/report_"*.pdf)
  ```
  - Utiliza un comodín (`report_*.pdf`) para encontrar el archivo de reporte.
  - Los parentesis **crean un array**, permitiendo que hayan múltiples resultados (aunque sólo esperamos uno).

- **Construyendo el Nombre Combinado de Archivo**
  ```sh
  output_pdf="$dir/merged_$(basename "${report_pdf[0]}")"
  ```
  - `${report_pdf[0]}` selecciona el primer resultado (y esperamos que el único).
  - `basename` elimina al ruta del directorio, manteniendo únicamente el nombre de archivo.
  - `$( ... )` ejecuta una **sustitución de comando**, insertando el resultado dinámicamente.
  - `"merged_"` es antepuesto para crear el nombre final del archivo combinado.

- **Uniendo con Ghostscript**
  ```sh
  gs -dBATCH -dNOPAUSE -sDEVICE=pdfwrite \
     -sOutputFile="$output_pdf" \
     "$cover_pdf" \
     "${report_pdf[0]}"
  ```
  - Une los PDFs de la carta de presentación y del reporte, guardándo el resultado como `merged_report_project-N.pdf`.
  - Si tenés curiosidad sobre las banderas de `gs`, chequeálas con `man gs`.

¡Y eso es todo! Ahora todos mis archivos `merged_report_project-x.pdf` se generan automáticamente.

**Bash me ha ahorrado un montón de tiempo**, que he utilizado para escribir este artículo. ¡Ahora de nuevo a trabajar! 😇
