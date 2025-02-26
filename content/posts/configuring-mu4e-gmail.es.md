---
author: "Pavodive"
title: "Configurando mu4e para cuentas de gmail en emacs"
date: "2025-02-24"
description: "Este artículo presenta cómo configurar mu4e para cuentas de gmail en emacs"
tags: ["mu4e", "emacs"]
categories: ["emacs"]
series: ["How-to"]
aliases: ["mu4e-gmail-emacs"]
ShowToc: true
TocOpen: true
---

En este artículo quiero mostrarles como configuré **mu4e** para gestionar mis cuentas de Gmail en Emacs.

<!--more-->

Una buena parte de mi tiempo frente a un computador es utilizada en Emacs. De hecho, este preciso blog está siendo escrito en Emacs. Mi agenda está  en Emacs, programo en Emacs, escribo en Emacs e incluso leo libros en Emacs. Así que lo más natural es que yo quiera getsionar mi email en Emacs también.

Yo utilizo múltiples cuentas de Gmail, para las cuales necesitaba configurar un lector de email dentro de Emacs.

## Alternativas de Gestión de Email en Emacs

Hay dos alternativas principales para leer email en Emacs: **Gnus** y **mu4e**. Como la mayoría de las cosas en Emacs, cuál escoger depende de las preferencias personales de cada quien –ambas son poderosas, altamente personalizables y ofrecen numerosas características y funcionalidades.

Yo intenté usar **Gnus** hace algún tiempo. Es un lector de mensajes bastante poderoso, no sólo para email, y permite una personalización enorme. Uno de sus características más celebradas es su capacidad de asignar puntajes a los mensajes, ayudándole al usuario a priorizar lo más importante.

Sin embargo, para ser honesto, nunca pude desarrollar un buen flujo de trabajo con Gnus. Tal vez tiene demasiadas funcionalidades para mis necesidades. Yo no recibo una gran cantidad de emails, ya que de manera consistente he recortado los mensajes innnecesarios –nada de suscripciones, ni _newsletters_ y la mayoría de mis amigos y colegas saben que prefiero una llamada telefónica a un email.

Así que intenté con la alternativa: **mu4e**. En cierta forma, **mu4e** es más simple que **Gnus**. No es tan multi-propósito, pero de todas formas ofrece muchas características y personalización.

## mu4e

