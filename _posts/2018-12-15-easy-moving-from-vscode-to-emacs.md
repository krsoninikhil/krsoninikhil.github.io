If you have already decided to go with Emacs as your choice of editor,
congratulations! This is going to be great. However, the part after
this is what's hard and takes time. You might already be missing some
of the very basic functionalities like opening a project file without
typing the full path or searching something project-wide that you are
very accustomed to. These features usually comes out of the box or by
installing some extensions that do not require any configuration,
with most of graphics based editors like Atom, VS Code or Sublime. If
that is the case, keep reading. When I first started using Emacs, I
spent quite some time reading and trying out stuffs to make these
things work and this article is exactly because of that -- to make
your transition smoother. The focus is NOT on being more productive
with Emacs or to give you tricks and fancy lisp code that will make
you Emacs ninja overnight (they exists btw). That is something you can
focus on once you're comfortable using Emacs. This is my attempt to
make every tyro feel home, to give you a setup that will allow you to
do all the stuff that you're used to.

Emacs is built around the philosophy of extensibility and flexibility
-- being able to hook your own code at any event of your choice is
what makes it awesome. And because of that, the huge community around
the editor have highly opinionated setups based on what works for
them. There exists modified versions of Emacs like Prelude, Spacemacs,
etc. that are loaded with a lot of functionalities you wouldn't find
anywhere else but in my personal opinion, it's good to know what is cake
and what are the cherries.

