---
author: "Pavodive"
title: "Escribiendo un Paquete para R con Emacs"
date: "2025-01-11"
description: "Guía paso a paso para escribir paquetes de R con emacs"
tags: ["R", "emacs", "ESS"]
categories: ["R", "emacs"]
series: ["How-to"]
aliases: ["escribir-paquete-r-emacs"]
ShowToc: true
TocOpen: true
---

Este artículo ofrece una guía paso a paso para escribir paquetes de R utilizando el modo ESS de emacs (Emacs Speaks Statistics). Al finalizarla, vos habrás creado un paquete sencillo llamado `dummyaddition`, que puede hacer la sencilla tarea de sumar dos números o concatenar dos cadenas de texto.

<!--more-->

## ¿Por Qué Emacs?

Emacs es una de esas piezas de software que ha pasado la prueba del tiempo. Aunque algunos pueden argumentar que la edad es una desventaja, yo pienso que el hecho de que emacs permanezca relevante después de más de 40 años es una prueba irrefutable de su diseño robusto y de su adaptabilidad a las necesidades modernas.

Emacs es mucho más que solo un editor de texto o un IDE —es una herramienta poderosa que me ha permitido desarrollar un flujo de trabajo eficiente.

Aquí les presento algunas de las razones por las que prefiero emacs:

### Eficiencia Centrada en Teclado
Emacs está basado en la línea de comando, lo que significa que casi toda acción posible puede ser ejecutada a través del teclado. Yo rara vez utilizo el mouse durante mi trabajo. Depender del mouse pude causar dolor o molestia en manos y brazos, y desperdicia tiempo en movimientos innecesarios.

### Expansibilidad y Personalización
Emacs es increíblemente expansible y altamente personalizable. Ya sea que prefirás mantenerlo simple o convertirlo en una máquina compleja y poderosa, podés ajustarlo a tus necesidades y para satisfacer tus deseos específicos.

### Fácil de Aprender
A pesar de que algunas personas dicen que la curva de aprendizaje de emacs es empinada, yo opino distinto. Emacs sin modificaciones (o vainilla, como se le llama en inglés) es muy fácil de usar y viene con una excelente documentación. Podés comenzar utilizando el mouse mientras te familiarizás con los atajos de teclado. No será una diferencia muy grande con lo que seguramente estás usando ahora. Con el paso del tiempo, irás aprendiendo a ampliar y personalizar emacs para que sea más eficiente en tu flujo de trabajo.

### Software Libre y de Código Abierto
Emacs es libre y gratuito. Podés instalarlo en cualquier sistema operativo sin costo. No sólo _no tendrás que pagar_, si no que **no te van a quitar nada**: ni tus datos, ni tus interacciones con el software, ni tu privacidad.

Que sea de código abierto significa que podés modificar emacs e incluso compartir tus modificaciones con otros sin que tengás que preocuparte por asuntos legales. Emacs representa la verdadera libertad de software.

## ¿Por Qué un Paquete?
Uno de los objetivos fundamentales del desarrollo de software es la **automatización**. Tareas que hacemos frecuentemente deberían hacerse más fáciles y rápidas de ejecutar. Es un escenario común: escribís una función una vez, y luego la necesitás otra y otra vez...

En lugar de reescribir la misma función múltiples veces (créeme, lo he hecho y no es divertido), o desperdiciar tiempo buscando por donde la escribiste la última vez, es mejor consolidar esas funciones en un paquete (o librería). Un paquete te permite cargar y reutilizar sin mayor esfuerzo esas funciones cuando sea que las necesités.

Crear un paquete también significa que podés compartir tus funciones con otros que podrían estar enfrentando el mismo problema que ya resolviste. Escribiendo paquetes estás contribuyendo a un ambiente colaborativo. Después de todo, estoy seguro que en algún momento has dependido fuertemente de paquetes de R escritos por otros, así que ¿por qué no devolver un poco de ese beneficio?

