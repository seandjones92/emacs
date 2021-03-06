
# Table of Contents

1.  [About](#org217b21c)
2.  [Installation](#org327eeaf)
3.  [Base Config](#org605d18c)
    1.  [Meta](#org6714e93)
    2.  [Base defaults](#org28ed60c)
    3.  [Home screen](#org59ec1ec)
    4.  [Functions](#org8d75fdc)
    5.  [Org Mode](#org7822490)
    6.  [Mode hooks](#org2264373)
    7.  [Keybindings](#org3f4ac55)
4.  [Packages](#org6330f86)
    1.  [Repositories](#org2e7fa51)
    2.  [use-package](#orgd74e96c)
    3.  [ag](#orgc639328)
    4.  [all-the-icons](#org2a91b39)
    5.  [auto-complete](#orgfe53065)
    6.  [dired-rsync](#org4f16798)
    7.  [docker](#org9942248)
    8.  [dockerfile-mode](#org8547af9)
    9.  [elpy](#orgadcad36)
    10. [emojify](#orgbce5211)
    11. [evil](#org25a5b39)
    12. [evil-magit](#org4c36047)
    13. [flymake-python-pyflakes](#org6eaf89c)
    14. [flymake-shellcheck](#org9560fc4)
    15. [git-gutter](#org88c5dc2)
    16. [helm](#orgc7c69e4)
    17. [helm-ag](#orgea6fc8a)
    18. [helm-projectile](#orgac63bde)
    19. [helm-tramp](#org39e3052)
    20. [magit](#orge4ab98a)
    21. [markdown-mode](#org12e7bc1)
    22. [neotree](#org443428e)
    23. [pipenv](#org01a2401)
    24. [projectile](#org87d4dc5)
    25. [vlf](#org7274edd)
    26. [which-key](#org0799282)
    27. [Themeing](#org27e84be)
5.  [Systemd unit file](#orgcc097d0)
6.  [Nautilus Scripts](#orge62a12e)
7.  [Licensing](#orga6bd5f0)



<a id="org217b21c"></a>

# About

This configuration is based off of the system shown [here](https://github.com/larstvei/dot-emacs). The idea is
that the configuration should serve as it's own plain english
documentation.


<a id="org327eeaf"></a>

# Installation

Clone the repo to `~/.emacs.d`:

    git clone git@github.com:seandjones92/Emacs.git ~/.emacs.d

Once the repo is cloned execute the following commands to prevent the
dynamic configuration from being tracked in git:

    cd ~/.emacs.d
    git update-index --assume-unchanged init.el

If you want to make changes to the repo-version of init.el start tracking again with:

    git update-index --no-assume-unchanged init.el


<a id="org605d18c"></a>

# Base Config

This section contains all of the configurations that do not rely on
external packages. If the configuration cannot be accomplished by a
standalone Emacs installation with no internet connection then it does
not belong here.


<a id="org6714e93"></a>

## Meta

All changes to the config should be made to `init.org` **not** to
`init.el`. The running configuration is generated at first launch and
whenever `init.org` is saved from within Emacs. Any changes made
directly to `init.el` will be lost, it is regenerated regularly.

The initial `init.el` looks like this:

    ;; This file replaces itself with the actual configuration at first run.
    
    ;; We can't tangle without org!
    (require 'org)
    ;; Open the configuration
    (find-file (concat user-emacs-directory "init.org"))
    ;; tangle it
    (org-babel-tangle)
    ;; load it
    (load-file (concat user-emacs-directory "init.el"))
    ;; finally byte-compile it
    (byte-compile-file (concat user-emacs-directory "init.el"))

It tangles the org-file, replacing itself with the actual configuration.

To make sure that the encoding prompt is not shown on launch we start
the init with this line:

    (set-language-environment "UTF-8")

The function defined below generates a new `init.el` each time
`init.org` is saved from within Emacs.

    (defun tangle-init ()
      "If the current buffer is 'init.org' the code-blocks are tangled, and the tangled file is compiled"
      (interactive)
      (when (equal (buffer-file-name)
    	       (expand-file-name (concat user-emacs-directory "init.org")))
        ;; Avoid running hooks when tangling
        (let ((prog-mode-hook nil))
          (org-babel-tangle)
          (byte-compile-file (concat user-emacs-directory "init.el")))))
    
    (add-hook 'after-save-hook 'tangle-init)

This section will generate `README.md` after each save.

    (defun generate-init-readme ()
      "If the current buffer is 'init.org' then 'README.md' is generated"
      (when (equal (buffer-file-name)
    	       (expand-file-name (concat user-emacs-directory "init.org")))
        ;; Avoid running hooks
        (let ((prog-mode-hook nil))
          (org-md-export-to-markdown)
          (rename-file "init.md" "README.md" t))))
    
    (add-hook 'after-save-hook 'generate-init-readme)

If there is anything that should be kept private (not tracked by git,
and therefore not in this configuration) put it in
`~/.emacs.d/private.el`, it will be loaded if it exists.

    (add-hook
     'after-init-hook
     (lambda ()
       (let ((private-file (concat user-emacs-directory "private.el")))
         (when (file-exists-p private-file)
           (load-file private-file)))))


<a id="org28ed60c"></a>

## Base defaults

Here we define the basic look and feel of Emacs.

Make sure that Emacs always starts in the users home directory.

    (setq default-directory (concat (getenv "HOME") "/"))

Set the `PATH` variable.

    (setenv "PATH"
    	(concat
    	 (concat (getenv "HOME") "/.local/bin" ":")
    	 (getenv "PATH")))

Set the `exec-path` variable.

    (add-to-list 'exec-path (concat (getenv "HOME") "/.local/bin"))

Remove scrollbars, menu bars, and toolbars:

    (when (fboundp 'menu-bar-mode) (menu-bar-mode -1))
    (when (fboundp 'tool-bar-mode) (tool-bar-mode -1))
    (when (fboundp 'scroll-bar-mode) (scroll-bar-mode -1))

Instead of typeing "yes" or "no" for interactive functions followed by
`<enter>`, all you need to do is press "y" or "n". No `<enter>`
required!

    (defalias 'yes-or-no-p 'y-or-n-p)

Disable the system bell. No flashing, no sounds.

    (setq ring-bell-function 'ignore)

Enable column numbers.

    (column-number-mode 1)

For me this allows for better handling of parenthesis and quotes. As
you type `(` a matching `)` is also created. The same goes for
quotes. It also adds some inteligent handling.

    (electric-pair-mode 1)
    (require 'paren)
    (setq show-paren-style 'parenthesis)
    (show-paren-mode 1)

Enable spell checking.

    (setq ispell-dictionary "american")

Disable word wrapping by default, I don't like it.

    (set-default 'truncate-lines t)

This change makes `dired` list files with "human readable" size instead of just bytes.

    (setq dired-listing-switches "-lh")

This disables backup and autosave files.

    (setq auto-save-default nil)
    (setq make-backup-files nil)


<a id="org59ec1ec"></a>

## Home screen

Use `*scratch*` as initial screen. Also, modify the message at the top
of the buffer.

    (setq inhibit-startup-screen t)
    (setq initial-scratch-message ";; Scratch page\n\n")


<a id="org8d75fdc"></a>

## Functions

These are my custom functions. I define them all here. If I want them
assigned to a keybinding I do so later in the config.

This function is to be run in `dired`. It prompts for a regular
expression and only shows the files or directories that match the
provided regular expression. This is good for working in directories
with lots of files. Think `ls -al | grep -E <expression>`.

    (defun dired-show-only (regexp)
      "Display files in the current directory that match the given
    regular expression."
      (interactive "sFiles to show (regexp): ")
      (dired-mark-files-regexp regexp)
      (dired-toggle-marks)
      (dired-do-kill-lines))

This function is used to terminate all TRAMP connections and to kill
all buffers associated with TRAMP connections. Sometimes I'll have a
lot going on, machines I'm no longer working on, too many buffers to
sort through and this helps.

    (defun go-local ()
      "Destroy all TRAMP connections and kill all associated
    buffers. Be aware that this will destroy local sudo/root TRAMP
    sessions."
      (interactive)
      (ignore-errors (tramp-cleanup-all-connections))
      (ignore-errors (tramp-cleanup-all-buffers)))

This, in my opinion, is how Emacs should behave by default when saving
files. Strip all white space from the end of the file and the ends of
lines before saving.

    (defun save-buffer-clean ()
      "Strip the trailing whitespace from lines and the end of the
    file and save it."
      (interactive)
      (widen)
      (delete-trailing-whitespace)
      (save-buffer))

Again, another function to get what I would like to be default
behavior. This one handles killing buffers. If there is more than one
buffer and I kill one, kill its window too.

    (defun smart-buffer-kill ()
      "If there is more than one buffer visible in the frame, kill the buffer and
    its associated window."
      (interactive)
      (if (= (count-windows) 1)
          (kill-buffer)
        (kill-buffer-and-window)))

This function allows you to quickly elevate your privileges to
`root`. If called without a prefix you will be placed in dired at `/`,
if you call it with a prefix the current file will be reloaded and
accessed as `root`.

    (defun become-root (&optional prefix)
      "Elevate persmissions to root using TRAMP. If run without a
    prefix, place the user at the root of the file system in
    dired. If run with a prefix open the current file with elevated
    permissions."
      (interactive "P")
      (if prefix
          (find-file (concat "/sudo:root@localhost:" buffer-file-name))
        (dired "/sudo:root@localhost:/")))

This is one I don't use very often but can be useful. Copy the SSH
public key to the clipboard.

    (defun ssh-clip ()
      "Copy '~/.ssh/id_rsa.pub' to clipboard. This will first empty
    the kill-ring (clipboard)"
      (interactive)
      (if (= (count-windows) 1)
          (let ((origin (current-buffer)))
    	(setq kill-ring nil)
    	(find-file "~/.ssh/id_rsa.pub")
    	(mark-page)
    	(kill-ring-save (point-min) (point-max))
    	(kill-buffer)
    	(message "Public key copied to clipboard"))
        (let ((origin (current-buffer)))
          (setq kill-ring nil)
          (find-file-other-window "~/.ssh/id_rsa.pub")
          (mark-page)
          (kill-ring-save (point-min) (point-max))
          (kill-buffer)
          (switch-to-buffer-other-window origin)
          (message "Public key copied to clipboard"))))

This function will open an `eshell` buffer named after the current
directory

    (defun eshell-here ()
      "Opens up a new shell in the directory associated with the
    current buffer's file. The eshell is renamed to match that
    directory to make multiple eshell windows easier."
      (interactive)
      (let* ((parent (if (buffer-file-name)
    		     (file-name-directory (buffer-file-name))
    		   default-directory))
    	 (height (/ (window-total-height) 3))
    	 (name   (car (last (split-string parent "/" t)))))
        (split-window-vertically (- height))
        (other-window 1)
        (eshell "new")
        (rename-buffer (concat "*eshell: " name "*"))))

This function will open `shell` using the full frame.

    (defun full-frame-shell ()
      "Opens `shell' in a full frame."
      (interactive)
      (shell)
      (delete-other-windows))

This function will toggle both the vertical and horizontal scroll
bars. Sometimes it's useful when reviewing large log files and using a
mouse to scroll.

    (defun toggle-bars (arg)
      "Toggle both horizontal and vertical scroll bars."
      (interactive "P")
      (if (null arg)
          (setq arg
    	    (if (frame-parameter nil 'vertical-scroll-bars) -1 1))
        (setq arg (prefix-numeric-value arg)))
      (modify-frame-parameters
       (selected-frame)
       (list (cons 'vertical-scroll-bars
    	       (if (> arg 0)
    		   (or scroll-bar-mode default-frame-scroll-bars)))
    	 (cons 'horizontal-scroll-bars
    	       (when (> arg 0) 'bottom)))))

This function will update the config from my github repository.

    (defun update-config ()
      "Pull the config from github, load and byte-compile it."
      (interactive)
      (async-shell-command "cd ~/.emacs.d && git pull")
      (find-file (concat user-emacs-directory "init.org"))
      (org-babel-tangle)
      (load-file (concat user-emacs-directory "init.el"))
      (byte-compile-file (concat user-emacs-directory "init.el")))

This function will use the gnome-screenshot tool to grab an area
screenshot, create a directory named after the current buffer, save
the screenshot inside that directory, and link to it in the current
buffer.

    (defun my-org-screenshot ()
      "Take a screenshot into a time stamped unique-named file in a
    directory named after the org-buffer and insert a link to this
    file."
      (interactive)
      (if (file-directory-p (concat buffer-file-name ".d"))
          (message "Directory already exists")
        (make-directory (concat buffer-file-name ".d")))
      (setq filename ;; do this first, if exit code is non 0 then do not proceed
    	(concat
    	 (make-temp-name
    	  (concat (buffer-file-name)
    		  ".d/"
    		  (format-time-string "%Y%m%d_%H%M%S_")) ) ".png"))
      (setq relative-filename
    	(concat "./" (mapconcat 'identity
    				(nthcdr (- (length (split-string filename "/")) 2)
    					(split-string filename "/")) "/")))
      (call-process "gnome-screenshot" nil nil nil "--area" "-f" filename)
      (insert (concat "[[" relative-filename "]]"))
      (org-display-inline-images))

This function changes the options passed to `ls` that are used to generate the `dired` output.

    (defcustom list-of-dired-switches
      '("-lh" "-lah")
      "List of ls switches for dired to cycle through.")
    
    (defun cycle-dired-switches ()
      "Cycle through the list `list-of-dired-switches' of swithes for ls"
      (interactive)
      (setq list-of-dired-switches
    	(append (cdr list-of-dired-switches)
    		(list (car list-of-dired-switches))))
      (dired-sort-other (car list-of-dired-switches)))

This configuration is to help handle progress bars in `eshell`. Shamelessly stolen from [here](https://oremacs.com/2019/03/24/shell-apt/).

    (advice-add
     'ansi-color-apply-on-region
     :before 'ora-ansi-color-apply-on-region)
    
    (defun ora-ansi-color-apply-on-region (begin end)
      "Fix progress bars for e.g. apt(8).
    Display progress in the mode line instead."
      (let ((end-marker (copy-marker end))
    	mb)
        (save-excursion
          (goto-char (copy-marker begin))
          (while (re-search-forward "\0337" end-marker t)
    	(setq mb (match-beginning 0))
    	(when (re-search-forward "\0338" end-marker t)
    	  (ora-apt-progress-message
    	   (substring-no-properties
    	    (delete-and-extract-region mb (point))
    	    2 -2)))))))
    
    (defun ora-apt-progress-message (progress)
      (message
       (replace-regexp-in-string
        "%" "%%"
        (ansi-color-apply progress))))

This function will take you directly to the scratch page.

    (defun go-to-scratch ()
      "Quickly go to scratch page."
      (interactive)
      (switch-to-buffer "*scratch*"))


<a id="org7822490"></a>

## Org Mode

Here is my functional configuration of Org Mode.

Enable more babel languages.

    (org-babel-do-load-languages
     'org-babel-load-languages
     '((js . t)
       (sql . t)
       (perl . t)
       (python . t)
       (shell . t)))

Turn font lock on for Org Mode. This makes sure everything looks nice
and pretty.

    (add-hook 'org-mode-hook 'turn-on-font-lock)


<a id="org2264373"></a>

## Mode hooks

This is where mode hooks are manipulated.

For `text-mode` I do want word wrapping enabled and `auto-fill-mode`
enabled. For me this makes sense when thinking about regular old
`*.txt` files.

    (add-hook 'text-mode-hook 'toggle-truncate-lines)

This sets up line numbers for programming.

    (add-hook 'prog-mode-hook 'display-line-numbers-mode)


<a id="org3f4ac55"></a>

## Keybindings

This is where I define my custom keybindings.

    (global-set-key (kbd "C-x C-k") 'smart-buffer-kill)
    (global-set-key (kbd "C-c k") 'kill-this-buffer)
    (global-set-key (kbd "C-x C-s") 'save-buffer-clean)
    (global-set-key (kbd "C-+") 'calc)
    (global-set-key (kbd "C-c S") 'toggle-truncate-lines)
    (global-set-key (kbd "C-!") 'become-root)
    (global-set-key (kbd "C-~") 'eshell)
    (global-set-key (kbd "C-`") 'eshell-here)
    (global-set-key (kbd "C-c l") 'org-store-link)
    (global-set-key [f12] 'toggle-bars)
    (global-set-key [f5] 'update-config)
    (global-set-key [f1] 'go-to-scratch)
    (require 'dired)
    (define-key dired-mode-map [?%?h] 'dired-show-only)
    (define-key dired-mode-map [?%?G] 'find-grep-dired)
    (define-key dired-mode-map [?%?f] 'find-name-dired)
    (define-key dired-mode-map ")" 'cycle-dired-switches)
    (define-key org-mode-map (kbd "C-}") 'my-org-screenshot)
    (require 'flymake)
    (define-key flymake-mode-map [f6] 'flymake-show-diagnostics-buffer)

Enable keybindings that are disabled by default:

    (put 'narrow-to-page 'disabled nil)
    (put 'narrow-to-region 'disabled nil)
    (put 'narrow-to-defun 'disabled nil)


<a id="org6330f86"></a>

# Packages

Configurations after this point rely on external packages. Anything
added from here on out should be designed to fail gracefully in case
the package is not available.


<a id="org2e7fa51"></a>

## Repositories

Here we initialize the Emacs package management system and configure
our package repositories.

    (require 'package)
    (setq package-archives
          '(("gnu" . "https://elpa.gnu.org/packages/")
    	("melpa stable" . "https://stable.melpa.org/packages/")
    	("melpa" . "https://melpa.org/packages/"))
          package-archive-priorities
          '(("melpa stable" . 10)
    	("melpa"        . 0)))

One the repositories are installed let's initialize package management
and get the latest package metadata from the repos.

    (package-initialize)
    (unless package-archive-contents (package-refresh-contents))


<a id="orgd74e96c"></a>

## [use-package](https://github.com/jwiegley/use-package)

We need to configure `use-package` first since it will assist us in
configuring the remaining packages in the config.

First we need to ensure that `use-package` is installed.

    (package-install 'use-package)

Once installed we can start up `use-package`.

    (eval-when-compile
      (require 'use-package))
    (require'bind-key)


<a id="orgc639328"></a>

## [ag](https://github.com/Wilfred/ag.el)

Ag.el allows you to search using ag from inside Emacs. You can filter
by file type, edit results inline, or find files.

    (use-package ag
      :ensure t)


<a id="org2a91b39"></a>

## [all-the-icons](https://github.com/domtronn/all-the-icons.el)

A utility package to collect various Icon Fonts and propertize them
within Emacs.

    (use-package all-the-icons
      :ensure t)


<a id="orgfe53065"></a>

## [auto-complete](https://github.com/auto-complete/auto-complete)

Auto-Complete is an intelligent auto-completion extension for
Emacs. It extends the standard Emacs completion interface and provides
an environment that allows users to concentrate more on their own
work.

    (use-package auto-complete
      :ensure t
      :config
      (ac-config-default)
      (setq-default ac-sources '(ac-source-filename
    			     ac-source-functions
    			     ac-source-yasnippet
    			     ac-source-variables
    			     ac-source-symbols
    			     ac-source-features
    			     ac-source-abbrev
    			     ac-source-words-in-same-mode-buffers
    			     ac-source-dictionary)))


<a id="org4f16798"></a>

## [dired-rsync](https://github.com/stsquad/dired-rsync)

This package adds a single command `dired-rsync` which allows the user
to copy marked files in a `dired` buffer via rsync. This is useful,
especially for large files, because the copy happens in the background
and doesn’t lock up Emacs. It is also more efficient than using tramps
own encoding methods for moving data between systems.

    (use-package dired-rsync
      :ensure t
      :config
      (bind-key "C-c C-r" 'dired-rsync dired-mode-map))


<a id="org9942248"></a>

## [docker](https://github.com/Silex/docker.el)

Supports docker containers, images, volumes, networks, docker-machine
and docker-compose.

    (use-package docker
      :ensure t
      :bind ("C-c d" . docker))


<a id="org8547af9"></a>

## [dockerfile-mode](https://github.com/spotify/dockerfile-mode)

Adds syntax highlighting as well as the ability to build the image
directly (C-c C-b) from the buffer.

    (use-package dockerfile-mode
      :ensure t
      :config
      (add-to-list 'auto-mode-alist '("Dockerfile\\'" . dockerfile-mode)))


<a id="orgadcad36"></a>

## [elpy](https://github.com/jorgenschaefer/elpy)

Elpy is an Emacs package to bring powerful Python editing to Emacs. It
combines and configures a number of other packages, both written in
Emacs Lisp as well as Python. Elpy is fully documented at [Readthedocs](https://elpy.readthedocs.io/en/latest/index.html).

    (use-package elpy
      :ensure t
      :after python
      :bind (:map elpy-mode-map
    	      ("C-." . elpy-goto-definition-other-window))
      :custom
      (elpy-rpc-python-command "python3")
      (python-shell-interpreter "python3")
      :init
      (elpy-enable)
      :config
      (global-company-mode)
      (yas-global-mode)
      (setenv "WORKON_HOME"
    	  (concat (getenv "HOME") "/.local/share/virtualenvs")))


<a id="orgbce5211"></a>

## [emojify](https://github.com/iqbalansari/emacs-emojify)

Emojify is an Emacs extension to display emojis. It can display github
style emojis like :smile: or plain ascii ones like :). It tries to be
as efficient as possible, while also providing a lot of flexibility

    (use-package emojify
      :ensure t
      :config
      (add-hook 'after-init-hook #'global-emojify-mode))


<a id="org25a5b39"></a>

## [evil](https://github.com/emacs-evil/evil)

Evil is an extensible vi layer for Emacs. It emulates the main
features of Vim, [1] turning Emacs into a modal editor. Like Emacs in
general, Evil is extensible in Emacs Lisp.

    (use-package evil
      :ensure t
      :init
      (evil-mode))


<a id="org4c36047"></a>

## [evil-magit](https://github.com/emacs-evil/evil-magit)

This library configures Magit and Evil to play well with each other.

    (use-package evil-magit
      :ensure t)


<a id="org6eaf89c"></a>

## [flymake-python-pyflakes](https://github.com/purcell/flymake-python-pyflakes)

An Emacs flymake handler for syntax-checking Python source code using
pyflakes or flake8.

    (use-package flymake-python-pyflakes
      :ensure t
      :hook python-mode
      :custom
      (flymake-python-pyflakes-executable "flake8"))


<a id="org9560fc4"></a>

## [flymake-shellcheck](https://github.com/federicotdn/flymake-shellcheck)

An Emacs (26+) Flymake handler for bash/sh scripts, using
ShellCheck. Installing Flymake is not necessary as it is included with
Emacs itself.

    (use-package flymake-shellcheck
      :commands flymake-shellcheck-load
      :hook sh-mode)


<a id="org88c5dc2"></a>

## [git-gutter](https://github.com/emacsorphanage/git-gutter)

git-gutter.el is an Emacs port of the Sublime Text plugin [GitGutter](https://github.com/jisaacks/GitGutter).

    (use-package git-gutter
      :ensure t
      :hook (prog-mode . git-gutter-mode))


<a id="orgc7c69e4"></a>

## [helm](https://github.com/emacs-helm/helm)

Helm is an Emacs framework for incremental completions and narrowing
selections.

    (use-package helm
      :ensure t
      :bind (("M-x" . helm-M-x)
    	 ("C-x C-f" . helm-find-files)
    	 ("C-x x" . helm-mini)
    	 ("C-x C-b" . helm-buffers-list)
    	 ("C-c h o" . helm-occur)
    	 ("M-y" . helm-show-kill-ring))
      :custom
      (helm-M-x-fuzzy-match t)
      (helm-buffers-fuzzy-matching t)
      (helm-recentf-fuzzy-match t)
      (helm-semantic-fuzzy-match t)
      (helm-imenu-fuzzy-match t)
      (helm-apropos-fuzzy-match t)
      (helm-lisp-fuzzy-completion t)
      (helm-mode-fuzzy-match t)
      (helm-completion-in-region-fuzzy-match t)
      (helm-net-prefer-curl t)
      (helm-split-window-inside-p t)
      (helm-move-to-line-cycle-in-source t)
      (helm-ff-search-library-in-sexp t)
      (helm-scroll-amount 8)
      (helm-ff-file-name-history-recentf t)
      (helm-grep-default-command "ack-grep -Hn --no-group --no-color %e %p %f")
      (helm-grep-default-recurse-command "ack-grep -H --no-group --no-color %e %p %f")
      (helm-autoresize-mode 1)
      (helm-autoresize-max-height 65)
      :config
      (add-to-list 'helm-sources-using-default-as-input 'helm-source-man-pages)
      (helm-mode 1))
    
    (use-package helm-files
      :bind (:map helm-find-files-map
    	      ([tab] . helm-execute-persistent-action)))


<a id="orgea6fc8a"></a>

## [helm-ag](https://github.com/emacsorphanage/helm-ag)

helm-ag.el provides interfaces of [The Silver Searcher](https://github.com/ggreer/the_silver_searcher) with helm.

    (use-package helm-ag
      :ensure t)


<a id="orgac63bde"></a>

## [helm-projectile](https://github.com/bbatsov/helm-projectile)

Helm UI for Projectile

    (use-package helm-projectile
      :ensure t
      :bind (("C-c a" . helm-projectile-ag)
    	 ("C-c p" . helm-projectile)))


<a id="org39e3052"></a>

## [helm-tramp](https://github.com/masasam/emacs-helm-tramp)

Tramp helm interface for ssh server and docker and vagrant.

    (use-package helm-tramp
      :ensure t
      :bind ("C-c h h" . helm-tramp))


<a id="orge4ab98a"></a>

## [magit](https://github.com/magit/magit)

Magit is an interface to the version control system Git, implemented
as an Emacs package.

    (use-package magit
      :ensure t
      :bind (("C-x g" . magit-status)
    	 ("C-x M-g" . magit-dispatch-popup)))


<a id="org12e7bc1"></a>

## [markdown-mode](https://github.com/defunkt/markdown-mode)

markdown-mode is a major mode for editing Markdown-formatted text.

    (use-package markdown-mode
      :ensure t)


<a id="org443428e"></a>

## [neotree](https://github.com/jaypei/emacs-neotree)

A Emacs tree plugin like NerdTree for Vim.

    (use-package neotree
      :ensure t
      :bind (([f8] . neotree-toggle)
    	 ([f7] . neotree-projectile-action))
      :init
      (evil-define-key 'normal neotree-mode-map (kbd "TAB") 'neotree-enter)
      (evil-define-key 'normal neotree-mode-map (kbd "SPC") 'neotree-quick-look)
      (evil-define-key 'normal neotree-mode-map (kbd "q") 'neotree-hide)
      (evil-define-key 'normal neotree-mode-map (kbd "RET") 'neotree-enter)
      (evil-define-key 'normal neotree-mode-map (kbd "g") 'neotree-refresh)
      (evil-define-key 'normal neotree-mode-map (kbd "j") 'neotree-next-line)
      (evil-define-key 'normal neotree-mode-map (kbd "k") 'neotree-previous-line)
      (evil-define-key 'normal neotree-mode-map (kbd "A") 'neotree-stretch-toggle)
      (evil-define-key 'normal neotree-mode-map (kbd "H") 'neotree-hidden-file-toggle)
      :custom
      (neo-autorefresh nil)
      (neo-theme 'icons)
      (projectile-switch-project-action 'neotree-projectile-action))


<a id="org01a2401"></a>

## [pipenv](https://github.com/pwalsh/pipenv.el)

A Pipenv porcelain inside Emacs.

    (use-package pipenv
      :ensure t)


<a id="org87d4dc5"></a>

## [projectile](https://github.com/bbatsov/projectile)

Projectile is a project interaction library for Emacs. Its goal is to
provide a nice set of features operating on a project level without
introducing external dependencies (when feasible).

    (use-package projectile
      :ensure t
      :config
      ;; (projectile-discover-projects-in-directory default-directory))
      (projectile-discover-projects-in-directory "~/Code"))


<a id="org7274edd"></a>

## [vlf](https://github.com/m00natic/vlfi)

View Large Files in Emacs

    (use-package vlf
      :ensure t)


<a id="org0799282"></a>

## [which-key](https://github.com/justbur/emacs-which-key/)

Emacs package that displays available keybindings in popup

    (use-package which-key
      :ensure t
      :init
      (which-key-mode))


<a id="org27e84be"></a>

## Themeing

Themeing configuration should happen last


### [solaire-mode](https://github.com/hlissner/emacs-solaire-mode)

solaire-mode is an aesthetic plugin that helps visually distinguish
file-visiting windows from other types of windows (like popups or
sidebars) by giving them a slightly different &#x2013; often brighter &#x2013;
background.

    (use-package solaire-mode
      :ensure t
      :hook
      ((change-major-mode after-revert ediff-prepare-buffer) . turn-on-solaire-mode)
      (minibuffer-setup . solaire-mode-in-minibuffer)
      :config
      (solaire-global-mode +1)
      (solaire-mode-swap-bg))


### [doom-themes](https://github.com/hlissner/emacs-doom-themes)

An opinionated pack of modern color-themes

    (use-package doom-themes
      :ensure t
      :config
      (setq doom-themes-enable-bold t
    	doom-themes-enable-italic t)
      (load-theme 'doom-one t)
      (doom-themes-visual-bell-config)
      (doom-themes-neotree-config)
      (doom-themes-org-config))


### [doom-modeline](https://github.com/seagle0128/doom-modeline)

A fancy and fast mode-line inspired by minimalism design.

    (use-package doom-modeline
      :ensure t
      :init (doom-modeline-mode 1)
      :config
      (display-battery-mode 1))


<a id="orgcc097d0"></a>

# Systemd unit file

Here is an example of a unit file for the emacs daemon. Place this in
`~/.config/systemd/user/emacs.service`.

    [Unit]
    Description=Emacs: the extensible, self-documenting text editor
    
    [Service]
    Type=forking
    ExecStart=/usr/bin/emacs --daemon
    ExecStop=/usr/bin/emacsclient --eval "(kill-emacs)"
    Environment=SSH_AUTH_DOCK=%t/keyring/ssh
    Restart=always
    
    [Install]
    WantedBy=default.target

Once this is created run `systemctl enable --user emacs.service` to
enable the daemon, and `systemctl start --user emacs.service`

To launch a client map a keyboard shortcut to:

    /usr/bin/emacsclient -c -e "(progn (raise-frame) (x-focus-frame (selected-frame)))"


<a id="orge62a12e"></a>

# Nautilus Scripts

Nautilus allows users to create scripts that are included in the
right-click menu in the file browser. Place these in individual files
located in `$HOME/.local/share/nautilus/scripts/` and mark the as
executable.

    #!/bin/bash
    
    emacsclient -c "$@"


<a id="orga6bd5f0"></a>

# Licensing

© Copyright 2016 Sean Jones

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.

