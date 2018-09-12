
# Table of Contents

1.  [About](#org3a4f08e)
2.  [Configurations (Internal)](#orgb107158)
    1.  [Meta](#org4beed65)
    2.  [Base defaults](#org137c8d4)
    3.  [Functions](#org40591ae)
    4.  [Org Mode](#orgd43b7f2)
    5.  [Mode hooks](#orgdbd3e2e)
    6.  [Keybindings](#org62ed213)
3.  [Configurations (External)](#org71a6fe3)
    1.  [Packages](#org835a628)
    2.  [Auto Complete](#orgc49beed)
    3.  [Elpy](#org2a4103f)
    4.  [Helm](#org291f166)
    5.  [Magit](#orgd623dc2)
    6.  [Multiple cursors](#org1718cb1)
    7.  [Paredit](#orgafaa2fa)
    8.  [Projectile](#org9a18e40)
    9.  [Neotree](#orgdf7397a)
    10. [Themeing](#orga54e778)
4.  [Systemd unit file](#org219d396)
5.  [Licensing](#org12cc6d4)



<a id="org3a4f08e"></a>

# About

This configuration is based off of the system shown [here](https://github.com/larstvei/dot-emacs). The idea is
that the configuration should serve as it's own plain english
documentation.

Install with:

    git clone git@github.com:seandjones92/Emacs.git ~/.emacs.d

Once the repo is cloned execute the following commands to prevent the
dynamic configuration from being tracked in git:

    cd ~/.emacs.d
    git update-index --assume-unchanged init.el

If you want to make changes to the repo-version of init.el start tracking again with:

    git update-index --no-assume-unchanged init.el


<a id="orgb107158"></a>

# Configurations (Internal)

This section contains all of the configurations that do not rely on
external packages. If the configuration cannot be accomplished by a
standalone Emacs installation with no internet connection then it does
not belong here.


<a id="org4beed65"></a>

## Meta

All changes to the config should be made to `init.org`, **not** to
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


<a id="org137c8d4"></a>

## Base defaults

Here we define the basic look and feel of Emacs.

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
quotes. It also adds some inteligent handling. Further in the
configuration we use `paredit`, which takes things a step further.

    (electric-pair-mode 1)
    (require 'paren)
    (setq show-paren-style 'parenthesis)
    (show-paren-mode 1)

Enable spell checking.

    (setq ispell-dictionary "american")

Disable word wrapping by default, I don't like it.

    (set-default 'truncate-lines t)

Use `*scratch*` as initial screen. Also, modify the message at the top
of the buffer.

    (setq inhibit-startup-screen t)
    (setq initial-scratch-message ";; Scratch page\n\n")


<a id="org40591ae"></a>

## Functions

These are my custom functions. I define them all here. If I want them
assigned to a keybinding I do so later in the config.

This function is to be run in `dired`. It prompts for a regular
expression and only shows the entries (files or directories) that
match that regular expression. This is good for working in directories
with lots of files. Think `ls -al | grep -E <expression>`.

    (defun dired-show-only (regexp)
      "Only show files matching the regexp."
      (interactive "sFiles to show (regexp): ")
      (dired-mark-files-regexp regexp)
      (dired-toggle-marks)
      (dired-do-kill-lines))

This function is used to terminate all TRAMP connections and to kill
all buffers associated with TRAMP connections. Sometimes I'll have a
lot going on, machines I'm no longer working on, too many buffers to
sort through and this helps.

    (defun go-local ()
      "Clean up all remote connections."
      (interactive)
      (ignore-errors (tramp-cleanup-all-connections))
      (ignore-errors (tramp-cleanup-all-buffers)))

This, in my opinion, is how Emacs should behave by default when saving
files. Strip all white space from the end of the file and the ends of
lines before saving.

    (defun save-buffer-clean ()
      "Strip the trailing whitespace from a file and save it."
      (interactive)
      (delete-trailing-whitespace)
      (save-buffer))

Again, another function to get what I would like to be default
behavior. This one handles killing buffers. If there is more than one
buffer and I kill one, kill its window too.

    (defun smart-buffer-kill ()
      "Kill buffers in a way that makes sense."
      (interactive)
      (if (= (count-windows) 1)
          (kill-buffer)
        (kill-buffer-and-window)))

This is one I don't use very often but can be useful. Copy the SSH
public key to the clipboard.

    (defun ssh-clip ()
      "Copy '~/.ssh/id_rsa.pub' to clipboard.
    This will first empty the kill-ring (clipboard)"
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


<a id="orgd43b7f2"></a>

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


<a id="orgdbd3e2e"></a>

## Mode hooks

This is where mode hooks are manipulated.

For `text-mode` I do want word wrapping enabled and `auto-fill-mode`
enabled. For me this makes sense when thinking about regular old
`*.txt` files.

    (add-hook 'text-mode-hook 'auto-fill-mode)
    (add-hook 'text-mode-hook 'toggle-truncate-lines)

I don't like `global-linum-mode` so I only turn it on for specific
modes.

    (add-hook 'sh-mode-hook 'linum-mode)
    (add-hook 'python-mode-hook 'linum-mode)


<a id="org62ed213"></a>

## Keybindings

This is where I define my custom keybindings.

    (global-set-key (kbd "C-x C-k") 'smart-buffer-kill)
    (global-set-key (kbd "C-c k") 'kill-this-buffer)
    (global-set-key (kbd "C-x C-s") 'save-buffer-clean)
    (require 'dired)
    (define-key dired-mode-map [?%?h] 'dired-show-only)

Enable keybindings that are disabled by default:

    (put 'narrow-to-page 'disabled nil)


<a id="org71a6fe3"></a>

# Configurations (External)

Configurations after this point rely on external packages. Anything
added from here on out should be designed to fail gracefully in case
the package is not available.


<a id="org835a628"></a>

## Packages

This section goes over the configuration of package management. To
start this off we need to define a few things. First we will configure
the repositories we wish to use. The `jorgenschaefer.github.io` repo
is only needed for the Elpy package.

    (require 'package)
    (setq package-archives '(("gnu" . "https://elpa.gnu.org/packages/")
    			 ("melpa" . "https://stable.melpa.org/packages/")
    			 ("elpy" . "https://jorgenschaefer.github.io/packages/")))

Next we define a function to determine if we have access to the
internet. We need to wrap this in a check for Windows since `ping`
options behave differently.

    (defun internet-up ()
      (call-process "ping" nil nil nil "-c" "1" "www.google.com"))

Next we define a list containing all of the packages that should be
installed to take full advantage of this configuration. The [Silver
Searcher](https://github.com/ggreer/the_silver_searcher) should be installed to use the `ag` and `helm-ag` packages.

    (setq my-packages '(ag
    		    all-the-icons
    		    auto-complete
    		    elpy
    		    gist
    		    helm
    		    helm-ag
    		    helm-projectile
    		    magit
    		    markdown-mode
    		    moe-theme
    		    multiple-cursors
    		    neotree
    		    org-bullets
    		    paredit
    		    projectile))

The next function defined is to loop through the provided list of
packages and to check if they are present. If not, the package is
installed:

    (defun auto-package-mgmt ()
      "Install my packages"
      (interactive)
      (package-initialize)
      (package-refresh-contents)
      (dolist (package my-packages)
        (if (ignore-errors (require package))
    	(message "%s is already installed..." package)
          (package-install package))))

To tie it all together we bring in the logic. If this is the first
launch of Emacs and we have access to the internet, loop through the
list of packages to ensure they are installed. If we do not have
access to the internet, or if this is not Emacs first launch then
nothing is done. Package dependent configuration is handled gracefully
so if there is no internet there should be no issue.

    (if (file-directory-p (concat user-emacs-directory "elpa"))
        (package-initialize)
      (if (internet-up)
          (auto-package-mgmt)))


<a id="orgc49beed"></a>

## Auto Complete

Here is where auto complete is configured. The `ac-sources` variable
needs to be set or the completion framework won't kick in.

    (defun my-autocomplete-setup ()
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
    
    (if (require 'auto-complete-config)
        (my-autocomplete-setup))


<a id="org2a4103f"></a>

## Elpy

Elpy is used to get IDE like functionality for Python. To get full use
of this package run `pip install --user jedi flake8 importmagic
autopep8`.

    (defun check-for-user-bin ()
      (if (file-directory-p "~/.local/bin")
          (setenv "PATH" (concat (getenv "PATH") ":~/.local/bin"))))
    
    (defun my-elpy-keybindings ()
      (define-key elpy-mode-map (kbd "<f12>") 'elpy-goto-definition)
      (define-key elpy-mode-map (kbd "S-<f12>") 'elpy-goto-definition-other-window))
    
    (defun my-elpy-setup ()
      (package-initialize)
      (elpy-enable)
      (check-for-user-bin)
      (add-hook 'elpy-mode-hook 'my-elpy-keybindings))
    
    (if (require 'elpy)
        (my-elpy-setup))


<a id="org291f166"></a>

## Helm

[Helm](https://github.com/emacs-helm/helm) is an Emacs framework for incremental completions and narrowing
selections. It's a much better way to interact with Emacs. I've broken
it out into smaller chunks so I can better explain what's going on.

This section enables fuzzy finding in almost everything Helm
does. This helps to really speed up interaction with emacs since you
can just type a couple partially completed words to get full phrases
instead of spelling everything out.

    (defun my-helm-fuzzy-settings ()
      (setq helm-M-x-fuzzy-match t
    	helm-buffers-fuzzy-matching t
    	helm-recentf-fuzzy-match t
    	helm-semantic-fuzzy-match t
    	helm-imenu-fuzzy-match t
    	helm-apropos-fuzzy-match t
    	helm-lisp-fuzzy-completion t
    	helm-mode-fuzzy-match t
    	helm-completion-in-region-fuzzy-match t))

This part is where keybindings relevant to Helm are defined. The one
I've found to be most useful is `helm-mini` which is activated with
`C-x x`. This will show you currently open buffers and recent files.

    (defun my-helm-keybindings ()
      (global-set-key (kbd "C-c h") 'helm-command-prefix)
      (global-unset-key (kbd "C-x c"))
      (global-set-key (kbd "M-x") 'helm-M-x)
      (global-set-key (kbd "M-y") 'helm-show-kill-ring)
      (global-set-key (kbd "C-x x") 'helm-mini)
      (global-set-key (kbd "C-x C-f") 'helm-find-files)
      (global-set-key (kbd "C-c h o") 'helm-occur)
      (global-set-key (kbd "C-x C-b") 'helm-buffers-list)
      (define-key helm-map (kbd "<tab>") 'helm-execute-persistent-action)
      (define-key helm-map (kbd "C-i") 'helm-execute-persistent-action)
      (define-key helm-map (kbd "C-z") 'helm-select-action))

This section has some more miscellaneous settings. In all honesty I
need to research them a bit more to accuratly describe what each of
these does.

    (defun my-helm-misc ()
      (add-to-list 'helm-sources-using-default-as-input 'helm-source-man-pages)
    
      (when (executable-find "curl")
        (setq helm-net-prefer-curl t))
    
      (when (executable-find "ack-grep")
        (setq helm-grep-default-command "ack-grep -Hn --no-group --no-color %e %p %f"
    	  helm-grep-default-recurse-command "ack-grep -H --no-group --no-color %e %p %f"))
    
      (setq helm-split-window-inside-p t
    	helm-move-to-line-cycle-in-source t
    	helm-ff-search-library-in-sexp t
    	helm-scroll-amount 8
    	helm-ff-file-name-history-recentf t))

This section tells the Helm interface that it should resize itself
depending on how much content it has to display, but should take up no
more than 65 percent of the Emacs interface.

    (defun my-helm-sizing ()
      (helm-autoresize-mode 1)
      (setq helm-autoresize-max-height 65))

Next we tie all of these pieces together in a setup function. It is
important to have the `(require 'helm-config)` on top or else the
configuration will fail.

    (defun my-helm-setup ()
      (require 'helm-config)
      (my-helm-fuzzy-settings)
      (my-helm-keybindings)
      (my-helm-misc)
      (my-helm-sizing)
      (helm-mode 1))

Finally we will check to see if Helm is available before applying any
of these settings.

    (if (require 'helm)
        (my-helm-setup))


<a id="orgd623dc2"></a>

## Magit

Magit is something that, in my opinion, should be shipped by default
with Emacs. It's the most robust Git interface out there.

    (defun my-magit-setup ()
      (global-set-key (kbd "C-x g") 'magit-status)
      (global-set-key (kbd "C-x M-g") 'magit-dispatch-popup))
    
    (if (require 'magit)
        (my-magit-setup))


<a id="org1718cb1"></a>

## Multiple cursors

This is pretty self explanitory. Everyone has seen Sublime Texts
multiple cursors feature, this lets you do it in Emacs.

    (defun my-multicursor-setup ()
      (global-set-key (kbd "C-S-c C-S-c") 'mc/edit-lines)
      (global-set-key (kbd "C->") 'mc/mark-next-like-this)
      (global-set-key (kbd "C-<") 'mc/mark-previous-like-this)
      (global-set-key (kbd "C-c C-<") 'mc/mark-all-like-this))
    
    (if (require 'multiple-cursors)
        (my-multicursor-setup))


<a id="orgafaa2fa"></a>

## Paredit

This is for better handling of S-expressions in lisp languages.

    (autoload 'enable-paredit-mode "paredit" "Turn on pseudo-structural editing of Lisp code." t)
    (add-hook 'emacs-lisp-mode-hook       #'enable-paredit-mode)
    (add-hook 'eval-expression-minibuffer-setup-hook #'enable-paredit-mode)
    (add-hook 'ielm-mode-hook             #'enable-paredit-mode)
    (add-hook 'lisp-mode-hook             #'enable-paredit-mode)
    (add-hook 'lisp-interaction-mode-hook #'enable-paredit-mode)
    (add-hook 'scheme-mode-hook           #'enable-paredit-mode)
    (add-hook 'eshell-mode-hook           #'enable-paredit-mode)
    (add-hook 'clojure-mode-hook          #'enable-paredit-mode)
    (add-hook 'cider-repl-mode            #'enable-paredit-mode)


<a id="org9a18e40"></a>

## Projectile

Projectile makes emacs "project aware". This is good if you work on
multiple code bases and want to navigate between them and within them
efficiently.

    (defun my-projectile-keybindings ()
      (define-key projectile-mode-map (kbd "C-c a") 'helm-projectile-ag))
    
    (defun my-projectile-setup ()
      (projectile-mode)
      (projectile-discover-projects-in-directory default-directory)
      (add-hook 'projectile-mode-hook 'my-projectile-keybindings))
    
    (if (require 'projectile)
        (my-projectile-setup))


<a id="orgdf7397a"></a>

## Neotree

Adds a file tree to the left hand side, like in most IDEs. This only
works if you are in a project.

In order for this to look right the fonts for `all-the-icons` must be
installed. This is accomplished by `M-x all-the-icons-install-fonts`.

    (defun neotree-project-dir ()
      "Open NeoTree using the git root."
      (interactive)
      (let ((project-dir (projectile-project-root))
    	(file-name (buffer-file-name)))
        (neotree-toggle)
        (if project-dir
    	(if (neo-global--window-exists-p)
    	    (progn
    	      (neotree-dir project-dir)
    	      (neotree-find file-name)))
          (message "Could not find git project root."))))
    
    (defun my-neotree-setup ()
      (global-set-key (kbd "C-c n") 'neotree-project-dir)
      (if (eq system-type 'windows-nt)
          (setq neo-theme 'arrow)
        (setq neo-theme 'icons))
      (setq projectile-switch-project-action 'neotree-projectile-action)
      (setq neo-window-width 30))
    
    (if (require 'neotree)
        (my-neotree-setup))


<a id="orga54e778"></a>

## Themeing

Here we do some themeing of emacs. None of this has any functional
impact, it just make the editor a little nicer to look at. We can see
that we have theming for the modeline, org mode bullets, and the
general theme of Emacs. I try to make this as robust as possible. If
one of these pieces is missing (for whatever reason) the rest of the
theme should still be put together.

    (defun my-moetheme-setup ()
      (setq moe-theme-highlight-buffer-id t)
      (setq moe-theme-resize-markdown-title '(2.0 1.7 1.5 1.3 1.0 1.0))
      (setq moe-theme-resize-org-title '(2.2 1.8 1.6 1.4 1.2 1.0 1.0 1.0 1.0))
      (moe-dark))
    
    (if (require 'moe-theme)
          (my-moetheme-setup))
    
    (if (require 'org-bullets)
        (add-hook 'org-mode-hook
    	      (lambda ()
    		(org-bullets-mode 1))))


<a id="org219d396"></a>

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


<a id="org12cc6d4"></a>

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

