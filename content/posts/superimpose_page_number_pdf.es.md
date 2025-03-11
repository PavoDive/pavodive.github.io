---
author: "Pavodive"
title: "Añadiendo Números de Página a Documento PDF"
date: "2025-03-10"
description: "Añadir números de página a un pdf con LaTeX"
tags: ["latex"]
categories: ["latex"]
series: ["How-to"]
aliases: ["superimpose-page-number"]
ShowToc: true
TocOpen: true
---

Recientemente me enfrenté al siguiente problema: Tenía un documento PDF creado uniendo varios otros documentos, varios de ellos de varias páginas. Yo necesitaba mostrar los números de página. ¡**LaTeX** y **pdftk** al rescate!

<!--more-->

# El Problema

Yo necesitaba generar un PDF uniendo varios documentos de diferentes fuentes, lo que resultó en un archivo de 57 páginas. Después, necesitaba mostrar el número de página como "Folio x/57", donde "x" representaba el número de la página actual.

# La Solución

Mi plan para resolver este problema es de dos pasos:

1. **Generar un nuevo PDF sólo con los encabezados** ("Folio x/57") en cada página.
2. **Combinar** este documento con el original, de tal forma que los encabezados quedaran sobrepuestos.

## Generando un PDF Sólo con Números de Página

Yo necesitaba crear **57 páginas mayormente vacías** con "Folio x/57" en la esquina superior derecha. El número de página debía actualizarse dinámicamente para cada página. Aunque hay varias formas de lograr esto, generar un PDF era la solución que más sentido tenía.

Hay varias herramientas con las que se puede crear un documento así, pero yo escogí `LaTeX` porque ofrece la solución más simple y personalizable. La personalización es crucial, particularmente por el ajuste en los márgenes, ya que en el documento original todos los márgenes eran inconsistentes. Los paquetes `geometry` y `fancyhdr` de `LaTeX` me daban la flexibilidad que necesitaba.

Aquí está el código que usé en el archivo `pagination.tex`:

```latex
\documentclass[12pt]{article}
\usepackage[paperwidth=8.5in,
  paperheight=11in,
  top=0.7in,
  right=0.5in,
  left=0.5in,
  bottom=0.5in]{geometry}
\usepackage{fancyhdr}
\usepackage{pgffor}
\pagestyle{fancy}
\fancyhf{}
\rhead{Folio \thepage/57}

\begin{document}
  \setcounter{page}{1}
    \foreach \n in {1,...,57} {
      \null\vfill\eject
    }
\end{document}
```

Este _script_ establece el tamaño de papel y márgenes del documento, define el formato del encabezado y finalmente genera las páginas usando un _loop_, insertando un salto de página en cada iteración.

Si te estás preguntando por qué utilicé `\vfill\eject` en lugar de `\newpage`, te recomiendo que chequeés [esta pregunta en tex.stackexchange.com](https://tex.stackexchange.com/questions/208698/difference-between-newpage-and-vfill-eject) (está en Inglés, _sorry_ 🇬🇧).

Después de compilar el archivo de LaTeX, se obtiene `pagination.pdf`, un hermoso documento en blanco con los números de página presentados en los lugares apropiados.

## Combinando el Nuevo Documento con el PDF Original

El paso final era combinar los dos PDFs. No sólo **unirlos**, sino realmente **sobreponerlos**, de tal forma que los encabezados de mi nuevo documento aparecieran sobre el contenido original.

Para esto utilicé `pdftk` un fantástica herramienta para manipular PDFs. La siguiente línea de código hizo la magia:

```shell
pdftk original.pdf multistamp pagination.pdf output numbered.pdf
```

- `original.pdf` es el documento principal.
- `pagination.pdf` contenía los encabezados.
- La opción `multistamp` le dice a **pdftk** que sobreponga cada página en `pagination.pdf` sobre la página correspondiente en `original.pdf`.
- La opción `output` genera el nuevo PDF numerado como `numbered.pdf`.

Y ¡ya está! Logré resolver un problema tedioso con sólo unas pocas líneas de código. Ahora estoy compartiendo esto con vos –y con _mi yo futuro_ 😉.
