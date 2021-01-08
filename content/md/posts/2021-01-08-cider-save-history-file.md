{:title "Cider: Save History File per Project"
 :layout :post
 :tags  ["Clojure" "clojure" "cider"]
 :toc? true
 :draft? false}

## How to have a cider-nrepl history file per project

Well, indeed it is very simple, just set into `.dir-locals.el`
Emacs file the variable `cider-repl-history-file`

```lisp
((cider-repl-mode
  (cider-repl-history-file . ".cider-nrepl.history")))
```

When Emacs starts it reads this file and the variable get assigned the relative path
bounded to the current directory - where the `.dir-locals.el` is found -.

When Emacs is closed the history is saved into the `.cider-nrepl.history` file,
as expected.

Quit everything togheter (cider-nrepl and Emacs) to avoid what follows.


A little nuisance to keep in mind:

when cider-nrepl starts it binds some hooks in Emacs, to save history either at exiting or closing the buffer,
if the cider-nrepl is quitted before exiting Emacs the aforementioned hooks are onset but the variable
`cider-repl-history-file` is not bounded anymore (only in `cider-repl-mode`), and trying to exit from Emacs
triggers the error

```lisp
stringp, nil
```

To overcome this either quit everything togheter via `C-x C-c` or open another buffer/file \*Scratch\* whatever
not in cider-mode and exit from there.
Reopening another cider-nrepl and hitting `C-x C-c` works too.

Unluckily setting the variable to be bounded with other modes - i.e. cider-mode, clojure-mode or nil (all modes) -
does not work in a smooth way, an empty `.cider-nrepl.history` might be created nested in the project
structure, in the same directory where Emacs started opening a file - i.e. `src/yourapp/.cider-nrepl.history` -