## Resultado Final
Siguiendo esta guía paso a paso, crearás un nuevo paquete llamado `dummyaddition`. Este paquete, a pesar de ser muy básico, demostrará los pasos esenciales involucrados en la creación de un paquete para R con emacs y ESS. Seguramente albergarás tu versión en tu repositorio de GitHub, pero si querés ver la mía como referencia, aquí está el enlace: [`dummyaddition`](https://github.com/pavodive/dummyaddition).

## Paso a Paso
### Magit para Control de Versiones
#### Crear un Repositorio en GitHub
El primer paso es crear un nuevo repositorio en tu cuenta de GitHub. Yo he llamado al mío `dummyaddition`, pero podés escoger un nombre que tenga sentido para tu proyecto. Aunque llenar el campo **descripción** es opcional, de veras te recomiendo que lo hagas —es muy útil tener una descripción clara de cada repositorio.

Tené en mente que las convenciones que CRAN establece para nombrar paquetes sólo permiten caracteres ASCII y números. Evitá usar caracteres especiales como guiones, subrayados o puntos en el nombre.

También tendrás la opción de hacer tu repositorio **público** o **privado**. Si estás planeando compartir tu código (a lo que te exhorto), hacé que el repositorio sea público.

Antes de crear el repositorio, configurá los siguientes ajustes menores pero muy importantes:

- **Inicializá con un `README.md`**: Marcá esta casilla. Usaremos este archivo posteriormente.
- **Añadí un archivo `.gitignore`**: Seleccioná la plantilla para R. Esto nos asegura que Git ignora archivos innecesarios para nuestro proyecto, tales como `.history`, que no agregan ningún valor.
- **Escogé una licencia**: Github nos ofrece varias opciones de licenciamiento de nuestro software. Dos muy comunes son la _MIT License_ y la _GNU General Public License v3.0_. Aseguráte de leer sobre sus diferencias, ya que ellas pueden afectar cómo puede usarse tu software. Para este tutorial, he escogido la licencia _GNU General Public License v3.0_.
    - Adicionalmente, chequeá las licencias de los paquetes de R que planeás utilizar en tu código. Algunas licencias requerirán que adoptés licencias compatibles o menos restrictivas en tu paquete.
    
Una vez has configurado estos ajustes, hacé click en **Create Repository**.

#### Magit
**Magit** es un paquete de emacs que actúa como un híbrido entre un cliente gráfico de Git y la interfaz de línea de comando estándar. Magit nos ofrece comandos amigables para las acciones de Git. Yo uso Magit porque simplifica las interacciones con Git, y sólo necesito recordar unos pocos comandos básicos. Es bastante amigable, y si quieres explorar los comandos de Git que Magit ejecuta internamente, están fácilmente disponibles para que puedas auditarlos o estudiarlos.

Si Git no está instalado en tus sistema, podés seguir las instrucciones oficiales de instalación [aquí](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

Después de instalar Git, configurá tu nombre y email para asociar los commits con tu identidad:

```shell
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

Posteriormente, instalá **Magit** para emacs. Si no lo has instalado aún, re recomiendo que lo hagás vía [MELPA](https://melpa.org/), el repositorio de paquetes de emacs. Añadí las siguientes líneas a tu archivo `.emacs`:

```lisp
(require 'package)
(add-to-list 'package-archives
             '("melpa" . "https://melpa.org/packages/") t)
```

Seleccioná el bloque de texto que acabas de pegar y evaluálo en emacs con `M-x eval-region RET`, y luego refrescá tu lista de paquetes con:

```lisp
M-x package-refresh-contents RET
```

Ahora podés instalar Magit y sus dependencias ejecutando:


```lisp
M-x package-install RET magit RET
```

Para asegurar que todo funciona como esperamos, reiniciá tu sesión de emacs. Esto restablece el `load-path` y evita problemas potenciales asociados a configuraciones anteriores.

#### Clonando el Repositorio
Ahora es el momento de clonar el repositorio de GitHub a tu computador.

Navegá al directorio donde querés clonar el repositorio. Por ejemplo, si querés clonarlo en `~/Documentos`, abrí el modo Dired de emacs con `C-x D` y escribí `~/Documents` para abrir el directorio correspondiente.

Ahora corré los siguientes comandos en emacs:

```lisp
M-x magit-clone
```

Magit te preguntará de dónde querés clonar el repositorio. Escogé `[u]` para **URL**. Luego pegá la URL del repositorio de tu página de GitHub. La podés encontrar haciendo click en el botón verde que dice  "<> Code" y copiás la URL para HTTPS. Por ejemplo:

```plaintext
https://github.com/PavoDive/dummyaddition.git
```

Pegá esta URL en el minibuffer de Magit.

Magit te va a pedir confirmar el nombre del subdirectorio en el que el repositorio será clonado. Por defecto, él nos sugiere el nombre del repositorio (en nuestro caso `dummyaddition`). Yo recomiendo que lo mantengás así para guardar consistencia.

También te preguntará **"Set remote.pushDefault to origin? (y or n)"** (¿Establecer remote.pushDefault a origin? (s o n)). Ya que este es el comienzo de un nuevo repositorio, podés escoger **yes (y)** con tranquilidad.

Después de unos instantes, Magit terminará el proceso de clonado y mostrará un nuevo buffer con los detalles del repositorio y el commit más reciente. Podés presionar `TAB` para expandir los detalles de los commits.

Para verificar que el repositorio ha sido clonado, refrescá el buffer de Dired presionando `g`. Deberías ver un nuevo directorio —`dummyaddition`— dentro de tu directorio `~/Documents`. Navegá a este directorio y vas a encontrar:

```plaintext
drwxrwxr-x  8 gp gp 4.0K Jan 11 17:44 .git
-rw-rw-r--  1 gp gp  671 Jan 11 17:44 .gitignore
-rw-rw-r--  1 gp gp  35K Jan 11 17:44 LICENSE
-rw-rw-r--  1 gp gp   26 Jan 11 17:44 README.md
```

Estos incluyen los archivos y directorios creados durante la configuración de GitHub:

- `.gitignore`: Preconfigurado para ignorar archivos innecesarios.
- `LICENSE`: La licencia que escogiste durante la configuración inicial.
- `README.md`: El archivo README inicial.
- `.git/`: Un directorio oculto que contiene metadata sobre el control de versiones.

### Listo Para Comenzar a Desarrollar
¡Ahora estás listo para comenzar a desarrollar tu paquete de R! 🎉

### Paquetes Requeridos
Para hacer la escritura de paquetes de R más fácil y eficiente, necesitamos instalar dos paquetes esenciales:

- **`rmarkdown`**: Convierte documentos de R Markdown a otros varios formatos.
- **`devtools`**: Una colección de herramientas de desarrollo de paquetes. Literalmente ¡un paquete que te ayuda a desarrollar paquetes!

Si estos no están instalados aún, comenzá una nueva sesión de R en tu directorio `Documents` ejecutando `M-x R` en emacs. Una vez la sesión comience, ingresá los siguientes comandos:

```r
install.packages("rmarkdown")
install.packages("devtools")
```

Estos paquetes podrían requerir dependencias adicionales. Ya que el proceso de instalación varía dependiendo de tu sistema, es posible que encontrés errores. Si es así ¡no te preocupés! Respirá profundo, buscá en DuckDuckGo los errores y resolvé cualquier problema que encontrés.

{{< figure src="/images/keep-calm-try-again.jpg" attr="Mantené la Calma e Intentá de Nuevo" align=center link="/images/keep-calm-try-again.jpg" target="_blank" >}}

Después de que hayás instalado los paquetes, cargálos en tu sesión de R:

```r
library(rmarkdown)
library(devtools)
```

### Creá el Paquete
Ahora que hemos configurado las herramientas necesarias, vamos a crear el paquete. En tu sesión de R, ejecutá:

```r
create("dummyaddition")
```

Esto creará un nuevo paquete de R llamado `dummyaddition`, dentro del correspondiente directorio `dummyaddition`.

⚠ **Nota Importante**: Aseguráte de que la sesión de R **no está** dentro del directorio `dummyaddition` cuando ejecutés `devtools::create`. De lo contrario tratará de crear un paquete anidado (es decir, un directorio `dummyaddition` dentro de otro directorio `dummyaddition`), lo cual puede ser problemático.

Cuando ejecuta, la función muestra la siguiente información en tu consola de R:

```
✔ Setting active project to "/home/gp/Documents/dummyaddition".
✔ Creating R/.
✔ Writing DESCRIPTION.
Package: dummyaddition
Title: What the Package Does (One Line, Title Case)
Version: 0.0.0.9000
Authors@R (parsed):
    * First Last <first.last@example.com> [aut, cre]
Description: What the package does (one paragraph).
License: `use_mit_license()`, `use_gpl3_license()` or friends to
    pick a license
Encoding: UTF-8
Roxygen: list(markdown = TRUE)
RoxygenNote: 7.3.2
✔ Writing NAMESPACE.
✔ Setting active project to "<no active project>".
```

#### ¿Qué Fue Creado?
Notarás los siguientes elementos en tu directorio `dummyaddition`:

- **DESCRIPTION**: Describe la metadata de tu paquete (por ejemplo, nombre, autor, versión y descripción). Editá este archivo siguiendo las instrucciones en él.
- **NAMESPACE**: Maneja las funciones importadas y exportadas. Este archivo se genera automáticamente—evitá editarlo manualmente.
- **R/**: Un directorio para que guardés las funciones y código de tu paquete. En este momento estará vacío.

En este punto, ya tenemos el esqueleto del paquete. ¡Ahora escribamos el código!

### Escribiendo el Código
Vamos a definir la función central de nuestro paquete. Creá un nuevo archivo llamado `addition.R` dentro del directorio `R/` y en él ingresá el siguiente código:

```r{linenos=true}
addition <- function(a, b) {
  if (length(a) > 1 || length(b) > 1) {
    stop("I can't accept a vector. Sorry.")
  }
  if (is.numeric(a) == TRUE && is.numeric(b) == TRUE) {
    a + b
  }else {
    paste0(as.character(a), as.character(b))
  }
}
```

Por supuesto, tu paquete puede incluir múltiples funciones de complejidad variable. Como recomendación general, poné cada función **exportada** en su propio archivo. Está bien incluir funciones auxiliares (internas) en el mismo archivo que las utiliza.

### Documentando las Funciones
La documentación es un aspecto crucial del desarrollo de paquetes, y con `devtools` fácilmente la podés manejar utilizando `roxygen2`. Esto involucra la escritura de comentarios especiales en el encabezado de cada función para describe su propósito, uso y argumentos.

Vamos a documentar nuestra función `addition` añadiendo el siguiente texto **encima** de su definición en el archivo `addition.R`:

```r
#' Add two numeric values or paste two character values.
#' 
#' This function takes two single values and adds them,
#' if they are numeric, or pastes them together, otherwise.
#' The function checks the length of each argument, and
#' returns an error if any of the arguments has length
#' greater than one.
#' 
#' @usage addition(a, b)n
#' @keywords addition, pasting, sum.
#' @param a A single value that is numeric, character or that
#'  can be coerced to character.
#' @param b A single value that is numeric, character or that
#'  can be coerced to character.
#' @return A numeric value with the sum of a and b, if both
#'  are numeric, or a string if both values can be coerced to
#'  to string.
#' @examples
#' # Adding numerc values
#' addition(1, 6)
#' # Pasting strings together
#' addition("nice ", "function")
#' @export
```

#### Puntos Clave Sobre la Documentación:

- `#'` denota un comentario de documentación.
- La etiqueta `@export` es esencial para hacer disponible nuestra función a usuarios del paquete. Las funciones sin esta etiqueta son tratadas como internas.
- Etiquetas como `@param`, `@return`, `@keywords` y `@examples` ofrecen detalles estructurados a cerca de la función.

### Una Nota Sobre la Importación de Otros Paquetes
Si tu paquete depende de librerías externas, deberás incluir la etiqueta `@import` en tu documentación, así:

```r
#' @import data.table
```

Esto asegura que el paquete importa correctamente la funcionalidad externa.

Con la función principal implementada y documentada, estás en camino de construir un paquete funcional de R . ¡Gran trabajo hasta ahora! 🎉

### Construir la Documentación
Para construir la documentación para tu paquete, aseguráte de que tu directorio de trabajo está establecido como el directorio raíz de tu paquete (`dummyaddition`). Si no has cerrado tu sesión de R, simplemente podés usar:

```r
setwd("dummyaddition")
```

Si ya habías cerrado la sesión, simplemente comenzá una nueva sesión de R en el directorio correcto y cargá de nuevo las librerías (`devtools` y `rmarkdown`) usando `library()`.

Ahora, usá la función `devtools::document()`, un wrapper de la función `roxygen2::roxygenize()`, para generar la documentación. Ejecutá:

```r
document()
```

Esto produce el siguiente output:

```
ℹ Updating dummyaddition documentation
ℹ Loading dummyaddition
Writing NAMESPACE
Writing addition.Rd
```

Veamos en detalle lo que ocurre cuando corrés este comando:

- El archivo **`NAMESPACE`** fue actualizado automáticamente: ahora incluye la información requerida para exportar la función `addition`.
- El directorio **`man`** fue creado, si no existía ya. Este directorio contiene archivos de documentación para sus funciones exportadas, escritos en formato `.Rd`. Por ejemplo:
    - `addition.Rd`: Este archivo fue generado a partir de los comentarios especiales en el archivo `addition.R`. Contiene documentación en un formato reconocido por R. **Nota**: No modifiqués este archivo directamente, ya que es generado automáticamente.
    
En este punto, ya podés acceder a la ayuda de tu función, aún cuando el paquete no ha sido aún instalado. Para hacerlo, seguí este proceso específico:

```r
> ? 
+ addition
```

En este código
- `>` representa el prompt de R.
- `+` indica una línea de continuación (una característica de emacs).

Esto quiere decir: escribí `?` y cuando emacs te ofrezca la siguiente línea, escribí `addition`.

Cuando corrés este comando, vas a ver una página de ayuda para tu función cuidadosamente formateada, presentada como si tu paquete estuviera completamente instalado.

```
ℹ Rendering development documentation for "addition"
addition             package:dummyaddition             R Documentation

Add two numeric values or paste two character values.

Description:

     This function takes two single values and adds them, if they are
     numeric, or pastes them together, otherwise. The function checks
     the length of each argument, and returns an error if any of the
     arguments has length greater than one.

Usage:

     addition(a, b)
     
Arguments:

       a: A single value that is numeric, character or that can be
          coerced to character.

       b: A single value that is numeric, character or that can be
          coerced to character.

Value:

     A numeric value with the sum of a and b, if both are numeric, or a
     string if both values can be coerced to to string.

Examples:

     # Adding numerc values
     addition(1, 6)
     # Pasting strings together
     addition("nice ", "function")
```

Esto confirma que tu documentación está funcionando correctamente y que tu paquete está bien estructurado.

#### Un PDF Bonito
Vamos a generar un PDF bien formateado para nuestro paquete—exactamente como los que has visto en los paquetes famosos de R.

Para producir este PDF, vas a necesitar utilizar el terminal de emacs (shell) o eshell. Abrílo con `M-x shell` o con `M-x eshell` y navegá al directorio **padre** de tu proyecto (`Documents` en este ejemplo). Luego corré el siguiente comando en la terminal:

```shell
R CMD Rd2pdf dummyaddition
```

Esto generará un archivo PDF llamado `dummyaddition.pdf` en el directorio `Documentos`.

⚠ **Nota Importante**: Podrías necesitar LaTeX y Pandoc instalados en tu sistema para poder crear el PDF. Si no están instalados, buscá en línea instrucciones específicas de instalación para tu plataforma (y preparáte un café, eso puede requerir tiempo y paciencia).

### Actualizá el Archivo `README.md`

El archivo `README.md` es la "cara" de tu paquete. Es lo que la gente ve primero cuando visitan tu repositorio (especialmente en GitHub), así que es importante actualizarlo con información relevante.

Como mínimo, tu archivo `README.md` debe incluir:

- **Lo que tu paquete hace**: Brinda una explicación clara y simple del problema que tu paquete resuelve.
- **Cómo instalarlo**: Incluye instrucciones de instalación. Por ejemplo, podés añadir lo siguiente si tu paquete está alojado en GitHub:

```r
devtools::install_github("yourusername/dummyaddition")
```

Reemplazá `"yourusername"`con tu usuario de GitHub y `"dummyaddition"` con el nombre de tu paquete.

### Subiendo tu Paquete Nuevo a GitHub
Una vez has completado tu código, documentación y actualizaciones al archivo `README.md`, es hora de subir (push) tus cambios a GitHub. Magit hace este proceso fácil y eficiente.

1. **Abrí el buffer de Magit** de tu repositorio, típicamente llamado `magit: dummyaddition`. Si no está abierto, lo podés activar con el comando:

   ```
   M-x magit-status
   ```

2. **Refrescá el status del repositorio** presionando `g` en el buffer de Magit. Verás algo como esto:

   ```
   Head:     main Initial commit
   Merge:    origin/main Initial commit
   Push:     origin/main Initial commit
   
   Untracked files (4)
   Unstaged changes (1)
   
   Recent commits
   ```

   Podés expandir secciones específicas (por ejemplo untracked files, unstaged changes, recent commits) presionando `TAB`.

#### Staging y Committing Cambios

- **Untracked files**: Cuando expandís la sección "Untracked files", verás archivos como `DESCRIPTION`, `NAMESPACE`, y directorios como `man/` y `R/`. Estos son los nuevos archivos y directorios que hemos creado durante el proceso de desarrollo.
  - Stage estos cambios seleccionando el archivo o directorio y presionando `s` (de _stage_). Los archivos staged ahora aparecerán en la sección "Staged changes".

- **Unstaged changes**: Vas a notar que el archivo `README.md` aparece aquí porque fue modificado (no creado nuevo). Stage estos cambios con `s` nuevamente. Magit podría preguntarte si querés stage todos los cambios —presioná **y** para confirmar.

- Una vez todos los cambios están staged en la sección "Staged changes", hacé el commit de ellos presionando `c` (de _commit_). Un menú aparecerá. Presioná `c` nuevamente para crear el commit.

#### Escribiendo el Mensaje de Commit
Una vez comencés el proceso de commit, un buffer se abrirá para que escribás el mensaje de commit. Los mensajes de commit debe ser concisos y explicativos. Por ejemplo:

- **Título**: Resumí el commit claramente en una línea.
- **Cuerpo (opcional)**: Adicioná más detalles si es necesario, especialmente si los cambios son complejos.

Aquí está un ejemplo de mensaje de commit para nuestro trabajo:

```
Código y Documentación Generada
- Creada la función central `addition`.
- Creada por auto-generación la documentación con roxygen2.
- Actualizado el archivo README.md.
```

Después de escribir tu mensaje de commit, guardálo presionando `C-c C-c` para confirmar y cerrar el buffer.

Esto actualizará el buffer de magit, que ahora podrá decir algo como:

```
Unmerged into origin/main (1)
56d6bd3 main Código y Documentación Generada
```

Esto significa que tus cambios han sido commited localmente pero aún no hemos subido (pushed) al repositorio de GitHub.

#### Subiendo (Pushing) a GitHub
Para pushar tus cambios a GitHub, presioná `P` (de _push_) en el buffer de Magit. Un menú aparecerá, presioná `p` para subir tus cambios a `origin/main`.

Magit podría preguntarte por tus credenciales de GitHub (usuario y password), si no las has configurado para SSH o autenticación con token. Ingresá la información requerida, y después de un momento, el buffer de Magit reflejará que los cambios han sido pushed:

```
Recent commits
56d6bd3 origin/main Código y Documentación Generada
4b1b6c8 Initial commit
```

En este punto tus cambios están activos en GitHub. Podés verificarlos visitando tu repositorio en  GitHub.

## Compartí tu Paquete
¡Tu camino no está completo hasta que no compartás tu trabajo! A pesar de que este paquete resuelva un problema particular para vos, es bien probable que alguno de los 8 mil millones de humanos que poblamos este planeta 🌍 pueda enfrentar el mismo reto en el futuro. Compartir tu trabajo puede ayudarlos a ellos también.

Aquí hay algunas formas de compartir tu paquete:

- Publicá sobre él en tus **redes sociales**.
- Escribí un **blog** sobre lo que hace.
- Compartílo en plataformas como [Hacker News](https://news.ycombinator.com) o [Lobste.rs](https://lobste.rs).
- Mencionálo en **foros o comunidades** relevantes.

Cualquier método que escojás, ¡no te olvidés de compartir tu excelente trabajo con el mundo!
