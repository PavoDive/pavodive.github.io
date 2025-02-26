---
author: "Pavodive"
title: "Configure mu4e for gmail accounts in emacs"
date: "2025-02-24"
description: "This post shows how to configure mu4e for gmail accounts in emacs"
tags: ["mu4e", "emacs"]
categories: ["emacs"]
series: ["How-to"]
aliases: ["mu4e-gmail-emacs"]
ShowToc: true
TocOpen: true
---

In this blog post, I want to show how I configured **mu4e** to read my Gmail accounts in Emacs.

<!--more-->

A lot of my time in front of a computer is spent in Emacs. In fact, this very blog is being written in Emacs. My agenda is kept in Emacs, I code in Emacs, write in Emacs, and even read books in Emacs. So, naturally, I wanted to manage my email in Emacs as well.

Since I have multiple Gmail accounts, I needed to configure an email reader within Emacs.

## Email Alternatives in Emacs

There are two main alternatives for reading email in Emacs: **Gnus** and **mu4e**. As with many things in Emacs, which one you choose depends on personal preference—both are powerful, highly customizable, and offer numerous features.

I tried **Gnus** some time ago. It’s a very powerful message reader, not limited to email, and allows extensive customization. One of its highly praised features is the ability to assign scores to messages, helping you prioritize what's most important.

However, to be honest, I never really developed a good workflow with Gnus. It was too feature-rich for my needs. I don’t receive an overwhelming number of emails, as I’ve consistently trimmed unnecessary messages—no subscriptions, no newsletters, and most of my friends and colleagues know that I prefer a phone call over an email.

So, I tried the alternative: **mu4e**. In some ways, **mu4e** is simpler than **Gnus**. It’s not as multipurpose, but it still offers a lot of customization and features.

## mu4e

