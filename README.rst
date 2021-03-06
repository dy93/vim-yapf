========================
vim-yapf
========================

.. image:: https://badges.gitter.im/Join%20Chat.svg
   :alt: Join the chat at https://gitter.im/mindriot101/vim-yapf
   :target: https://gitter.im/mindriot101/vim-yapf?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge

**ATTENTION:** you probably don't need this plugin! See `Why you may not need this plugin`_.

``vim-yapf`` is a Vim plugin that applies yapf to your current file.
``yapf`` automatically formats Python code, based on improved syntax styles.


Required
=====================

* `yapf <https://pypi.python.org/pypi/yapf/>`_

``yapf`` can be installed either with ``pip``:

::

 pip install yapf

or by installing with conda (OSX and linux64 only) with my binstar channel:

::

 conda install -c https://conda.binstar.org/srwalker101 yapf

TL;DR
=====================
To format only your changes, use ``vim-yapf`` with ``vim-gitgutter``. For example, add following to your ``vimrc``:

.. code:: vim

   call plug#begin('~/.vim/plugged')
   " yapf formater
   Plug 'dy93/vim-yapf', { 'for': ['python'] }

   " gitgutter
   Plug 'airblade/vim-gitgutter'
   call plug#end()

   function! MyAutoFormat()
     let g:gitgutter_async = 0
     execute "GitGutterAll"
     let hunks = GitGutterGetHunks()
     if len(hunks) > 0
       let lines = []
       for hunk in GitGutterGetHunks()
         if hunk[2] <= hunk[2]+hunk[3]-1
           call add(lines, '-l '.hunk[2].'-'.(hunk[2]+hunk[3]-1))
         else
           call add(lines, '-l '.(hunk[2]+hunk[3]-1).'-'.hunk[2])
         endif
       endfor
       echom "Yapf --style ~/.style.yapf ".join(lines, ' ')
       let save = winsaveview()
       execute "Yapf --style ~/.style.yapf ".join(lines, ' ')
       call winrestview(save)
     else
       execute "Yapf --style ~/.style.yapf"
     endif
     let g:gitgutter_async = 1
   endfunction

   " formater
   autocmd BufWritePre *.py :call MyAutoFormat()

and generate your yapf config:

.. code:: bash

   $ yapf --style-help > ~/.style.yapf



Installation
=====================

Either use a plugin manager and add ``Plug[in] 'dy93/vim-yapf'`` to your ``vimrc``, or use pathogen.

Bindings
=====================

The plugin does not create any bindings by default, this is left up to the user. An example could be:


::

 :nnoremap <leader>y :call Yapf()<cr>

or

::

 :nnoremap <leader>y :Yapf<cr>



Usage
=====================

call function

::

 :Yapf

with arguments

::

 :Yapf --style google

or

::

 :call Yapf(" --style pep8")

Customization
=====================

The yapf style can be globally set, in your vimrc:

::

 let g:yapf_style = "google"

or

::

 let g:yapf_style = "pep8"

Why you may not need this plugin
================================

The plugin itself is very simple. It handles user options granted, but at its core it uses ex commands to perform its magic. ``yapf`` behaves like a good unix command: it takes text on ``stdin`` and spits the altered result to ``stdout``, which is exactly what vim expects.

At its core, this plugin runs the ex command:

::

 0,$!yapf

This pipes the range ``0,$`` i.e. the whole file through a shell command ``yapf`` and replaces the range with the altered result.

Instead of installing this plugin, one could add a mapping e.g.:

::

 autocmd FileType python nnoremap <leader>y :0,$!yapf<Cr><C-o>

Alternatively yapf could be set as the ``formatprg`` for the python filetype, and reformatting can be performed with the `gq{motion}`_ operator (e.g. with visual selection) to reformat a part of the file.
Alternatively alternatively yapf could be set as the ``equalprg``:

::

 setlocal equalprg=yapf

and reformat the whole file with ``gg=G`` or a single line with ``=``.

.. _gq{motion}: https://github.com/vim/vim/blob/b182b40080a23ea1e1ffa28ea03b412174a236bb/runtime/doc/change.txt#L1299