Cuando investigué **mu4e** por primera vez, me pareció un poco complicado de instalar y configurar, especialmente para Gmail. Así que seguí leyendo y buscando con un poco de escepticismo. **mu4e** fue desarrollada y es mantenida por Dirk-Jan Binnema, y podés chequear su página oficial [aquí](https://www.djcbsoftware.nl/code/mu/mu4e.html "mu4e docs").

La documentación es buena, pero no es exactamente amistosa con los novatos.

Durante mi investigación, encontré una fuente particularmente útil para instalar y configurar **mu4e** con Gmail: [este hilo de Reddit](https://www.reddit.com/r/emacs/comments/bfsck6/mu4e_for_dummies/). Es bastante completo, pero fue escrito en 2019, así que tuve algunas dificultades con los varios cambios que Google ha introducido en Gmail desde ese entonces.

La mayor parte de lo que estoy compartiendo en este artículo es un re-make de lo que escribió el usuario **skizmi**.

## Instalando mu4e y sus Dependencias

Es necesario instalar `isync` junto con algunas librerías requeridas por *mu4e*. Ya que yo estoy usando una distribución de Linux basada en Debian, es simplemente ejecutar:

```shell
sudo apt update
sudo apt install isync -f
sudo apt install libgmime-3.0-dev libxapian-dev
```

Esto instala `isync` y las librerías requeridas por **mu4e**.

Ahora a instalar **mu** (yo uso un directorio `apps` para guardar instalaciones similares a esta):

```shell
mkdir -p apps/mu4e
git clone https://github.com/djcb/mu.git
cd mu
./autogen.sh && make
sudo make install
```

Esto deberá haber instalado **mu** en tu sistema. Si encontrás algunos errores, leé cuidadosamente los mensajes –lo más probable es que te hagan falta algunas dependencias (librerías, seguramente) que pueden ser instaladas con:

```shell
sudo apt install [librería-faltante]
```

## Obteniendo tu Password Específico para Aplicación de Google
Ahora necesitamos un password o contraseña específica para aplicación para tu cuenta o cuentas de Google. Google te sugiere iniciar sesión vía OAuth, pero de todas formas también permiten utilizar contraseñas específicas para aplicación, para casos como este. Chequeá [esta página de soporte](https://support.google.com/accounts/answer/185833) para más detalles.

Para generar una contraseña específica para aplicación:

1. Visitá [este enlace](https://myaccount.google.com/apppasswords).
2. Ingresá tus credenciales (es posible que tengás que utilizar tu autenticación en dos pasos, si está habilitada).
3. Especificá un nombre para tu aplicación (escogé un nombre que le haga sentido a tu **vos del futuro**).
4. Google te dará un password de 16 caracteres formateado como este:

   ```
   aaaa bbbb cccc dddd
   ```

Desde esa misma página también podés eliminar contraseñas específicas de aplicación que ya no usés o que sean obsoletas. Es una buena práctica de seguridad eliminar cualquiera que ya no sea necesaria.

Guardá esta contraseña, la vamos a usar en los pasos siguientes.

## Creando los Archivos de Configuración `mbsync`

Necesitamos dos archivos:

1. `.mbsyncrc`
2. `.mbsyncpass` (uno por cada cuenta de Gmail)

### `.mbsyncpass-test.gpg`

Yo recomiendo guardar este archivo en `~/.emacs.d/mu4e/`. En Emacs, creá el archivo con:

```emacs-lisp
C-x C-f ~/.emacs.d/mu4e/.mbsyncpass-test.gpg
```

(Por supuesto, reemplazá "test" con el nombre real de tu cuenta). Pegá tu contraseña de 16 caracteres en el archivo:

```
aaaa bbbb cccc dddd
```

Cuando guardés (`C-x C-s`), Emacs te va a pedir seleccionar recipientes para la encriptación (select recipients for encryption). Confirmá presionando "OK", registrá una contraseña (y confirmála), y tu contraseña específica de aplicación para Gmail estará encriptada y segura.

Repetí esto para cada cuenta de Gmail que estés configurando.

### `.mbsyncrc`
Guardá este archivo en `~/.emacs.d/mu4e/`. Este archivo va a contener los detalles de la cuenta, los directorios de correo y otras configuraciones de Gmail.

**Nota**: Chequeá si el folder de reciclaje de tu Gmail se llama **"Trash"** o **"Bin"**, esto es crucial para la correcta operación.

Los contenidos del archivo son:

```config
# Información de la Cuenta
IMAPAccount test-gmail
# Dirección a la que se conectará
Host imap.gmail.com
User test@gmail.com
PassCmd "gpg -q --for-your-eyes-only --no-tty -d ~/.emacs.d/mu4e/.mbsyncpass-test.gpg"
AuthMechs LOGIN
SSLType IMAPS
SSLVersions TLSv1.2
CertificateFile /etc/ssl/certs/ca-certificates.crt

# Almacenamiento remoto (utilizar la cuenta IMAP especificada arriba)
IMAPStore test-gmail-remote
Account test-gmail

# Almacenamiento local (crear directorios con mkdir -p ~/Maildir/test-gmail)
MaildirStore test-gmail-local
Path ~/Maildir/test-gmail/
Inbox ~/Maildir/test-gmail/INBOX

# Las conexiones especifican enlaces entre lar carpetas remotas y locales
#
# Las conexiones son especificadas utilizando patrones, los cuales hacen
# match con las carpetas de mail remoto. Algunos patrones comunmente usados:
#
# 1 "*" para hacer match con todo.
# 2 "!DIR" para excluir "DIR"
# 3 "DIR" para hacer match con DIR

Channel test-gmail-inbox
Far :test-gmail-remote:
Near :test-gmail-local:
Patterns "INBOX"
Create Both
Expunge Both
SyncState *

Channel test-gmail-trash
Far :test-gmail-remote:"[Gmail]/Trash"
Near :test-gmail-local:"[test].Trash"
Create Both
Expunge Both
SyncState *

Channel test-gmail-sent
Far :test-gmail-remote:"[Gmail]/Sent Mail"
Near :test-gmail-local:"[test].Sent Mail"
Create Both
Expunge Both
SyncState *

Channel test-gmail-all
Far :test-gmail-remote:"[Gmail]/All Mail"
Near :test-gmail-local:"[test].All Mail"
Create Both
Expunge Both
SyncState *

Channel test-gmail-starred
Far :test-gmail-remote:"[Gmail]/Starred"
Near :test-gmail-local:"[test].Starred"
Create Both
Expunge Both
SyncState *

# Los grupos juntan canales, así que podemos invocar
# mbsync en un grupo para sincronizar todos los canales
#
# Por ejemplo: "mbsync gmail" trae el correo de
# "gmail-inbox", "gmail-sent", y "gmail-trash"
#
Group test-gmail
Channel test-gmail-inbox
Channel test-gmail-sent
Channel test-gmail-trash
Channel test-gmail-all
Channel test-gmail-starred
```

Si el reciclaje de tu Gmail se llama **"Bin"**, reemplazá:

```config
Channel test-gmail-trash
Far :test-gmail-remote:"[Gmail]/Trash"
Near :test-gmail-local:"[test].Trash"
```

con:

```config
Channel test-gmail-trash
Far :test-gmail-remote:"[Gmail]/Bin"
Near :test-gmail-local:"[test].Bin"
```

### `.authinfo.gpg`

Si no tenés todavía un archivo `.authinfo.gpg`, creálo en `~/.authinfo.gpg`. Añadíle:

```config
machine smtp.gmail.com login test password "aaaa bbbb cccc dddd" port 465
machine imap.gmail.com login test password "aaaa bbbb cccc dddd" port 993
```

Reemplazá "test" con tu nombre real de usuario e insertá tu contraseña específica de aplicación.

Para varias cuentas, ingresá los datos correspondientes a cada una.

## Creando los Directorios de Maildir y Ejecutando `mbsync`

Luego, creá los directorios en Maildir y corré `mbsync`:

```shell
mkdir -p ~/Maildir/test-gmail
```

(Aseguráte que el nombre corresponde con el campo **IMAPAccount** en el archivo `.mbsyncrc`). Repetí para cada cuenta.

Luego ejecutá:

```shell
mbsync -c .emacs.d/mu4e/.mbsyncrc -Dmn test-gmail
```

## Indexando Maildir
Ahora, vamos a indexar tu correo. Esto puedo tomar algún tiempo, dependiente del número y tamaño de tus mensajes guardados.

```shell
mu init --maildir=Maildir
mu init --my-address=test@gmail.com
mu index
```

Para múltiples cuentas, listálas todas en una sóla línea:

```shell
mu init --my-address=test@gmail.com --my-address=other@gmail.com
```

## Adicionando los Ajustes de mu4e a `.emacs`
Finalmente, adicioná los ajustes necesarios de mu4e a tu `.emacs` (o cualquier otro archivo de inicialización que utilicés).

```lisp
(require 'org-mime)

(add-to-list 'load-path "/usr/local/share/emacs/site-lisp/mu4e/")
(require 'mu4e)

(setq mu4e-maildir (expand-file-name "~/Maildir"))

; obtener el correo
(setq mu4e-get-mail-command "mbsync -c ~/.emacs.d/mu4e/.mbsyncrc -a"
  mu4e-view-prefer-html t
  mu4e-update-interval 180
  mu4e-headers-auto-update t
  mu4e-compose-signature-auto-include nil
  mu4e-compose-format-flowed t)

;; Para ver los mensajes en el navegador, sin signin, solo mail html
(add-to-list 'mu4e-view-actions
  '("ViewInBrowser" . mu4e-action-view-in-browser) t)

;; Habilitar imágenes inline
(setq mu4e-view-show-images t)
;; usar imagemagick si está disponible
(when (fboundp 'imagemagick-register-types)
  (imagemagick-register-types))

;; Cada nuevo mensaje obteiene su propio frame!
(setq mu4e-compose-in-new-frame t)

;; No guardar mensajes a "enviados", IMaP se encarga de esto.
;;(setq mu4e-sent-messages-behavior 'sent)

(add-hook 'mu4e-view-mode-hook #'visual-line-mode)

;; <tab> para navegar a los links, <RET> para abrirlos en el navegador
(add-hook 'mu4e-view-mode-hook
  (lambda()
;; Tratar de emular algunos de los key-bindings de eww
(local-set-key (kbd "<RET>") 'mu4e~view-browse-url-from-binding)
(local-set-key (kbd "<tab>") 'shr-next-link)
(local-set-key (kbd "<backtab>") 'shr-previous-link)))

;; copiado de https://www.reddit.com/r/emacs/comments/bfsck6/mu4e_for_dummies/elgoumx
(add-hook 'mu4e-headers-mode-hook
      (defun my/mu4e-change-headers ()
	(interactive)
	(setq mu4e-headers-fields
	      `((:human-date . 25) ;; alternatively, use :date
		(:flags . 6)
		(:from . 22)
		(:thread-subject . ,(- (window-body-width) 70)) ;; alternatively, use :subject
		(:size . 7)))))

;; Corrector de ortografía
(add-hook 'mu4e-compose-mode-hook
    (defun my-do-compose-stuff ()
       "My settings for message composition."
       (visual-line-mode)
;       (org-mu4e-compose-org-mode)
           (use-hard-newlines -1)
       (flyspell-mode)))

(require 'smtpmail)

;;renombrar archivos al mover
;; es requerido por mbsync
(setq mu4e-change-filenames-when-moving t)

;;set up queue for offline email
;;use mu mkdir  ~/Maildir/acc/queue to set up first
(setq smtpmail-queue-mail nil)  ;; start in normal mode

;; copied from https://emacs.stackexchange.com/questions/46257/sending-email-fails-with-process-smtpmail-not-running
(setq smtpmail-stream-type 'ssl)

;; del manual
(setq mu4e-attachment-dir  "~/Downloads")

(setq message-kill-buffer-on-exit t)
(setq mu4e-compose-dont-reply-to-self t)

(require 'mu4e-org)

;; Convertir org mode a HTML automáticamente
(setq org-mu4e-convert-to-html t)

;; de la configuración de vxlabs
;; mostrar la dirección completa en la vista de mensaje (en lugar de sólo los nombres)
;; toggle per name with M-RET
(setq mu4e-view-show-addresses 't)

;; no confirmar al salir
(setq mu4e-confirm-quit nil)

;; mu4e-context
(setq mu4e-context-policy 'pick-first)
(setq mu4e-compose-context-policy 'always-ask)

(setq mu4e-contexts
  (list
   (make-mu4e-context
    :name "gmail" ;;for test-gmail
    :enter-func (lambda () (mu4e-message "Entering context gmail"))
    :leave-func (lambda () (mu4e-message "Leaving context gmail"))
    :match-func (lambda (msg)
		  (when msg
		(mu4e-message-contact-field-matches
		 msg '(:from :to :cc :bcc) "test@gmail.com")))
    :vars '((user-mail-address . "test@gmail.com")
	    (user-full-name . "Full Name Here")
	    (mu4e-sent-folder . "/test-gmail/[test].Sent")
	    (mu4e-drafts-folder . "/test-gmail/[test].drafts")
	    (mu4e-trash-folder . "/test-gmail/[test].Trash")
	    (mu4e-compose-signature . (concat "My Name Here\n"))
	    (mu4e-compose-format-flowed . t)
	    (smtpmail-queue-dir . "~/Maildir/test-gmail/queue/cur")
	    (message-send-mail-function . smtpmail-send-it)
	    (smtpmail-smtp-user . "test")
	    (smtpmail-starttls-credentials . (("smtp.gmail.com" 465 nil nil)))
	    (smtpmail-auth-credentials . (expand-file-name "~/.authinfo.gpg"))
	    (smtpmail-default-smtp-server . "smtp.gmail.com")
	    (smtpmail-smtp-server . "smtp.gmail.com")
	    (smtpmail-smtp-service . 465)
	    (smtpmail-debug-info . t)
	    (smtpmail-debug-verbose . t)
	    (mu4e-maildir-shortcuts . ( ("/test-gmail/INBOX"            . ?i)
					("/test-gmail/[test].Sent Mail" . ?s)
					("/test-gmail/[test].Bin"       . ?t)
					("/test-gmail/[test].All Mail"  . ?a)
					("/test-gmail/[test].Starred"   . ?r)
					("/test-gmail/[test].drafts"    . ?d)
					               ))))))
```

Si estás configurando varias cuentas, adicionálas a la lista `mu4e-contexts`, asegurándote que cada una tiene un `:name` **único**.

De nuevo, configurá el folder de reciclaje o papelera de la manera correcta (ya sea **"Trash"** o **"Bin"**).

## Disfrutá mu4e
El paso final: reiniciar Emacs y ejecutar **mu4e**:

```emacs-lisp
M-x mu4e
```

Deberías ver el menú de **mu4e**. Desde allí, es relativamente simple. Para explorar los comandos disponibles, presioná `h` dentro del buffer `mu4e-main`.

Feliz email en Emacs! 🚀