When I first looked into **mu4e**, it seemed a bit complicated to install and configure—especially for Gmail. So, I approached it with some skepticism. **mu4e** was developed and is maintained by Dirk-Jan Binnema, and you can check out its official page [here](https://www.djcbsoftware.nl/code/mu/mu4e.html "mu4e docs").

While the documentation is great, it isn’t exactly beginner-friendly.

During my research, I found one particularly useful resource for installing and configuring **mu4e** with Gmail: [this Reddit thread](https://www.reddit.com/r/emacs/comments/bfsck6/mu4e_for_dummies/). It's quite comprehensive, but since it's from 2019, I struggled with the various changes Google has made to Gmail since then.

Most of what I’ll be sharing here is a reworking of what Reddit user **skizmi** wrote.

## Installing mu4e and Dependencies


I needed to install `isync` along with some libraries required by **mu4e**. Since I’m using a Debian-based Linux distribution, I ran:

```shell
sudo apt update
sudo apt install isync -f
sudo apt install libgmime-3.0-dev libxapian-dev
```

This installs `isync` and the libraries required by **mu4e**.

Now, to install **mu** (I use an `apps` directory to store installed software):

```shell
mkdir -p apps/mu4e
git clone https://github.com/djcb/mu.git
cd mu
./autogen.sh && make
sudo make install
```

This should install **mu** on your system. If you run into errors, carefully read the messages—it's likely you're missing some dependencies that can be installed with:

```shell
sudo apt install [missing-library]
```

## Getting your Application-Specific Password From Google
Now, we need an application-specific password for your Gmail account(s). Google encourages signing in via OAuth, but they still allow app-specific passwords for cases like this. Check [this support page](https://support.google.com/accounts/answer/185833) for more details.

To generate an app-specific password:

1. Visit [this link](https://myaccount.google.com/apppasswords).
2. Enter your credentials (you may need to provide two-factor authentication if enabled).
3. Specify a name for your app (choose something meaningful to **you** in the future).
4. Google will provide a 16-character password formatted like this:

   ```
   aaaa bbbb cccc dddd
   ```

From the same page, you can remove unused or obsolete app-specific passwords. It’s good security practice to delete any that are no longer needed.

Save this password—we’ll use it in the next steps.

## Creating the `mbsync` Configuration Files

We need two files:
1. `.mbsyncrc`
2. `.mbsyncpass` (one per Gmail account)

### `.mbsyncpass-test.gpg`

I recommend saving this file in `~/.emacs.d/mu4e/`. In Emacs, create it with:

```emacs-lisp
C-x C-f ~/.emacs.d/mu4e/.mbsyncpass-test.gpg
```

(Of course, replace "test" with your actual account name.) Paste your 16-character password into the file:

```
aaaa bbbb cccc dddd
```

When you save (`C-x C-s`), Emacs will prompt you to select recipients for encryption. Confirm by hitting "OK", provide a password (and confirm it), and your Gmail app-specific password will be encrypted.

Repeat this for each Gmail account you’re configuring.

### `.mbsyncrc`
Save this file in `~/.emacs.d/mu4e/`. It contains account details, mail directories, and Gmail folder configurations.

**Note**: Check whether your Gmail Trash folder is named **"Trash"** or **"Bin"**—this is crucial for proper operation.

The file's contents are:

```config
# ACCOUNT INFORMATION
IMAPAccount test-gmail
# Address to connect to
Host imap.gmail.com
User test@gmail.com
PassCmd "gpg -q --for-your-eyes-only --no-tty -d ~/.emacs.d/mu4e/.mbsyncpass-test.gpg"
AuthMechs LOGIN
SSLType IMAPS
SSLVersions TLSv1.2
CertificateFile /etc/ssl/certs/ca-certificates.crt

# REMOTE STORAGE (USE THE IMAP ACCOUNT SPECIFIED ABOVE)
IMAPStore test-gmail-remote
Account test-gmail

# LOCAL STORAGE (CREATE DIRECTORIES with mkdir -p ~/Maildir/test-gmail)
MaildirStore test-gmail-local
Path ~/Maildir/test-gmail/
Inbox ~/Maildir/test-gmail/INBOX

# CONNECTIONS SPECIFY LINKS BETWEEN REMOTE AND LOCAL FOLDERS
#
# CONNECTIONS ARE SPECIFIED USING PATTERNS, WHICH MATCH REMOTE MAIl
# FOLDERS. SOME COMMONLY USED PATTERS INCLUDE:
#
# 1 "*" TO MATCH EVERYTHING
# 2 "!DIR" TO EXCLUDE "DIR"
# 3 "DIR" TO MATCH DIR

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

# GROUPS PUT TOGETHER CHANNELS, SO THAT WE CAN INVOKE
# MBSYNC ON A GROUP TO SYNC ALL CHANNELS
#
# FOR INSTANCE: "mbsync gmail" GETS MAIL FROM
# "gmail-inbox", "gmail-sent", and "gmail-trash"
#
Group test-gmail
Channel test-gmail-inbox
Channel test-gmail-sent
Channel test-gmail-trash
Channel test-gmail-all
Channel test-gmail-starred
```

If your trash is named **"Bin"**, replace:

```config
Channel test-gmail-trash
Far :test-gmail-remote:"[Gmail]/Trash"
Near :test-gmail-local:"[test].Trash"
```

with:

```config
Channel test-gmail-trash
Far :test-gmail-remote:"[Gmail]/Bin"
Near :test-gmail-local:"[test].Bin"
```

### `.authinfo.gpg`

If you don’t have a `.authinfo.gpg` file, create it in `~/.authinfo.gpg`. Add:

```config
machine smtp.gmail.com login test password "aaaa bbbb cccc dddd" port 465
machine imap.gmail.com login test password "aaaa bbbb cccc dddd" port 993
```

Replace “test” with your actual username and insert your app-specific password.

For multiple accounts, add entries for each one.

## Creating Maildir Folders and Running `mbsync`

Next, create the Maildir directories and run `mbsync`:

```shell
mkdir -p ~/Maildir/test-gmail
```

(Be sure the name matches the **IMAPAccount** field in `.mbsyncrc`.) Repeat for each account.

Then, run:

```shell
mbsync -c .emacs.d/mu4e/.mbsyncrc -Dmn test-gmail
```

## Indexing Maildir
Now, index your mail. This may take time, depending on the number and size of your stored messages.

```shell
mu init --maildir=Maildir
mu init --my-address=test@gmail.com
mu index
```

For multiple accounts, list them all in one line:

```shell
mu init --my-address=test@gmail.com --my-address=other@gmail.com
```

## Adding mu4e Settings to `.emacs`

Finally, add the necessary **mu4e** settings to your `.emacs` (or another initialization file).

```lisp
(require 'org-mime)

(add-to-list 'load-path "/usr/local/share/emacs/site-lisp/mu4e/")
(require 'mu4e)

(setq mu4e-maildir (expand-file-name "~/Maildir"))

; get mail
(setq mu4e-get-mail-command "mbsync -c ~/.emacs.d/mu4e/.mbsyncrc -a"
  mu4e-view-prefer-html t
  mu4e-update-interval 180
  mu4e-headers-auto-update t
  mu4e-compose-signature-auto-include nil
  mu4e-compose-format-flowed t)

;; to view selected message in the browser, no signin, just html mail
(add-to-list 'mu4e-view-actions
  '("ViewInBrowser" . mu4e-action-view-in-browser) t)

;; enable inline images
(setq mu4e-view-show-images t)
;; use imagemagick, if available
(when (fboundp 'imagemagick-register-types)
  (imagemagick-register-types))

;; every new email composition gets its own frame!
(setq mu4e-compose-in-new-frame t)

;; don't save message to Sent Messages, IMAP takes care of this
;;(setq mu4e-sent-messages-behavior 'sent)

(add-hook 'mu4e-view-mode-hook #'visual-line-mode)

;; <tab> to navigate to links, <RET> to open them in browser
(add-hook 'mu4e-view-mode-hook
  (lambda()
;; try to emulate some of the eww key-bindings
(local-set-key (kbd "<RET>") 'mu4e~view-browse-url-from-binding)
(local-set-key (kbd "<tab>") 'shr-next-link)
(local-set-key (kbd "<backtab>") 'shr-previous-link)))

;; from https://www.reddit.com/r/emacs/comments/bfsck6/mu4e_for_dummies/elgoumx
(add-hook 'mu4e-headers-mode-hook
      (defun my/mu4e-change-headers ()
	(interactive)
	(setq mu4e-headers-fields
	      `((:human-date . 25) ;; alternatively, use :date
		(:flags . 6)
		(:from . 22)
		(:thread-subject . ,(- (window-body-width) 70)) ;; alternatively, use :subject
		(:size . 7)))))

;; spell check
(add-hook 'mu4e-compose-mode-hook
    (defun my-do-compose-stuff ()
       "My settings for message composition."
       (visual-line-mode)
;       (org-mu4e-compose-org-mode)
           (use-hard-newlines -1)
       (flyspell-mode)))

(require 'smtpmail)

;;rename files when moving
;;NEEDED FOR MBSYNC
(setq mu4e-change-filenames-when-moving t)

;;set up queue for offline email
;;use mu mkdir  ~/Maildir/acc/queue to set up first
(setq smtpmail-queue-mail nil)  ;; start in normal mode

;; copied from https://emacs.stackexchange.com/questions/46257/sending-email-fails-with-process-smtpmail-not-running
(setq smtpmail-stream-type 'ssl)

;;from the info manual
(setq mu4e-attachment-dir  "~/Downloads")

(setq message-kill-buffer-on-exit t)
(setq mu4e-compose-dont-reply-to-self t)

(require 'mu4e-org)

;; convert org mode to HTML automatically
(setq org-mu4e-convert-to-html t)

;;from vxlabs config
;; show full addresses in view message (instead of just names)
;; toggle per name with M-RET
(setq mu4e-view-show-addresses 't)

;; don't ask when quitting
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


If configuring multiple accounts, add them to the `mu4e-contexts` list, ensuring each has a **unique** `:name`.

Again, configure the trash folder correctly (either **"Trash"** or **"Bin"**).

## Enjoy mu4e
Now, restart Emacs and launch **mu4e**:

```emacs-lisp
M-x mu4e
```

You should see the **mu4e** menu. From there, it's relatively straightforward. To explore available commands, press `h` within the `mu4e-main` buffer.

Happy emailing in Emacs! 🚀