Before we start, familiarize yourself with basic key combinations and
terminology of Emacs. Built-in tutorial explains that very neatly
which you can read by `C-h t` (hold control key and press h, then
leave control key and press t). [This awesome
article](http://www.jesshamrick.com/2012/09/10/absolute-beginners-guide-to-emacs/)
from last week's HN front page is also a great read for this.

Let's start. After moving from Atom and VSCode to Emacs, I missed these
functionalities:

1. Being able to duplicate current line
2. Move or drag a line up and down
3. Use multiple cursors to edit at a time
4. Auto-completion of variables and functions name
5. Fuzzy file search i.e. Ctrl-P or Cmd-Shift-o in other editors
6. Searching a term of string in the whole project
7. A tree view of the project to explore file structure

   Few more things that I had installed extensions for in VSCode:

8. Goto functions and class definitions
9. Come back to previous cursor position
10. Find all the reference of a function
11. Markdown preview

Surprisingly, some of these are as easy as hooking a couple of lines
of code to a keystroke, for some a little more than a couple of lines
so we'll use community developed packages for them. The latest version
of Emacs ships with a package manager -- `package.el`. We just need to
add MELPA repository to it so it can search packages for us. Add this
Emacs Lisp ([ref](https://emacs.stackexchange.com/a/2989/21028)) to
your `~/.emacs.d/init.el` file to do that.

```elisp
(require 'package)
(setq package-archives
      '(("GNU ELPA"     . "https://elpa.gnu.org/packages/")
        ("MELPA Stable" . "https://stable.melpa.org/packages/")
        ("MELPA"        . "https://melpa.org/packages/"))
      package-archive-priorities
      '(("GNU ELPA"     . 10)
        ("MELPA Stable" . 5)
        ("MELPA"        . 0)))
(package-initialize)
```

This will also give priorities to repositories to avoid duplicate
listing in case of certain packages being present in more than
one. After adding this restart Emacs to reload `init.el` and do this:
`M-x package-refresh-contents`. Now you are ready to install most of
the packages. Note that you can also reload you `init.el` by doing
`M-x load-file` and then providing file path.

Rest of the article is about enabling above mentioned functionalities
one by one.


1. Add this code to same `init.el`

    ```elisp
    (defun duplicate-line ()
      (interactive)
      (save-mark-and-excursion
        (beginning-of-line)
        (insert (thing-at-point 'line t))))

    (global-set-key (kbd "C-S-d") 'duplicate-line)
    ```

    Now, you can use `C-S-d` (Control-Shift-d) to duplicate current
    line. Change these key combination to whatever suits you.

2. Add these lines for using `C-S-j` and `C-S-k` to move a line up or
   down one line.

    ```elisp
    (defun move-line-down ()
      (interactive)
      (let ((col (current-column)))
        (save-excursion
          (forward-line)
          (transpose-lines 1))
        (forward-line)
        (move-to-column col)))

    (defun move-line-up ()
      (interactive)
      (let ((col (current-column)))
        (save-excursion
          (forward-line)
          (transpose-lines -1))
        (forward-line -1)
        (move-to-column col)))

    (global-set-key (kbd "C-S-j") 'move-line-down)
    (global-set-key (kbd "C-S-k") 'move-line-up)
    ```

3. To get mutiple cursor, I use `multiple-cursor` package, which can
   be installed by `M-x package-install <RET> multiple-cursor <RET>`. Add these
   key bindings to easily use it.

    ```elisp
    (require 'multiple-cursors)
    (global-set-key (kbd "C-|") 'mc/edit-lines)
    (global-set-key (kbd "C->") 'mc/mark-next-like-this)
    (global-set-key (kbd "C-<") 'mc/mark-previous-like-this)
    (global-set-key (kbd "C-c C-<") 'mc/mark-all-like-this)
    (global-set-key (kbd "C-S-<mouse-1>") 'mc/add-cursor-on-click)
    (define-key mc/keymap (kbd "<return>") nil)
    ```

   These commands do exactly what they look like. To get out of multiple
   cursors, use `C-g`, the last line in above code prevents using `<RETURN>`
   key to do same. You can read more about it in the project [doc page](http://stable.melpa.org/#/multiple-cursors).

4. To let Emacs auto-complete function names and variables, I use
   [`company`](http://stable.melpa.org/#/company), which can be
   installed by `M-x package-install <RET> company-mode <RET>`. And can be
   activated by adding this line to `init.el`:

    ```elisp
    (add-hook 'after-init-hook 'global-company-mode)
    ```

5. I use the combination of
   [`projectile`](https://www.projectile.mx/en/latest/) and
   [`helm`](https://emacs-helm.github.io/helm/) package to deal with
   fuzzy file search. Configurations for `helm` can be a bit overwhelming so
   just start with following.

    ```elisp
    (require 'projectile)
    (setq projectile-indexing-method 'alien)
    (setq projectile-enable-caching t)
    (projectile-global-mode)

    (require 'helm)
    (require 'helm-config)
    (global-set-key (kbd "C-c h") 'helm-command-prefix)
    (global-unset-key (kbd "C-x c"))
    (helm-autoresize-mode 1)
    (global-set-key (kbd "M-x") 'helm-M-x)
    (setq helm-M-x-fuzzy-match t)
    (global-set-key (kbd "C-x C-f") 'helm-find-files)
    (helm-mode 1)
    ```

   This will allow you to use `C-c p f` for opening your project files
   just by typing file name and not the whole path. And when you use `C-x
   C-f` to open a file that is outside the project directory, you can
   type any part of the directory or file name or even non contiguous
   parts separated by space to narrow down the suggestions e.g. if you
   want to select `controllers` out of `controllers` and `contol` folder
   name, you can type `co s`. Use `C-j` (not TAB) to choose highlighted
   option.

6. `projectile` can also be used to search for something in the whole
   project: `C-c p s s`. This command used `ag` system package, which
   should definitely have and a `ag.el` Emacs package. I don't
   generally use project wild search and replace but if that's your
   thing, [this](https://emacs.stackexchange.com/a/243/21028) answer
   from Stackexchange explains a great way of doing so.

7. Built-in `speedbar` provides good enough interactive tree view of
   project. It can be started with `M-x speedbar`. It's easier to bind
   it to some key like F8 using

    ```elisp
    (global-set-key (kbd "<f8>") 'speedbar)
    ```

8. I use
   [`dumb-jump`](https://github.com/jacktasia/dumb-jump/tree/260054500d4731c36574b6cbc519de29fdd22f43)
   package to jump to definitions using `C-M-g`. Add this line to auto enable it
   every time:

    ```elisp
    (dumb-jump-mode)
    ```

9. You can jump back to function call by `C-M-p`.

10. To find all the references of a function, priviously mentioned
    project-wide search with `projectile` works well for me.

11. I use `fymd` package to real time markdown preview, with this
    key binding:

    ```elisp
    (global-set-key (kbd "<f9>") 'flymd-flyit)
    ```

Most of the snippets on this page are taken from
[emacsrocks.com](http://emacsrocks.com/),
[whattheemacsd.com](http://whattheemacsd.com/) and official
documentation of mentioned packages. Both of these resources are
great places to learn more about
Emacs. [Here](https://github.com/krsoninikhil/dotfiles/tree/master/.emacs.d)
is my `.emacs.d` directory, if don't feel like doing this all by yourself,
feel free to clone mine. I have all the `defuns` defined in
`~/.emacs.d/user-lisp/defuns.el` file which is imported in `init.el` by:

```elisp
(add-to-list 'load-path "~/.emacs.d/user-lisp")
(require 'defuns)
```

I keep all the references to these defuns in `init.el` so all the key
bindings remain in one file.

I would highly appreciate any feedback you may have or to listen about
any other features you miss from your old editor and should be in this
list.

[Edit]

Code snippet for duplicating line (first point) has been updated with
current cleaner version suggested by [Philip K.](https://zge.us.to/),
earlier it was:

```elisp
(defun duplicate-line ()
   (interactive)
   (let ((col (current-column)))
     (move-beginning-of-line 1)
     (kill-line)
     (yank)
     (newline)
     (yank)
     (move-to-column col)))
```

-- [@krsoninikhil](https://twitter.com/krsoninikhil)
