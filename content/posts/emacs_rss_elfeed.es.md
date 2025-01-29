---
author: "Pavodive"
title: "RSS con elfeed en emacs"
date: "2025-01-29"
description: "Así configuré elfeed para tener un compendio de los blogs y sitios que me gustan"
tags: ["blog", "emacs"]
categories: ["emacs"]
series: ["How-to"]
aliases: ["rss-elfeed-emacs"]
ShowToc: true
TocOpen: true
---

Así de fácil fue configurar elfeed en emacs para tener un compendio de todos los blogs y sitios que me gustan de emacs.
En este artículo comparto la forma (y algunos RSS) de configurar el paquete elfeed para tener los RSS de los blogs y canales de video que me gusta leer o ver. Esto es importante para mi, porque siempre gasto mucho tiempo filtrando cosas que no me interesa ver en los foros como [lemmy.ml](https://lemmy.ml) o [reddit.com](https://reddit.com). Es también una forma de evitar los anuncios y poder ir directamente al contenido.

<!--more-->

## Elfeed

Elfeed es un paquete para emacs que sirve exactamente para eso: agregar _feeds_. La mayoría de sitios tienen feeds en los que "anuncian" los nuevos artículos que se han añadido. Esa información es procesada por elfeed y presentada como una simple tabla ordenada por fecha, en la que podés ver los títulos de las páginas añadidas, y algunos otros detalles.

Para instalar elfeed en tu emacs, simplemente (si tenés melpa o elpa) hacés `M-x package-install RET elfeed RET`, y luego añadí esto a tu `.emacs`:

```lisp
(global-set-key (kbd "C-x w") 'elfeed)
```

(Podés usar `C-x w` u otra de tu preferencia, si tenés ocupada la combinación `C-x w`).

Ahora simplemente invocando `M-x elfeed` o `C-x w` vas a tener una impresionante pantalla vacía, porque aún no añadimos los feeds 📝.

## Añadiendo los Feeds

Para añadir los feeds a elfeed, simplemente se utiliza código semejante a:

```lisp
(setq elfeed-feeds
      '(("https://url.del.feed/atom.xml" nombre-de-tag)))
```

`nombre-del-tag` es el nombre (o nombres) que le querás dar a la etiqueta de esa fuente espeífica. Mirá estos ejemplos:

```lisp
(setq elfeed-feeds
      '(("https://planet.emacslife.com/atom.xml" emacs)
        ("https://xenodium.com/rss" emacs)
        ("https://lemmy.ml/feeds/c/emacs.xml?sort=Active" emacs)
        ("https://programming.dev/feeds/c/django.xml?sort=Active" django)))
```

## Disfrutando el Resultado

Después de haber cargado estos feeds, ahora podemos abrir nuevamente elfeed y recargar con `G` (`elfeed-search-fetch`).

Debe resultar algo similar a esto:

{{< figure src="/images/scshot_elfeed.png" attr="Captura de pantalla de elfeed" align=center link="/images/scshot_elfeed.png" target="_blank" >}}

Ahora podés navegar, hacer búsquedas, y por supuesto leer los artículos que despierten tu interés.

La navegación en esta pantalla es posible con los atajos que tengas definidos para siguiente y anterior línea (`C-n` y `C-p` en emacs vainilla) con `n` (`next-line`) y `p` (`previous-line`). Es posible marcar una entrada que no te interesa leer como "leída" con `r` (`elfeed-search-untag-all-unread`), y obviamente entrar a leerla con `RET` (`elfeed-search-show-entry`).

La funcionalidad de búsqueda es posible con `s` (`elfeed-search-live-filter`). Es una búsqueda incremental, y es posible filtrar por fechas y por etiquetas:

- Escribir "lis" mostrará resultados que contengan:
    - La palabra "list",
    - La palabra "lisp",
    - La palabra "elisp",
    - La palabra "playlist".
    
{{< figure src="/images/scshot_elfeed_search.png" attr="Búsqueda en elfeed" align=center link="/images/scshot_elfeed_search.png" target="_blank" >}}

Hay otros varios atajos de teclado, aseguráte de revisarlos con `C-h m` o simplemente `h` (`describe-mode`).

## Leyendo una entrada

Para leer una entrada, simplemente la seleccionás, y `RET`. Listo. En la mayoría de los casos la entrada va a estar ahí, visible en emacs y la podés leer. En otros casos es posible que prefirás verla en el browser que tenés instalado. Para hacer eso, simplemente `b` (`elfeed-search-browse-url`) abre la entrada en tu navegador.

{{< figure src="/images/scshot_elfeed_post.png" attr="Viendo una entrada en emacs" align=center link="/images/scshot_elfeed_post.png" target="_blank" >}}

Así se ve el post en el navegador:

{{< figure src="/images/scshot_elfeed_browser.png" attr="Viendo una entrada en el navegador firefox" align=center link="/images/scshot_elfeed_browser.png" target="_blank" >}}

Un comportamiento interesante de `b` es que abre la entrada en una pestaña del navegador y **simultáneamente** avanza a la siguiente línea. Cuando terminés de hacer `b` en varias de las entradas que te interesaron, vas a tener varias pestañas abiertas en el navegador.

Aunque la verdad, yo prefiero leer las entradas en el propio emacs 😉.

{{< figure src="/images/winking_gnu.png" attr="" align=center link="/images/winking_gnu.png" target="_blank" >}}




Te animo a explorar esta sencilla forma de mantenerte al tanto de las cosas que te interesan, sin exponerte al contenido que no te interesa y no querés. Para saber más sobre elfeed, por favor vistiá su [página de github](https://github.com/skeeto/elfeed).
